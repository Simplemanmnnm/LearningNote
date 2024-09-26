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

### 全局唯一ID
26 epo
### 根据用户id加锁
synchronized(userId.toString().intern()) // intern 去常量池找到此字符串 返回地址
集群模式下无法做到线程安全 -> redis分布式锁

### @Transactional方法直接调用无法触发事务
必须由spring管理注入的对象来调用
```
AopContext.currentProxy() // 当前对象的代理对象，取代this(由spring管理注入的对象)
```
### 布尔类型避免自动拆箱
```
Boolean flag;
return Boolean.TRUE.equals(flag);
```
### 自定义redis锁操作类
```
上锁tryLock：利用setnx，成功返回true
放弃锁unlock：删除Key
```
### 业务流程过久导致锁超时释放
会出现线程安全问题
```
A 获取锁
A 业务阻塞，锁超时消失
B 获取锁
A 业务完成，删除锁
C 获取锁
此时B C同时拿到了锁并进行业务处理
解决：获取锁存入一个UUID，删除锁时对比UUID，若不一致，则不删除
疑问：那A在锁超时后仍然在执行业务逻辑，岂不是也出现了线程安全问题
```
### 判断锁一致后准备删除锁，此时阻塞，导致线程安全问题
如果碰巧发生了full GC - -
```
解决：保证判断锁一致 + 释放锁 操作的原子性 -> LUA脚本
```
### LUA脚本
通过redisTemplate类中的execute命令调用执行
#### Redis调用函数
```
redis.call('命令名称', '参数', ....)
redis.call('set', 'key', 'value')
local name = redis.call('get', 'key')
...
```
#### redis调用脚本 - EVAL
```
      脚本内容                               key类型参数格式
EVAL "return redis.call('set', KEYS[1], ARGV[1])" 1 name Rose
KEYS ARGV获取参数
```
### 仍然存在的问题
1. 不可重入 A 获取锁 调用方法A.method 再次获取锁
2. 不可重试 只尝试获取一次锁，不能多试几次
3. 超时释放 导致线程安全问题
4. redis主从一致性
**都可以通过Redisson框架解决**
## Redisson
> redis基础上实现的分布式工具集合
> 默认锁超时释放时间30s
### maven依赖
```
<groupId>org.redisson</groupId>
<artifactId>redisson</artifactId>
```
### 可重入锁原理
```
底层是通过LUA脚本实现
使用hash结构，多一个字段，保存重入次数
获取锁时若field相同则重入次数加一
释放锁时 重入计数减一，若为0 则删除
```
### 锁重试机制
```
redisson tryLock获取锁成功返回null，获取失败返回锁的ttl
在最大重试等待时间内，订阅锁的释放发布通知(publish),通过future的await方法实现
```
### 看门狗锁延时机制
```
首先已拿到锁 RedissonLock有一个静态Map EXPIRATION_RENEWAL_MAP 保存了当前已获取到的锁 key->锁名 value->entry
entry保存了firt->线程id second->延时任务对象(递归新建此延时任务，每延时一次新建一个任务，每次second保存最新的任务)
每隔 锁释放时间/3 去刷新锁持续时间
```
### 分布式锁主从一致性问题
```
解决：多个redis都作主节点，获取锁需要在所有redis都拿到锁，就能保证不会因为主从一致性导致其他线程获取到锁
```
### 秒杀的性能优化
```
性能问题：多次的串行数据库操作，查询、写入 + 分布式锁
解决：分成两个角色
角色1：秒杀资格判断->库存、一人一单 -> 优化：交给redis LUA
       库存：String 已购买过的用户id：set
角色2：库存、订单操作 -> 消息队列
```
### redis List实现消息队列
```
优点：
redis持久化
消息有序 先进先出
缺点：
无法实现一条消息多个消费者
消息丢失：消息被服务拿走后，服务宕机 消息丢失
```
### redis 基于PubSub的消息队列
```
优点：发布订阅，多生产、多消费
缺点：
无法持久化
消息丢失
消息堆积有上限
```
### 基于Stream数据类型的消息队列
```
优点：
消息可回溯
一条消息可多消费者
可以阻塞读取
缺点：
有漏读风险 只能读到最后一条 -> 可避免

常用命令
XADD：往队列里新增消息，可新建队列
XADD queue_name * name jack age 21 (队列名queue_name，不存在则新建，*代表ID由redis自动生成，消息内容为{name=jack, age=21}，返回消息ID)
XLEN：消息队列长度
XREAD：读取消息，读取并不会删除队列中的消息
```
#### stream消费者组
```
作用：
1.消息分流给组内不同消费者
2.标识最后被处理的消息，避免漏读，所有消息都被消费
3.消息确认机制，消费者获取消息并通过XACK命令确认，消息才会被标记为已处理，避免消息拿到却没被消费掉

命令：
创建消费者组：
XGROUP CREATE
从消费者组读取消息：
XREADGROUP
```
