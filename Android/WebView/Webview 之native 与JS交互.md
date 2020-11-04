# Webview 之native 与JS交互

## 客户端调用 JS

### loadUrl()

我们明白 WebView 其实就是在加载网页，所以客户端可以直接访问 `javascript:console.log('hello')` 这样的伪 URL 即可实现在页面注入需要执行的 JS 代码。调用方法如下：

```java
WebView webview = (WebView) findViewById(R.id.webView);webview.loadUrl("javascript:console.log('hello')");
```

这样我们就实现了调用 JS 的目的了。`loadUrl()` 的方案从另外一个角度来看可以算是 hack 方案了，对客户端来说，他们的 JS 交互本质上其实就是一个拼接 JS 字符串的过程。

### evaluateJavascript()

刚才我们也说了 `loadUrl()` 不是 Android 的正经解决方法。好在官方也想到了这点，在 Android 4.4+ 之后，官方给提供了原生的方法支持调用，那就是 `evaluateJavascript()`。这个方法最大的好处就是能够直接在一次执行的时候获取到 JS 返回的结果。如果是使用 `loadUrl()` 的方式的话，执行完后对客户端来说这句话就结束了，如果想要拿到返回的结果的话另外需要 JS 调用客户端的方法返回。

```java
WebView webview = (WebView) findViewById(R.id.webView);webview.evaluateJavascript（"javascript:Date.now()", new ValueCallback<String>() {    @Override    public void onReceiveValue(String value) {        System.out.println(value); //1515827651551    }});
```

## JS 调用客户端

相比较客户端调用 JS 的方法，JS 调用客户端的方法就比较多了，简单归类一下其实可以分为注入映射和方法劫持两种。注入映射主要是使用官方提供的 `addJavascriptInterface()` 方法将 Java 对象和 JS 对象进行映射。而方法劫持则是利用 JS 的一些系统方法调用会触发 Java 的事件回调，然后在回调中进行事件劫持，从而执行客户端方法。下面我们来具体看看。

### addJavascriptInterface

`addJavascriptInterface()` 方法的使用非常简单，定义好被调用的方法对象后直接配置映射关系即可。

```java
//定义好 Java 接口对象
public class SDK extends Object {
    @JavascriptInterface
    public void hello(String msg) {
        System.out.println("Hello World");
    }
}
//Webview 中调用
WebView webview = (WebView) findViewById(R.id.webview);
webview.addJavascriptInterface(new SDK(), 'sdk');
webview.loadUrl('http://imnerd.org'); //注入后加载页面
```

这样加载的页面中就可以直接执行 `sdk.hello()` 方法来执行客户端方法了。不过这种官方推荐的方法在 4.2- 的系统上存在[远程执行安全漏洞](https://jaq.alibaba.com/blog.htm?id=48)，对 4.2 以下系统版本有要求的应用需要谨慎使用。目前来看 4.2 还是需要保持支持的。

### URL劫持

URL劫持主要是使用 `shouldOverrideUrlLoading()` 进行 WebView URL 劫持。从方法名可以看出，它是 WebView 拦截 URL 的一种回调，当 WebView 发生 URL 跳转的时候会触发该回调。在该回调中我们能够获取到前端提供的 URL 地址。我们通过构造约定协议的 URL 地址提供给客户端识别，识别成功后执行对应的方法即可。

```java
WebView webview = (WebView) findViewById(R.id.webview);
webview.loadUrl('http://imnerd.org');
webview.setWebViewClient(new WebViewClient() {
  @Override
  public boolean shouldOverrideUrlLoading(WebView view, String url) {
    if(url.equals('sdk:hello')) {
      System.out.println('hello world');
      return true;
    }
    return super.shouldOverrideUrlLoading(view, url);
  }
});
```

### 方法劫持

同 URL 劫持类似，方法劫持主要是利用 JS 的一些方法执行时会触发 Android 客户端中的一些回调，通过对前端参数进行识别来执行对应的客户端代码。目前前端主要有以下四种方法会触发对应的回调方法，对应关系如下：

| JS方法 | 客户端回调 |
|-------------|------------------|
| alert | onJsAlert |
| prompt | onJsPrompt |
| confirm | onJsConfirm |
| console.log | onConsoleMessage |

将这四个方法列在一块是因为这几个方法的本质上都是差不多，定义好对应的回调方法即可。客户端具体的配置如下：

```java
//定义好劫持回调类
private class hijackWebChromeClient extends WebChromeClient {  
  public boolean hijack(String text) {
    if(text.equals('sdk:hello')) {
      System.out.println('hello world');
      return true;
    }
    return false;
  }
  @Override
  public boolean onJsPrompt(WebView view, String message, String defaultValue, JSPromptResult result) {
    if(this.hijack(message)) {
      return true;
    }
    return super.onJsPrompt(view, url, message, defaultValue, result);
  }
  @Override
  public boolean onJsAlert(WebView view, String url, String message, JsResult result) {
    if(this.hijack(message)) {
      return true;
    }
    
    return super.onJsAlert(view, url, message, result);
  }
  @Override
  public boolean onJsConfirm(WebView view, String url, String message, JsResult result) {
    if(this.hijack(message)) {
      return true;
    }
    
    return super.onJsConfirm(view, url, message, result);
  }
  @Override  
  public boolean onConsoleMessage(ConsoleMessage consoleMessage) {  
    String message = consoleMessage.message();
    if(this.hijack(message)) {
      return true;
    }
    
    return super.onConsoleMessage(consoleMessage);  
  }  
  @Override  
  public void onConsoleMessage(String message, int lineNumber, String sourceID) {
    if(this.hijack(message)) {
      return true;
    }
    super.onConsoleMessage(message, lineNumber, sourceID);  
  }  
}
//注入劫持回调类
WebView webview = (WebView) findViewById(R.id.webview);
webview.loadUrl('http://imnerd.org');
webview.setWebChromeClient(new hijackChromeClient);
```

这里为了方便展示，将所有回调的方法都写全了，实际上在实际的使用过程中一般都是约定好一种调用方式即可。另外 `console.log` 对应的回调写了两种，三参数的是老版本方法，在新API中已经被[废弃](https://developer.android.com/reference/android/webkit/WebChromeClient.html#onConsoleMessage(java.lang.String,%20int,%20java.lang.String))，推荐使用 `ConsoleMessage` 对象传参方式。