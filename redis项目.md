## Session
> 用户首次访问服务器时，服务器会为该用户创建一个新的Session，并生成一个唯一的sessionId
> 重启服务后数据清空
### 操作
SpringMVC框架下，将 HttpSession session 加入controller方法的参数列表中
```
session.setAttribute(key, value);
Object object = session.getAttribute(key);
```
可以通过 HttpServletRequest 对象获取当前会话的Session

### sessionId
非新用户链接时cookie自动带上sessionId，服务器自动根据这个Id将本次请求找到并绑定对应session，开发者直接使用session即可，不需要关心获取

### 集群的session共享问题
多台tomcat不共享session，请求发到不同tomcat会出现问题
解决方案：终于引出了Redis!!!!

## 拦截器
```
拦截器类 实现接口org.springframework.web.servlet.HandlerInterceptor
有三个方法
preHandle -> controller执行之前
        return true 则放通
        return false 拦截
postHandle -> controller执行之后 处理器执行之后，视图渲染之前
afterCompletion -> 整个请求完成之后,视图渲染之后

拦截器生效：
配置类 实现接口org.springframework.web.servlet.config.annotation.WebMvcConfigurer
@Configuration注解修饰
重写方法public void addInterceptors(InterceptorRegistry registry)
registry.addInterceptor(new TestInterceptor());
可以添加或者排除需要拦截的接口
```

## BeanUtil
```
BeanUtil.copyproperties(数据源对象, 目标类对象)
```
## redis保存对象结构
两种方式：String形式的json;Hash结构
### Hash结构
value是若干键值对
key         value
        field    value
tool:1   name     lsq
         age      18
### 存储用户信息
生成一个随机的token作为鉴权参数
同时将token(可修饰一下)作为redis中存储用户信息的key
#### token
token的必要性：
1.随机生成然后可以做到一定时间段内有效，就算泄露也影响有限
