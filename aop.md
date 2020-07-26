- 使用
    ```java
    @Component
    @Aspect
    public class AspectJ {
        /**
         * execution可以定义到方法、入参类型、返回值
         * aop包所有类 所有方法 任意入参(如果要指定入参类型写完整类路径) 任意返回值 任意方法类型
         * pointCut里面写代码不会被执行
         */
        @Pointcut("execution(* top.oitm.practice.aop.*.*(..))")
        public void pointCut(){
        }
        
        @Before("pointCut()")
        public void before(){
            System.out.println("before");
        }  
        
        //第二种
        /**
         * within只能定义到里类
         @Pointcut("within(top.oitm.practice.aop.*)")
         *args 指定参数
         @Pointcut("args(java.lang.String,java.lang.Integer)")
         public void withIn(){
        
         }
         @Before("withIn()")
         public void before(){
         System.out.println("before");
         }
         */
    }
    
    
    @Configuration
    //使用Java @Configuration启用@AspectJ支持，请添加@EnableAspectJAutoProxy注释
    @EnableAspectJAutoProxy 
    @ComponentScan("top.oitm.practice.aop")
    public class AppConfig {
        public static void main(String[] args) {
            AnnotationConfigApplicationContext context = new AnnotationConfigApplicationContext(AppConfig.class);
            IndexDao bean = context.getBean(IndexDao.class);
            bean.query();
        }
    }    
    ```