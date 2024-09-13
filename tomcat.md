# tomcat
## HTTP工作流程
首先用户发送请求触发HTTP工作
1、浏览器 发起建立TCP链接请求，三次握手建立
2、浏览器 生成HTTP格式数据包 发送给 服务端
3、服务端 解析HTTP格式数据包 执行请求 生成HTTP格式数据包 发送给 浏览器
4、浏览器 解析HTTP格式数据包 呈现给 用户

### web服务器的处理流程
方案一：由HTTP服务器 直接调用 业务类
   缺点：http服务器需要写一堆的判断分支，根据请求路径转发到不同的业务类
方案二：服务器先把请求封装成ServletRequest格式，交给Servlet容器处理，通过请求地址和servlet的映射找到业务类
       (封装是由Coyote实现的)
  疑问：
      请求地址和servlet的映射具体怎么实现的？
          Mapper对象，其中存储路径和Wrapper（就是servlet）的映射关系，
      为什么需要servlet这个东西？直接建立一个路径和服务类的映射不行吗？
          servlet作用：不只是负责映射，还负责解析发来的、生成返回的ServletRequest

## 链接器 - Coyote
Coyote是Tomcat的连接器框架的名称
供客户端访问，客户端发送请求被Coyote接收到，负责建立链接、响应结果
将客户端发来的请求封装，转发给Catalina(servlet容器)

Coyote = EndPoint + Processor + Adapter
ProtocolHandler = EndPoint + Processor 在tomcat中实际上EndPoint和Processor是合并的
### EndPoint
通信端点，作用：监听请求，接受Socket请求，发送给Porcessor

### Processor
将socket请求转为HTTP请求,解析字节流成Request、Response对象

### Adapter
将Request转化成servletRequest

## servlet容器 Catalina
=server = n*service
service = n*connector + container*1

### container
包含一个Engine
疑问：既然只包含一个engine，为什么要多这一层呢？
     扩展性，虽然目前engine是唯一顶层容器，但是后续可能会扩展，如果扩展只需要实现container接口即可，container里定义了各个容器(engine、context、host)通用的方法，不同的容器可以通过相同的方法名来管理和操作
#### Engine
包含多个Host
##### Host
虚拟站点，包含多个Context
###### Context
就是web应用程序，一个项目就是一个context，包含多个Wrapper（就是servlet）
###### Wrapper
就是servlet，tomcat中最底层的容器

## Tomcat调用流程
connector 监听端口 socket形式的http请求
接收到请求后，调用protocolHandler将请求信息转化成request response
交给coyoteAdapter,将request response转化成servletRequest servletResponse
coyoteAdapter调用engine，层层调用找到servlet服务类
服务类将结果写入response。返回connector
connector返回客户端
