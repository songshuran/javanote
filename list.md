* ArrayList

  * 默认初始化的是`Object[] DEFAULTCAPACITY_EMPTY_ELEMENTDATA = {};` 数组
  * DEFAULT\_CAPACITY 表示数组的初始大小，默认是 10
  * size 表示当前数组的大小，类型 int，没有使用 volatile 修饰，非线程安全的
  * modCount 统计当前数组被修改的版本次数，数组结构有变动，就会 +1。
  * 允许 put null 值，会自动扩容
  * size、isEmpty、get、set、add 等方法时间复杂度都是 O \(1\)
  * 无参构造器初始化时，默认大小是空数组，并不是10，10 是在第一次 add 的时候扩容的数组值
  * 扩容后的大小是原 来容量的 1.5 倍
  * ArrayList 中的数组的最大值是 MAX\_ARRAY\_SIZE = Integer.MAX\_VALUE - 8，超过这个值，JVM 就不会给数组分配 内存空间了。
  * 扩容本质  Arrays.copyOf——&gt; System.arraycopy native方法，把数组指定位置向前或者向后移动，空出的index置null

* LinkedList

  * 链表每个节点我们叫做 Node，Node 有 prev 属性，代表前一个节点的位置，next 属性， 代表后一个节点的位置;
  * first 是双向链表的头节点，它的前一个节点是 null。
  * last 是双向链表的尾节点，它的后一个节点是 null;
  * 当链表中没有数据时，first 和 last 是同一个节点，前后指向都是 null; 
  * 因为是个双向链表，只要机器内存足够强大，是没有大小限制的。
  * get\(index\)
    ```java
      Node<E> node(int index) {
          // 如果 index 处于队列的前半部分，从头开始找，size >> 1 是 size 除以 2 的意思。
          if (index < (size >> 1)) {
              Node<E> x = first;
              for (int i = 0; i < index; i++)
                  x = x.next;
              return x;
          } else {
          // 如果 index 处于队列的后半部分，从尾开始找
              Node<E> x = last;
              for (int i = size - 1; i > index; i--)
                  x = x.prev;
              return x;
          }
      }
    ```

* 常见问题：

  * 数组初始化，加入一个值，然后使用addAll方法，一下子add15个值，那么数组最终大小多少

    ```java
    private void grow(int minCapacity) {
      // overflow-conscious code
      int oldCapacity = elementData.length;
      int newCapacity = oldCapacity + (oldCapacity >> 1);
      if (newCapacity - minCapacity < 0)
          newCapacity = minCapacity;
      if (newCapacity - MAX_ARRAY_SIZE > 0)
          newCapacity = hugeCapacity(minCapacity);
      // minCapacity is usually close to size, so this is a win:
      elementData = Arrays.copyOf(elementData, newCapacity);
    }
    // 数组在加入一个值后，实际大小是 1，最大可用大小是 10 ，现 在需要一下子加入 15 个值，那我们期望数组的大小值就是 16，此时数组最大可用大小只有 10，明显不够，需要扩容，扩容后的大小是:10 + 10 /2 = 15，这时候发现扩容后的大小仍 然不到我们期望的值 16,

    if (newCapacity - minCapacity < 0)
      newCapacity = minCapacity;

    // 所以最终数组扩容后的大小为 16。
    ```

  * 线程安全的list
    * Collections.synchronizedList\(arrayList\);
    * CopyOnWriteArrayList


- CopyOnWriteArrayList
    - 数据结构同ArrayList底层是数组
    - 加锁ReentrantLock
    - 从原数组中copy出新数组
    - 在新数组上操作，并把新数组赋值给数组容器
    - 解锁
    - 需要线程安全的使用list时使用CopyOnWriteArrayList，通过锁+Volatile+数组拷贝相互配合实现了List的线程安全。数组容器被volatile关键字修饰，保证了数组内存地址被任意线程修改后，都会通知到其他线程。
    - 其他细节问题：
        - add方法中对数组进行加锁后，已经线程安全，为什么还需要对老数组进行拷贝？———— 对数组进行加锁是能保证同一时刻，只有一个线程能对数组进行add，在同单核CPU下的多线程环境下肯定没问题，但多核CPU，如果不通过复制拷贝新建数组，修改原数组容器内存地址的话，是无法触发volatile可见性效果的，那么其他CPU下的线程就无法感知数组原来已经被修改了，就会引发多核CPU下的线程安全问题。假设我们不复制拷贝，而是在原来数组上直接修改值，数组的内存地址就不会变，而数组被 volatile 修饰时，必须当数组的内存地址变更时，才能及时的通知到其他线程，内存地址不变， 仅仅是数组元素值发生变化时，是无法把数组元素值发生变动的事实，通知到其它线程的。
        - 为什么数组结构变动不会抛ConcurrentModificationException异常？———— 每次操作都会产生新的数组，而迭代时，持有的是老数组的引用。
    

