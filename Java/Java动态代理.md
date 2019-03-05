# Java 动态代理





```java
package proxy;

public interface Subject {
    void doSomthing();
}
```



```java
package proxy;

public class RealSubject implements Subject{

    @Override
    public void doSomthing() {
        System.out.println("I need do some things");
    }
}
```



```java
package proxy;

import java.lang.reflect.InvocationHandler;
import java.lang.reflect.Method;

public class DynamicProxyHandler implements InvocationHandler {
    Object proxied;

    public DynamicProxyHandler(Object proxied){
        this.proxied = proxied;
    }

    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        System.out.println("method="+method+"   proxy="+proxy.getClass());
        if (args != null) {
            for (Object arg: args) {
                System.out.println("arg="+arg);
            }
        }
        return method.invoke(proxied, args);
    }
}
```



```java
package proxy;

import java.lang.reflect.Proxy;

public class Test {
    public static void main(String[] args) {
        // 动态代理
        Subject real = new RealSubject();
        Subject proxy = (Subject) Proxy.newProxyInstance(Subject.class.getClassLoader(),
                new Class[]{Subject.class}, new DynamicProxyHandler(real));
        proxy.doSomthing();
    }
}
```

我们可以看出，通过Proxy.newProxyInstance方法动态创建代理，这个方法的参数分别为类加载器，该代理实现的接口列表，以及InvocationHandler接口的一个实现。

