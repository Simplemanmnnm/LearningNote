# SDK

1.将API作为方法保存在Interface接口中

2.**动态代理**生成接口的实例对象，将appKey、appSecret等参数保存到InvocationHandler实现类中

3.调用实例对象的方法

4.InvocationHandler实现类中构造并发送请求

### 动态代理

```java
// 请求参数保存到handler中，在handler中构造发送请求
ServiceHandler serviceHandler = new ServiceHandler(credential);
return (T) Proxy.newProxyInstance(ServiceFactory.class.getClassLoader(), new Class[]{clazz}, serviceHandler);

// ServiceFactory中存储了各个接口 使用注解保存了路径和访问方法
```

```java
// 注解,用来存储接口的访问路径、访问方法
@Target({ElementType.METHOD})
@Retention(RetentionPolicy.RUNTIME)
@Documented
public @interface ServiceMethod {
    String path();
    
    String method();
}
```

```java
public class ServiceHandler implements InvocationHandler {
    @Override
    public Object invoke(Object obj, Method method, Object[] objects) {
        // 通过注解获取到接口的 路径、访问方式
        ServiceMethod mark = method.getAnnotation(ServiceMethod.class);
        Type type = method.getGenericReturnType(); // 获取方法的返回体
        if (objects != null && objects.length > 0) { // 获取入参
            req = objects[0];
        }
        return this.exec(mark.path, req, type); // exec是自己写的方法，发送请求
    }
}
```

