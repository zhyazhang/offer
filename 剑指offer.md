# 剑指offer

## 第一章 面试的流程

#### 1.3.2 技术面试环节

> 面试提示：
>
> 面试官除了希望应聘者的代码能够完成基本的功能，还会关注应聘者是否考虑了边界条件、特殊输入(如空指针、空字符串等)及错误处理。



## 第二章 面试需要的基础知识

### 2.2 编程语言

通常语言面试有3种类型：

1. 面试官直接询问应聘者对语言感念的理解
2. 面试官拿出事先准备好的代码，让应聘者分析代码的执行结果，这种题型选择的代码通常包含比较复杂的语言特性。
3. 要求应聘者写代码定义一个类型或者实现类型中的成员函数。

#### 面试题2:实现SIngleton模式

**1.单例模式的定义**

**单例模式**确保某个类只有一个实例，而且自行实例化并向整个系统提供这个实例。在计算机系统中，线程池、缓存、日志对象、对话框、打印机、显卡的驱动程序对象常被设计成单例。这些应用都或多或少具有资源管理器的功能。每台计算机可以有若干个打印机，但只能有一个Printer Spooler，以避免两个打印作业同时输出到打印机中。每台计算机可以有若干通信端口，系统应当集中管理这些通信端口，以避免一个通信端口同时被两个请求同时调用。总之，选择单例模式就是为了避免不一致状态，避免政出多头。

**2.单例模式的特点**

- 单例类只能有一个实例
- 单例类必须自己创建自己的唯一实例
- 单例类必须给所有其他对象提供这一实例

单例模式保证了全剧对象的唯一性，比如系统启动读取配置文件就需要单例保证配置的一致性。

**3.线程安全问题**

一方面在获取单例的时候，要保证不能产生多个实例对象，后面会详细讲到五种实现方式；
另一方面，在使用单例对象的时候，要注意单例对象内的实例变量是会被多线程共享的，推荐使用无状态的对象，不会因为多个线程的交替调度而破坏自身状态导致线程安全问题，比如我们常用的VO，DTO等（局部变量是在用户栈中的，而且用户栈本身就是线程私有的内存区域，所以不存在线程安全问题）。

**4.实现单例模式的八种方式**

4.1饿汉式(静态常量) ==可用==

优点：实现简单，在类装载的时候完成实例化，避免了线程同步问题

缺点：在类装载的时候实例化，没有达到Lazy Loading的效果，==如果从始至终从未使用过这个实例，会造成内存浪费==

```java
public class Singleton{
  private final static Singleton INSTANCE = new Singleton();
  private Singleton(){}
  
  public static SIngleton getInstance(){
    return INSTANCE;
  }
}

```

4.2饿汉式(静态代码块)==可用==

这种方式和上面的类似，只不过将类实例化的过程放在了静态代码块中，也是在类封装的时候就执行静态代码块中的代码，初始化类的实例。优缺点和上面是一样的。

```java
public class Singleton{
  private static Singleton instance;
  
  static{
    instance = new Singleton();
  }
  
  private Singleton(){}
  
  public static Singleton getInstance(){
    return instance;
  }
}
```

4.3懒汉式(线程不安全)==不可用==

这种写法起到了Lazy Loading的效果，但是只能在单线程下使用。如果在多线程下，一个线程进入了if (singleton == null)判断语句块，还未来得及往下执行，另一个线程也通过了这个判断语句，这时便会产生多个实例。所以在多线程环境下不可使用这种方式。



```java
public class Singleton{
  private static Singleton singleton;
  
  private Singleton(){}
  
  public static Singleton getInstance(){
    if(singleton == null){
      singleton = new Singleton();
    }
    return singleton;
  }
}
```

4.4 懒汉式(线程安全，同步方法)==不推荐使用==

解决上面第三种实现方式的线程不安全问题，做个线程同步就可以了，于是就对getInstance()方法进行了线程同步。
缺点：效率太低了，每个线程在想获得类的实例时候，执行getInstance()方法都要进行同步。而其实这个方法只执行一次实例化代码就够了，后面的想获得该类实例，直接return就行了。方法进行同步效率太低要改进。

```java
public class Singleton{
  private static Singleton singleton;
  
  private Singleton(){}
  
  public static synchronized Singleton getInstance(){
    if(singleton == null){
      singleton = new Singleton();
    }
    return singleton;
  }
}
```

4.5懒汉式(线程安全，同步代码块)==不可用==
由于第四种实现方式同步效率太低，所以摒弃同步方法，改为同步产生实例化的的代码块。但是这种同步并不能起到线程同步的作用。跟第3种实现方式遇到的情形一致，假如一个线程进入了if (singleton == null)判断语句块，还未来得及往下执行，另一个线程也通过了这个判断语句，这时便会产生多个实例。

```java
public class Singleton{
  private static Singleton singleton;
  
  private Singleton(){}
  
  public static Singleton getInstance(){
    if(singleton == null){
      synchronized(Singleton.class){
        singleton = new Singleton();
      }
    }
    return singleton;
  }
}
```

4.6双重检查==推荐使用==

Double-Check概念对于多线程开发者来说不会陌生，如代码中所示，我们进行了两次if (singleton == null)检查，这样就可以保证线程安全了。这样，实例化代码只用执行一次，后面再次访问时，判断if (singleton == null)，直接return实例化对象。
优点：线程安全；延迟加载；效率较高

```java
public class Singleton{
  
  private static volatile SIngleton singleton;
  
  private Singleton(){}
  
  public static Singleton getInstance(){
    if(singleton == null){
      synchronized(Singleton.class){
        if(singleton == null){
          singleton = new Singleton();
        }
      }
    }
    return singleton;
  }
}
```

4.7静态内部类==推荐使用==
这种方式跟饿汉式方式采用的机制类似，但又有不同。两者都是采用了类装载的机制来保证初始化实例时只有一个线程。不同的地方在饿汉式方式是只要Singleton类被装载就会实例化，没有Lazy-Loading的作用，而静态内部类方式在Singleton类被装载时并不会立即实例化，而是在需要实例化时，调用getInstance方法，才会装载SingletonInstance类，从而完成Singleton的实例化。
类的静态属性只会在第一次加载类的时候初始化，所以在这里，JVM帮助我们保证了线程的安全性，在类进行初始化时，别的线程是无法进入的。
优点：避免了线程不安全，延迟加载，效率高。

```java
public class Singleton{
  private Singleton(){}
  
  private static class SingletonInstance{
    private static final Singleton INSTANCE = new Singleton();   
  }
  
  public static Singleton getInstance(){
    return SingletonInstance.INSTANCE;
  }
}
```

4.8枚举==推荐使用==
借助JDK1.5中添加的枚举来实现单例模式。不仅能避免多线程同步问题，而且还能防止反序列化重新创建新的对象

```java
public enum Singleton{
  INSTANCE;
  public void whatevermethod(){
    
  }
}
```



单例模式的优点
系统内存中该类只存在一个对象，节省了系统资源，对于一些需要频繁创建销毁的对象，使用单例模式可以提高系统性能。

单例模式的缺点
当想实例化一个单例类的时候，必须要记住使用相应的获取对象的方法，而不是使用new，可能会给其他开发人员造成困扰，特别是看不到源码的时候。

单例模式的使用场景

• 需要频繁的进行创建和销毁的对象；

• 创建对象时耗时过多或耗费资源过多，但又经常用到的对象；

• 工具类对象；

• 频繁访问数据库或文件的对象。

#### 面试题3:数组中重复的数字

##### 题目1，找出数组中重复的数字

> 在一个长度为n的数组里的所有数字都在0～n-1的范围内。数组中某些数字是重复的，但不知道有几个数字重复了，也不知道每个数字重复了几次，请找出数组中任意一个重复的数字，例如，如果输入长度为7的数组{2,3,1,0,2,5,3}，那么对应的输出是重复的数字2或者3.

`方法1`：排序

先把输入的数组排序，从排序的数组中找出重复的数字是一件很容易的事情，只需要从头到尾扫描排序后的数组就可以了。排序一个长度`n`的数组需要$O(nlogn)$的时间。

`方法2:`哈希表

从头到尾按顺序扫描数组的每个数字，没扫描到一个数字的时候，都可以用$O(1)$，的时间来判断哈希表里是否已经包含了该数字。如果哈希表里还没有这个数字，就把它加入哈希表。如果哈希表里已经存在该数字，就找到了一个重复的数字。这个算法的时间复杂度是$O(n)$，但他提高时间效率是一个大小为$O(n)$的哈希表为代价。

`方法3:`

从头到位依次扫描这个数组中的每个数字。当扫描到下表为i的数字时，首先比较这个数字(m)是不是等于i。如果是，则接着扫描下一个数字；如果不是，则再拿它和第m个数字进行比较。如果它和第m个数字相等，就找到了一个重复的数字；如果它和第m个数字不相等，就把第i个数字和第m个数字交换，把m放到属于它的位置，接下来重复这个比较、交换的过程，直到发现一个重复的数字。

```java
public Solution{
  public int findRepeatNumber(int[] nums){
    int i = 0;
    while(i<nums.length){
      if(nums[i] == i){
        i++;
        continue;
      }
      if(nums[nums[i]] == nums[i]){
        return nums[i];
      }
      int tmp = nums[i];
      nums[i]=nums[tmp];
      nums[tmp]=tmp;    
    }
    return -1; 
  }
}
```

##### 题目2，不修改数组找出重复的数字

> 在一个长度为n+1的数组里的所有数字都在1～n的范围内，所有数组中至少有一个数字是重复的，请找出数组中任意一个重复的数字，但不能修改输入的数组。

`方法1`，可以创建一个长度为n+1的辅助数组，然后逐一把原数组的数字复制到辅助数组，如果原数组中被复制的数字是m，则把它复制到辅助数组中下标为m的位置，该方案需要$O(n)$的辅助空间。

`方法2，`把从1～n的数字从中间的数字m分为两部分，前面一半为1～m，后面一半为m+1～n。如果1～m的数字的数目超过m，那么这一半的区间里一定包含重复的数字；否则，另一半m+1～n的区间里一定包含重复的数字。然后继续把包含重复数字的区间一分为二，直到找到一个重复数字。这个过程和二分查找算法类似，只是多了一步统计区间里数字的数目。空间复杂度$O(1)$，时间复杂度$O(n)$，以时间换空间。

```java
    public int findRepeatNumber(int[] nums) {

        if (nums.length <= 0) {
            return -1;
        }

        int start = 1;
        int end = nums.length - 1;

        while (end >= start) {
            int middle = ((end - start) >> 1) + start;
            int count = countRange(nums, nums.length, start, middle);

            if (end == start) {
                if (count > 1) {
                    return start;
                } else {
                    break;
                }
            }

            if (count > (middle - start + 1)) {
                end = middle;
            } else {
                start = middle + 1;
            }
        }
        return -1;
    }

    private int countRange(int[] nums, int length, int start, int end) {

        if (nums.length == 0) {
            return 0;
        }
        int count = 0;
        for (int i = 0; i < length; i++) {
            if (nums[i] >= start && nums[i] <= end) {
                ++count;
            }
        }
        return count;
    }
```

#### 面试题4:二维数组中的查找

> 在一个二维数组中，每一行都按照从左到右递增的顺序排序，每一列都按照从上到下递增的顺序排序。请完成一个函数，输入这样的一个二维数组和一个证书，判断数组中是否含有该整数。给定 target = `5`，返回 `true`。给定 target = `20`，返回 `false`。
>
> [
>
>  [1,   4,  7, 11, 15],
>
> [2,   5,  8, 12, 19],
>
> [3,   6,  9, 16, 22],
>
> [10, 13, 14, 17, 24],
>
> [18, 21, 23, 26, 30]
>
> ]

#### 面试题5:替换空格

> 实现一个函数，把字符串中的每个空格替换成%20，例如输入“We are happy.”，则输出“We%20are%20happy.”





#### 面试题6:从尾到头打印链表

> 输入一个链表的头节点，从尾到头反过来打印出每个节点的值。

### 2.3.4 树

面试的时候提到的树，大部分是二叉树，所谓二叉树是树的一种特殊结构，在二叉树中每个节点最多只能有两个字节点，在二叉树中最重要的操作，是遍历，按照某一顺序访问树中的所有节点。通常有三种便利方式：

1. 前序遍历：先访问根节点，再访问左子树，最后访问右子树
2. 中序遍历：先访问左子树，再访问跟节点，最后访问右子树
3. 后序遍历：先访问左子节点，再访问右子节点，最后访问根节点

`宽度优先搜索`:先访问树的第一层节点，再访问树的第二层节点，直到访问到最下面一层节点。再同一层节点中，以从左到右的顺序依次访问。

二叉树有很多特例，==二叉搜索树==中，左子节点总是小于或等于跟节点，而右子节点总是大于或等于跟节点。可以在$O(logn)$的时间内根据数值在二叉搜索树中找到一个节点。

二叉树的另外两个特例是==堆==和==红黑树==。堆分为最大堆和最小堆，在最大堆中跟节点的值最大，在最小堆中根节点的值最小。==有许多需要快速找到最大值或最小值的问题都可以用堆来解决。==红黑树是把书中的节点定义为红、黑两种颜色，并通过规则确保从根节点到叶节点的最长路径的长度不超过最短路径的两倍。















