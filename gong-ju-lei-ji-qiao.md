- 工具类通用特征
    - 构造器必须私有，这样工具类就不能被初始化，因为工具类使用时也无需初始化，直接使用即可，就不用开放构造器
    - 工具类的工具方法必须是static final关键字修饰。保证方法不可变
    
- Arrays
    - sort方法主要用于排序，入参时各种类型的数组，也支持自定义类型数组，该方法性能比较高，使用了双轴快速排序算法
    - binarySearch：二分查找法快速查找数组中的值。如果倍速搜的数组是无序的则需要先排序，否则可能搜索不到对应值
    - copyOfRange：数组部分copy，实际上底层调用的是 System.arraycopy 这个 native 方法，若对底层拷贝方法比较熟悉的话，也可以直接使用。

- Collections
    - List初始化后不能修改，可使用`Collections.unmodifiableList(list)`