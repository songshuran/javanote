- springboot启动流程
    - spring ApplicationEvent ——> java事件模型 ——> 观察者设计模式
- @Autowired的装配实现
    - AutowiredAnnotationBeanPostProcessor#postProcessPropertyValues方法中，首先现根据类型进行找，如果找到两个，那么会继续根据属性名字找，找到就不注入进去，存在多个根据名字如果没找到就会抛异常。