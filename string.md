- public final class String （如果有需要类似String不可变的对象，可参考String类的设计)
    - 不可变 因为字符都存在 private final char value[];中

- 因为String  不可变，所以调用str.replace\(\)方法时必须要用一个变量接收

- 字符串转字节数组，字节数组转字符串指定编码
    ```java
    byte[] bytes = str.getBytes(StandardCharsets.UTF_8);
    String s = new String(bytes, StandardCharsets.UTF_8);
    ```

- 字符串截取
    ```java
    // substring(int beginIndex, int endIndex)从beginIndex截取到endIndex，不包含endIndex
    //substring(int beginIndex) 从 beginIndex截取到str末尾
    str = "World";
    String lowerStr = str.substring(0, 1).toLowerCase() + str.substring(1);
    // lowerStr = world
    ```
- 字符串相等判断哪
    ```java
    String s1 = new String("123");
    String s2 = new String("123");
    String s3 = "123";
    String s4 = "123";
    System.out.println(s1 == s2);                 //false
    System.out.println(Objects.equals(s1, s2));   //true
    System.out.println(s1 == s3);                 //false
    System.out.println(s4 == s3);                 //true
    ```    

    



