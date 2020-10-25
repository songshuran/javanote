- Hystrix：（单纯的Hystrix）提供了对于微服务调用状态的监控信息，需要结合spring-boot-actuator模块使用
- 使用
    - 依赖
    ```xml
    <dependency>
        <groupId>org.springframework.boot</groupId>             
        <artifactId>spring-boot-starter-actuator</artifactId>
    </dependency>
    ```
    - 访问 {$host}/actuator/hystrix.stream便可以看见微服务调用的状态信息，在Spring Finchley 版本以前访问路径是/hystrix.stream，如果是Finchley 的话 还得在yml里面加入配置:
    ```yaml
    management:
          endpoints:
            web:
              exposure:
                include: '*'
    ```
    