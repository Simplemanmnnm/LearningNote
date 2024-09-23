# Redis
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
BeanUtil.beanToMap
BeanUtil.fillBeanWithMap
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

## 使用redis 需要引入的依赖
```
<!-- 读取相关配置、链接、初始化bean，可直接注入RedisTemplate和StringRedisTemplate -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-redis</artifactId>
</dependency>
```

## Spring自动注入细节
```
只有spring管理的bean并且 由spring创建出来的 内可以使用resource注入
                        不能是自己new出来的
spring创建这个bean的时候才会读取resource并注入对应的bean
```

## redis和数据库数据一致性
```
优选方案：数据库更新时，主动让redis删除缓存(如果主动去更新redis可能不会被用到，做了无用功)
先操作数据库再操作缓存 + 数据有效期
```

## 缓存穿透
```
通过redis和数据库中都不存在的数据查询，请求直接打到数据库
这样的请求多了就可能压垮数据库（被攻击）

解决方案：
1. 限流 + 缓存空对象 + 命中空值时报错结束
2. 客户端与redis之间维护一个布隆过滤器， 存储数据库中数据的hash值
```

## 缓存雪崩
```
大量key失效或者redis宕机，大量请求发到数据库
解决：
1.TTL随机值 防大量key失效
2.redis集群 防宕机
3.服务降级限流 防止redis集群都宕机，舍弃部分服务保护数据库
4.多级缓存 亿级数据处理
```
## 缓存击穿
```
热Key问题，高并发访问且缓存重建业务复杂(涉及多表或者三方组件等)的key失效
导致：多线程都直接查数据库并耗时重建此key，数据库被压垮
解决：
1.重建Key做互斥操作 缺点：大量线程等待   一致性优先
2.逻辑过期，redis不设置TTL，自行维护过期时间  可用性优先
    发现过期 拿锁 新开线程重建key，新线程重建完毕自行释放锁，旧线程返回过期数据
RedisData {
   LocalDateTime expireTime;
   Object data;
}
```

## 秒杀
### 自增ID的局限性
> 1.可根据id猜数据量
> 2.不利于分表
