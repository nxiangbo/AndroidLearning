# Android 中的Cookie

上文讲了WebView中的Cookie存储，那么在非webview的场景，会用到Cookie嘛？答案是会的，例如：我们在Android App中进行登录，然后在App打开一个web，web如何知道我们是否登录了呢？我们可以借助Cookie，在调用Http请求登陆成功将对应的登陆信息写入到Cookie中，web读取这些信息就知道我们是否登录了

## 利用CookieManager存储和读取Cookie

Android中的WebKit为我们提供了CookieManager，它是一个单例，我们可以利用它进行Cookie的读取和存储，例如

```
 /**
     * Sets a cookie for the given URL. Any existing cookie with the same host,
     * path and name will be replaced with the new cookie. The cookie being set
     * will be ignored if it is expired.
     *
     * @param url the URL for which the cookie is to be set
     * @param value the cookie as a string, using the format of the 'Set-Cookie'
     *              HTTP response header
     */
 val value = "gw_id=6404; Path=/; Max-Age=86400; HttpOnly"
 CookieManager.getInstance().setCookie(url, value)
 CookieManager.getInstance().getCookie(url)
复制代码
```

# 域名共享Cookie

在一些场景下，如果需要不同的域名共享Cookie，我们需要进行一些额外的操作，为什么呢？

1. 一条Cookie包含若干属性，其中包括它生效的域名（host）、name、path，例如上面例子中url是域名，gw_id是name， / 是path。
2. 我们前面提到，只有在特定的条件下，web才会在一次请求中将cookie带给server，这个特定的条件就是cookie的域名和当前请求的域名相等。
3. 但是往往有一些特殊的场景，例如公司有N个域名，用户在A域名的网站登录了，如果再去浏览B域名下的网站，难道还需要再登录一次？这个时候就需要域名共享
4. 在Android中域名共享可以利用上面的CookieManager，在特定的时机先读取A域名的Cookie，然后设置给B域名，这样B域名就可以访问A域名的Cookie了

# 解析数据库中的Cookie

1. 前面有提到Cookie一般是储存在数据库中，那么在Android中呢？这个数据库位于 data/data/包名/aap_webview下的Cookies文件，不同Android系统可能有所差异，例如华为位于 data/data/包名/aap_webview/default下



![img](https://user-gold-cdn.xitu.io/2020/3/19/170f38249bd6c7c7?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)



1. 既然是数据库文件，自然就可以通过数据库进行解析，我们可以通过上文CookieManager的setCookie方法设置一些Cookie（**注意**：数据库存在缓存，使用CookieManager进行存取后并不会立即写入Cookies，如果想立即写入，可以调用CookieManager的flush()方法），然后将Cookies文件取出，并更改名称为Cookies.db,之后用数据库软件（作者用了DB Browser for Sqlite）进行读取（**注意**：不同版本的webview生成的表的关键字可能不同）。



cookies数据库表中有以下字段

| key             | 含义                                                         |
| :-------------- | :----------------------------------------------------------- |
| creation_utc    | Cookie产生的utc时间                                          |
| host_key        | 当前cookie生效的域名（domain）（设置父域名，对子域名也生效） |
| name            | 要保存的key3value要保存的value（未加密                       |
| path            | 路径，如/docs/Web/,如设置的话，cookie仅对当前路径生效        |
| expires_utc     | Cookie的失效时间，单位ms                                     |
| is_secure       | 如果为true，只有在https是才会传送改cookie，http不会传送      |
| is_httponly     | 如果为true，该cookie不应被客户端操作（通过JavaScript的 Document.cookie API无法访问带有HttpOnly 标记的Cookie），只应发送给server处理 |
| last_access_utc | 上一次访问到该Cookie的utc时间                                |
| has_expires     | Cookie的期限是否有效                                         |
| is_persistent   | 如果expires_utc不为0，那么这个值为1（有两种存储类型的Cookie：会话性与持久性。Expires属性缺省时，为会话性Cookie，仅保存在客户端内存中，并在用户关闭浏览器时失效） |
| priority        | Cookie的删除优先级，Cookie也有存储上限的，当超出上限则需要删除，此时会有特定的删除策略来删除不同priority的Cookie |
| encrypted_value | 加密后的value值                                              |
| samesite        | None（0）浏览器会在同站请求、跨站请求下继续发送cookies，不区分大小写。         Strict（2）浏览器将只发送相同站点请求的cookie(即当前网页URL与请求目标URL完全一致)。如果请求来自与当前location的URL不同的URL，则不包括标记为Strict属性的cookie。 Lax（1）在新版本浏览器中，为默认选项，Same-site cookies 将会为一些跨站子请求保留，如图片加载或者frames的调用，但只有当用户从外部站点导航到URL时才会发送。 |
| firstpartyonly  | first-party以及third-party是HTTP Request的一种分类，first-party指的是当前所发送的HTTP请求的URL跟浏览器地址栏上的URL一致；否则就是third-party。 |

1. firstpartyonly 在之前截图中并没有出现，为什么呢，正如之前所说的，不同版本webview所生成的表的key是不一样的，在部分版本中会用firstpartyonly取代samesite，类似的还有用persistent取代is_persistent
2. samesite讲了这么多，到底啥意思？他的作用其实很简单，假设我们有一个场景，在A域名的网页下，需要请求B域名，那么这时候Cookie怎么带？会不会不安全？这个时候samesite就发挥了作用，[详见](https://www.ruanyifeng.com/blog/2019/09/cookie-samesite.html)
3. 上边还提到了utc时间，通过截图可以看到他大概长这样：13260634474538121，这是个什么标准的时间？它是从1601年1月1日开始计时，以微妙为单位的时间。它与我们常用的1970年1月1日中间相差了11644473600妙
4. 在字段的说明中我们有提到，is_persistent 和 has_expires表明一个cookie是否有过期时间，当cookie到达expires_utc过期时间时，cookie就会失效，我们将不再能从we获取到这个cookie。但是又一种特殊的场景，如果expires_utc为0，is_persistent为0，has_expires为0，我们仍能从web或者CookieManager获取到这条Cookie


