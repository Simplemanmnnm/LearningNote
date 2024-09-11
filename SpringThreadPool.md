# SpringThreadPool
## 线程池容器类
@Configuration
@EnableAsync (也可以加到启动类上，作用是开启多线程调用，让@Async注解方法异步执行)

@Bean(name = "${线程池名}") 修饰线程池生成方法
ThreadPoolTaskExecutor
setCorePoolSize
setMaxPoolSize
setQueueCapacity
setThreadNamePrefix
setRejectedExecutionHandler(new ThreadPoolExecutor.DiscardPolicy())
initialize

## 包含异步方法的类
@Component修饰 (将此类交给Spring管理)
@Async("${线程池名}") 修饰异步方法
