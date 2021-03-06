- Spring-Cloud-Sleuth：分布式链路跟踪，借用了Google Dapper、 Twitter
Zipkin和 Apache HTrace的设计
    - span(跨度):基本工作单元。 span用一个64位的id唯一标识。除ID外，span还包含其他数据，例如描述、时间 戳事件、键值对的注解(标签)， spanID、span父 ID等。 span被启动和停止时，记录了时间信息。初始化 span 被称为"rootspan"，该 span的 id和 trace的 ID相等。
    - trace(跟踪):一组共享"rootspan"的 span组成的树状结构称为 traceo trac也用一个64位的 ID唯一标识， trace 中的所有 span都共享该 trace的 ID
    - annotation(标注): annotation用来记录事件的存在，其中，核心annotation用来定义请求的开始和结束。 
        - CS( Client sent客户端发送):客户端发起一个请求，该 annotation描述了span的开 始。
        - SR( server Received服务器端接收):服务器端获得请求并准备处理它。如果用 SR减去 CS时间戳，就能得到网 络延迟。c)
        - SS( server sent服务器端发送):该 annotation表明完成请求处理(当响应发回客户端时)。如果用 SS减去 SR 时间戳，就能得到服务器端处理请求所需的时间。
        - CR( Client Received客户端接收): span结束的标识。客户端成功接收到服务器端的响应。如果 CR减去 CS时间 戳，就能得到从客户端发送请求到服务器响应的所需的时间
    ![](/assets/iShot2020-10-26上午06.51.05.png)
    
- sleuth整合Zipkin实现分布式链路跟踪：Zipkin是 Twitter开源的分布式跟踪系统，基于 Dapper的论文设计而来，提供界面来友好分析sleuth的跟踪数据
    - 搭建一个zipkin server（docker）
    - 自己搭建（在zipkin2.7.x以后便不支持自定义服务器需要使用官方的版本或者Docker ）
        - 依赖
        ```xml
        <dependency>
            <groupId>io.zipkin.java</groupId>     
            <artifactId>zipkin-autoconfigure-ui</artifactId>         
            <version>2.8.4</version>
        </dependency>
        <dependency>
            <groupId>io.zipkin.java</groupId>     
            <artifactId>zipkin-server</artifactId> 
            <version>2.8.4</version>
        </dependency>
        ```
        - yaml
        ```yaml
        # 2.7.x以后自己搭建需要加以下配置
        management:
            metrics:
              web: 
                server:
                  autoTimeRequests: false
        ```
        - 启动类
        ```java
        @EnableZipkinServer
        @SpringBootApplication
        public class AppSleuth {
            public static void main(String[] args) {             
                SpringApplication.run(AppSleuth.class);
            } 
        }
        ```
        - 访问
        ```
        http://{host}/zipkin/
        ```
- 微服务整合Zipkin
    - 依赖
    ```xml
     <dependency> 
         <groupId>org.springframework.cloud</groupId> 
         <artifactId>spring-cloud-starter-sleuth</artifactId>
     </dependency>
     <dependency> 
         <groupId>org.springframework.cloud</groupId>     
         <artifactId>spring-cloud-starter-zipkin</artifactId>
     </dependency>
    ```
    - 增加以下配置
    ```yaml
    spring:
        zipkin:
            base-url: http://localhost:9000 #指定Zipkin server地址         
        sleuth:
            sampler:
                probability: 1.0 
                #request采样的数量 默认是0.1 也即是10% 顾名思义 采取10%的请求数据,因为在分布式系统中，数据量可能会非常大，因此采样非常重要。我们示例数据少最好配置为1全采样
    ```

- zipkin server 数据持久化问题
    zipkin重启后我们的分布式链路数据全部清空了。
因为zipkin server 默认数据是存储在内存当中， 所以当你服务重启之后内存自然而然也就清空了,使用Elasticsearch 做数据持久化
    - zipkin server里面引入依赖:
    ```xml
      <dependency>
        <groupId>io.zipkin.java</groupId> 
        <artifactId>zipkin-autoconfigure-storage-elasticsearch-http</artifactId> 
        <version>2.3.1</version>
      </dependency>
    ```
    - yaml
    ```yaml
    zipkin:
        storage:
            type: elasticsearch
            elasticsearch:
              cluster: elasticsearch
              hosts: http://localhost:9200
              index: zipki  
    ```

        
    