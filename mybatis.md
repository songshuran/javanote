- 缓存
    - mybatis，一级缓存在和spring整合的时候会失效，原因？
        - mybatis的一级缓存是根据session来缓存的，和spring整合后，session交给了spring管理，spring再查询完毕后，会关闭session，所以一级缓存失效。
    - 二级缓存在mapper中加上注解@CacheNamespace。mybatis会根据命名空间（也就是类名），根据类来进行缓存，落文件，如果查询和变更在同一个类里，此时查询——>变更——>再查询是变更后的结果。如果查询在A类，变更在B类，此时，调用A查询——>B变更——>A查询，发现查询的结果并非B变更后的，因为命名空间是不同的，在B变更的时候，并不会去更新A命名空间的内容
- 默认使用的日志框架是slf4j
    - mybatis + spring5.x + log4j  不能打印日志
    - mybatis + log4j  能打印日志
    - 原因？因为springjcl 内部是jul，引入spring在下面第二行代码就实例化了，实例化了log对象。如果没有引入spring，走到下面第四行代码才实例化log
    ```java
    tryImplementation(LogFactory::useSlf4jLogging);
    tryImplementation(LogFactory::useCommonsLogging);
    tryImplementation(LogFactory::useLog4J2Logging);
    tryImplementation(LogFactory::useLog4JLogging);
    tryImplementation(LogFactory::useJdkLogging);
    tryImplementation(LogFactory::useNoLogging);
    ```
- 一级缓存失效原理？
    - mybatis —— sqlSession —— defaultSqlSession —— select —— 执行sql。使用的是defaultSqlSession所以缓存还在。
    - spring —— sqlSession —— SQLSessionTemplate —— select —— sqlSessionProxy.select —— 执行sql —— 关闭session
    ```java
    //SqlSessionTemplate.SqlSessionInterceptor.invoke中最后部分代码
     finally {
        if (sqlSession != null) {
          closeSqlSession(sqlSession, SqlSessionTemplate.this.sqlSessionFactory);
        }
      }
    ```
    
- mybatis和spring是如何整合的？
    - MapperScan扫描mapper所对应的BeanDefinition
    - 把mapper变成FactoryBean，MapperFactoryBean
    - 为BeanDefinition添加一个构造方法的值，因为mybatis的Mapper FactoryBean有一个有参构造方法，spring实例化时需要一个构造方法的值
    - mybatis利用spring的afterPropertiesSet方法来完成对mapper信息的初始化，比如注解的sql语句初始化。
    - 当一个MapperFactoryBean被实例化了之后，MapperFactoryBean内部实现了InitialBean接口afterPropertiesSet方法，afterPropertiesSet中调用了子类的checkDaoConfig()方法，checkDaoConfig中有configuration.addMapper(this.mapperInterface)，得到接口中所有的方法，判断方法中的有没有加mybatis的相关curd注解，然后把相关sql存放到 全类名+方法名 作为key 放到mapperedStatements的map中。在服务调用的时候从该map中取出sql进行执行。
    
    
    
    
    
    