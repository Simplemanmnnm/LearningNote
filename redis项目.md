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

## 拦截器
实现接口org.springframework.web.servlet.HandlerInterceptor
有三个方法
preHandle -> controller执行之前
postHandle -> controller执行之后 处理器执行之后，视图渲染之前
afterCompletion -> 整个请求完成之后,视图渲染之后
