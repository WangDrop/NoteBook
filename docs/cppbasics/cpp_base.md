# 编程语言C++

* C++: volatile static const extern等关键字 
* 常用库函数实现
    * malloc,strcpy,strcmp的实现，常用库函数实现
    strcpy

```
int32_t strcmp(const char* s1, const char* s2)
{
    assert(s1 != NULL && s2 != NULL);
    int32_t ret = 0;
    while (!(ret = *((unsigned char* )s1++) - *((unsigned char*)s2++)) == 0 && *s1);
    if (ret < 0)
    {
        return -1;
    }
    else if (ret > 0)
    {
        return 1;
    }
    return 0;
}

char * strcpy(char *dst,const char *src)   //[1]
{
    assert(dst != NULL && src != NULL);    //[2]
    char *ret = dst;  //[3]
    while ((*dst++=*src++)!='\0'); //[4]
    return ret;
}
```
* 虚函数的作用和实现原理，什么是虚函数,有什么作用?
    * 纯虚函数，为什么需要纯虚函数？
    * 为什么需要虚析构函数,什么时候不需要?父类的析构函数为什么要定义为虚函数?
    * 内联函数、构造函数、静态成员函数可以是虚函数吗?
        * 当使用非多态调用时，编译器可选择内联，构造函数和静态成员函数都不可以
    * 构造函数中可以调用虚函数吗?
        * 可以，但是构造函数中的base的虚函数不会下降到derived上。而是直接调用base类的虚函数。绝不在构造和析构函数中调用virtual函数
    * 为什么需要虚继承?虚继承实现原理解析，
        * 减少空间浪费，解决菱形继承问题

* decltype 使用

```cpp
// 尾置返回允许我们在参数列表之后声明返回类型
template <typename It>
auto fcn(It beg, It end) -> decltype(*beg) {
    return *beg;    // 返回序列中一个元素的引用
}
// 为了使用模板参数成员，必须用 typename
template <typename It>
auto fcn2(It beg, It end) -> typename remove_reference<decltype(*beg)>::type {
    return *beg;    // 返回序列中一个元素的拷贝
}
```

* 引用
    * 左值引用: 常规引用，一般表示对象的身份。
    * 右值引用: 右值引用就是必须绑定到右值（一个临时对象、将要销毁的对象）的引用，一般表示对象的值。
    * 右值引用可实现转移语义（Move Sementics）和精确传递（Perfect Forwarding），它的主要目的有两个方面：
        * 消除两个对象交互时不必要的对象拷贝，节省运算存储资源，提高效率。
        * 能够更简洁明确地定义泛型函数。

* 引用折叠
    * `X& &`、`X& &&`、`X&& &` 可折叠成 `X&`
    * `X&& &&` 可折叠成 `X&&`

* vptr与vtable
    * 虚函数指针：在含有虚函数类的对象中，指向虚函数表，在运行时确定。
    * 虚函数表：在程序只读数据段（`.rodata section`，见：[目标文件存储结构](#%E7%9B%AE%E6%A0%87%E6%96%87%E4%BB%B6%E5%AD%98%E5%82%A8%E7%BB%93%E6%9E%84)），存放虚函数指针，如果派生类实现了基类的某个虚函数，则在虚表中覆盖原本基类的那个虚函数指针，在编译时根据类的声明创建。

> [C++中的虚函数(表)实现机制以及用C语言对其进行的模拟实现](https://blog.twofei.com/496/

* 虚继承
    * 虚继承用于解决多继承条件下的菱形继承问题（浪费存储空间、存在二义性）。
        * 底层实现原理与编译器相关，一般通过**虚基类指针**和**虚基类表**实现，每个虚继承的子类都有一个虚基类指针（占用一个指针的存储空间，4字节）和虚基类表（不占用类对象的存储空间）（需要强调的是，虚基类依旧会在子类里面存在拷贝，只是仅仅最多存在一份而已，并不是不在子类里面了）；当虚继承的子类被当做父类继承时，虚基类指针也会被继承。
        * 实际上，vbptr 指的是虚基类表指针（virtual base table pointer），该指针指向了一个虚基类表（virtual table），虚表中记录了虚基类与本类的偏移地址；通过偏移地址，这样就找到了虚基类成员，而虚继承也不用像普通多继承那样维持着公共基类（虚基类）的两份同样的拷贝，节省了存储空间。

* 模板类、成员模板、虚函数
    * 模板类中可以使用虚函数
    * 一个类（无论是普通类还是类模板）的成员模板（本身是模板的成员函数）不能是虚函

* C 实现 C++ 类
    * 封装：使用函数指针把属性与方法封装到结构体中
    * 继承：结构体嵌套
    * 多态：父类与子类方法的函数指针不同

* C++ 内存分配机制
    * 栈，堆，BSS（未初始化的全局变量），代码段（不可写），数据段（已初始化的全局及静态变量）,栈向下增长，堆向上增长
* 指针
    * C++的智能指针及其原理
        * weak_ptr设计之初就是为了服务于shared_ptr的，所以不增加引用计数就是它的核心功能，解决了shared_ptr的循环引用问题
        * Class shared_ptr 实现共享式拥有（shared ownership）概念。多个智能指针指向相同对象，该对象和其相关资源会在 “最后一个 reference 被销毁” 时被释放。为了在结构较复杂的情景中执行上述工作，标准库提供 weak_ptr、bad_weak_ptr 和 enable_shared_from_this 等辅助类。
        * Class unique_ptr 实现独占式拥有（exclusive ownership）或严格拥有（strict ownership）概念，保证同一时间内只有一个智能指针可以指向该对象。你可以移交拥有权。它对于避免内存泄漏（resource leak）——如 new 后忘记 delete ——特别有用。
        * 支持定制型删除器（custom deleter），可防范 Cross-DLL 问题（对象在动态链接库（DLL）中被 new 创建，却在另一个 DLL 内被 delete 销毁）、自动解除互斥锁
        * weak_ptr 允许你共享但不拥有某对象，一旦最末一个拥有该对象的智能指针失去了所有权，任何 weak_ptr 都会自动成空（empty）。因此，在 default 和 copy 构造函数之外，weak_ptr 只提供 “接受一个 shared_ptr” 的构造函数。
        * auto_ptr 与 unique_ptr 比较
            1. auto_ptr 可以赋值拷贝，复制拷贝后所有权转移；unqiue_ptr 无拷贝赋值语义，但实现了`move` 语义；
            2. auto_ptr 对象不能管理数组（析构调用 `delete`），unique_ptr 可以管理数组（析构调用 `delete[]` ）；
        * boost::bind会把shared_ptr指针拷贝一份，无形延长了对象的生存周期，要注意可能导致的内存泄漏问题，这种问题同样可以提供bind一个weak_ptr来解决
        * shared_ptr有一定的拷贝开销，所以在一个线程最外层函数里面只要有一个shared_ptr，网内存传第时候都可以传递const reference
        * shared_ptr<void>可以持有任何对象，并且都能安全的释放
        * 当对后一个指向对象的shared_ptr析构，对象会在同一个线程析构，如果对象析构耗时，可以考虑放到单独线程处理，通过一个BlockQueue<shared_ptr>
        * 通过的做法是：owner包含child的shared_ptr，child持有owner的weak_ptr



* 写string类的构造，析构，拷贝函数

```
class String {
public:
    String(const char *str=NULL);//普通构造函数
    String(const String &str);//拷贝构造函数
    String & operator =(const String &str);//赋值函数
    ~String();//析构函数
 
private:
    char* m_data;//用于保存字符串
};
 
//普通构造函数
String::String(const char *str) {
    if (str==NULL) {
        m_data=new char[1]; //对空字符串自动申请存放结束标志'\0'的空间
        if (m_data==NULL) {//内存是否申请成功
            std::cout<<"申请内存失败！"<<std::endl;
            exit(1);
        }
        m_data[0]='\0';
    } else {
        m_data=new char[strlen(str)+1];
        if (m_data==NULL) {//内存是否申请成功
            std::cout<<"申请内存失败！"<<std::endl;
            exit(1);
        }
        strcpy(m_data,str);
    }
}
 
//拷贝构造函数
String::String(const String &other) { //输入参数为const型
    m_data=new char[strlen(other.m_data)+1];
    if (m_data==NULL) {//内存是否申请成功
        std::cout<<"申请内存失败！"<<std::endl;
        exit(1);
    }
    strcpy(m_data,other.m_data);
}
 
//赋值函数
String& String::operator =(const String &other) {//输入参数为const型
    if (this == &other) { return *this; }
    delete [] m_data;//释放原来的内存资源
    m_data= new char[strlen(other.m_data)+1];
    if (m_data==NULL) {//内存是否申请成功
        std::cout<<"申请内存失败！"<<std::endl;
        exit(1);
    }
    strcpy(m_data,other.m_data);
    return *this;//返回本对象的引用
}
 
//析构函数
String::~String() {
    delete [] m_data;
}
```
* delete this 合法吗？
    > [Is it legal (and moral) for a member function to say delete this?](https://isocpp.org/wiki/faq/freestore-mgmt#delete-this)
    1. 必须保证 this 对象是通过 `new`（不是 `new[]`、不是 placement new、不是栈上、不是全局、不是其他对象成员）分配的
    2. 必须保证调用 `delete this` 的成员函数是最后一个调用 this 的成员函数
    3. 必须保证成员函数的 `delete this ` 后面没有调用 this 了
    4. 必须保证 `delete this` 后没有人使用了
* STL

容器 | 底层数据结构 | 时间复杂度 | 有无序 | 可不可重复 | 其他
---|---|---|---|---|---
[array](https://github.com/huihut/interview/tree/master/STL#array)|数组|随机读改 O(1)|无序|可重复|支持随机访问
[vector](https://github.com/huihut/interview/tree/master/STL#vector)|数组|随机读改、尾部插入、尾部删除 O(1)<br/>头部插入、头部删除 O(n)|无序|可重复|支持随机访问
[deque](https://github.com/huihut/interview/tree/master/STL#deque)|双端队列|头尾插入、头尾删除 O(1)|无序|可重复|一个中央控制器 + 多个缓冲区，支持首尾快速增删，支持随机访问
[forward_list](https://github.com/huihut/interview/tree/master/STL#forward_list)|单向链表|插入、删除 O(1)|无序|可重复|不支持随机访问
[list](https://github.com/huihut/interview/tree/master/STL#list)|双向链表|插入、删除 O(1)|无序|可重复|不支持随机访问
[stack](https://github.com/huihut/interview/tree/master/STL#stack)|deque / list|顶部插入、顶部删除 O(1)|无序|可重复|deque 或 list 封闭头端开口，不用 vector 的原因应该是容量大小有限制，扩容耗时
[queue](https://github.com/huihut/interview/tree/master/STL#queue)|deque / list|尾部插入、头部删除 O(1)|无序|可重复|deque 或 list 封闭头端开口，不用 vector 的原因应该是容量大小有限制，扩容耗时
[priority_queue](https://github.com/huihut/interview/tree/master/STL#priority_queue)|vector + max-heap|插入、删除 O(log<sub>2</sub>n)|有序|可重复|vector容器+heap处理规则
[set](https://github.com/huihut/interview/tree/master/STL#set)|红黑树|插入、删除、查找 O(log<sub>2</sub>n)|有序|不可重复|
[multiset](https://github.com/huihut/interview/tree/master/STL#multiset)|红黑树|插入、删除、查找 O(log<sub>2</sub>n)|有序|可重复|
[map](https://github.com/huihut/interview/tree/master/STL#map)|红黑树|插入、删除、查找 O(log<sub>2</sub>n)|有序|不可重复|
[multimap](https://github.com/huihut/interview/tree/master/STL#multimap)|红黑树|插入、删除、查找 O(log<sub>2</sub>n)|有序|可重复|
[unordered_set](https://github.com/huihut/interview/tree/master/STL#unordered_set)|哈希表|插入、删除、查找 O(1) 最差 O(n)|无序|不可重复|
[unordered_multiset](https://github.com/huihut/interview/tree/master/STL#unordered_multiset)|哈希表|插入、删除、查找 O(1) 最差 O(n)|无序|可重复|
[unordered_map](https://github.com/huihut/interview/tree/master/STL#unordered_map)|哈希表|插入、删除、查找 O(1) 最差 O(n)|无序|不可重复|
[unordered_multimap](https://github.com/huihut/interview/tree/master/STL#unordered_multimap)|哈希表|插入、删除、查找 O(1) 最差 O(n)|无序|可重复|

* 面向对象
    * https://github.com/CyC2018/CS-Notes/blob/master/notes/%E9%9D%A2%E5%90%91%E5%AF%B9%E8%B1%A1%E6%80%9D%E6%83%B3.md

* 设计模式
    * https://github.com/CyC2018/CS-Notes/blob/master/notes/%E8%AE%BE%E8%AE%A1%E6%A8%A1%E5%BC%8F%20-%20%E7%9B%AE%E5%BD%95.md

# 数据结构与算法

* [数据结构与算法学习攻略](https://github.com/youngyangyang04/leetcode-master)
    * TODO

* 红黑树

  * 红黑树的特征是什么？
    1. 节点是红色或黑色。
    2. 根是黑色。
    3. 所有叶子都是黑色（叶子是 NIL 节点）。
    4. 每个红色节点必须有两个黑色的子节点。（从每个叶子到根的所有路径上不能有两个连续的红色节点。）（新增节点的父节点必须相同）
    5. 从任一节点到其每个叶子的所有简单路径都包含相同数目的黑色节点。（新增节点必须为红）

  * 调整
    1. 变色
    2. 左旋
    3. 右旋

# 算法
### 排序
排序算法 | 平均时间复杂度 | 最差时间复杂度 | 空间复杂度 | 数据对象稳定性
---|---|---|---|---
[冒泡排序](Algorithm/BubbleSort.h) | O(n<sup>2</sup>)|O(n<sup>2</sup>)|O(1)|稳定
[选择排序](Algorithm/SelectionSort.h) | O(n<sup>2</sup>)|O(n<sup>2</sup>)|O(1)|数组不稳定、链表稳定
[插入排序](Algorithm/InsertSort.h) | O(n<sup>2</sup>)|O(n<sup>2</sup>)|O(1)|稳定
[快速排序](Algorithm/QuickSort.h) | O(n*log<sub>2</sub>n) |  O(n<sup>2</sup>) | O(log<sub>2</sub>n) | 不稳定
[堆排序](Algorithm/HeapSort.cpp) | O(n*log<sub>2</sub>n)|O(n*log<sub>2</sub>n)|O(1)|不稳定
[归并排序](Algorithm/MergeSort.h) | O(n*log<sub>2</sub>n) | O(n*log<sub>2</sub>n)|O(n)|稳定
[希尔排序](Algorithm/ShellSort.h) | O(n*log<sup>2</sup>n)|O(n<sup>2</sup>)|O(1)|不稳定
[计数排序](Algorithm/CountSort.cpp) | O(n+m)|O(n+m)|O(n+m)|稳定
[桶排序](Algorithm/BucketSort.cpp) | O(n)|O(n)|O(m)|稳定
[基数排序](Algorithm/RadixSort.h) | O(k*n)|O(n<sup>2</sup>)| |稳定

> * 均按从小到大排列
> * k：代表数值中的 “数位” 个数
> * n：代表数据规模
> * m：代表数据的最大值减最小值
> * 来自：[wikipedia . 排序算法](https://zh.wikipedia.org/wiki/%E6%8E%92%E5%BA%8F%E7%AE%97%E6%B3%95)

### 查找
查找算法 | 平均时间复杂度 | 空间复杂度 | 查找条件
---|---|---|---
[顺序查找](Algorithm/SequentialSearch.h) | O(n) | O(1) | 无序或有序
[二分查找（折半查找）](Algorithm/BinarySearch.h) | O(log<sub>2</sub>n)| O(1) | 有序
[插值查找](Algorithm/InsertionSearch.h) | O(log<sub>2</sub>(log<sub>2</sub>n)) | O(1) | 有序
[斐波那契查找](Algorithm/FibonacciSearch.cpp) | O(log<sub>2</sub>n) | O(1) | 有序
[哈希查找](DataStructure/HashTable.cpp) | O(1) | O(n) | 无序或有序
[二叉查找树（二叉搜索树查找）](Algorithm/BSTSearch.h) |O(log<sub>2</sub>n) |   | 
[红黑树](DataStructure/RedBlackTree.cpp) |O(log<sub>2</sub>n) | |
2-3树 | O(log<sub>2</sub>n - log<sub>3</sub>n) |   | 
B树/B+树 |O(log<sub>2</sub>n) |   | 

### 图搜索算法
图搜索算法 |数据结构| 遍历时间复杂度 | 空间复杂度
---|---|---|---
[BFS广度优先搜索](https://zh.wikipedia.org/wiki/%E5%B9%BF%E5%BA%A6%E4%BC%98%E5%85%88%E6%90%9C%E7%B4%A2)|邻接矩阵<br/>邻接链表|O(\|v\|<sup>2</sup>)<br/>O(\|v\|+\|E\|)|O(\|v\|<sup>2</sup>)<br/>O(\|v\|+\|E\|)
[DFS深度优先搜索](https://zh.wikipedia.org/wiki/%E6%B7%B1%E5%BA%A6%E4%BC%98%E5%85%88%E6%90%9C%E7%B4%A2)|邻接矩阵<br/>邻接链表|O(\|v\|<sup>2</sup>)<br/>O(\|v\|+\|E\|)|O(\|v\|<sup>2</sup>)<br/>O(\|v\|+\|E\|)

### 其他算法
算法 |思想| 应用
---|---|---
[分治法](https://zh.wikipedia.org/wiki/%E5%88%86%E6%B2%BB%E6%B3%95)|把一个复杂的问题分成两个或更多的相同或相似的子问题，直到最后子问题可以简单的直接求解，原问题的解即子问题的解的合并|[循环赛日程安排问题](https://github.com/huihut/interview/tree/master/Problems/RoundRobinProblem)、排序算法（快速排序、归并排序）
[动态规划](https://zh.wikipedia.org/wiki/%E5%8A%A8%E6%80%81%E8%A7%84%E5%88%92)|通过把原问题分解为相对简单的子问题的方式求解复杂问题的方法，适用于有重叠子问题和最优子结构性质的问题|[背包问题](https://github.com/huihut/interview/tree/master/Problems/KnapsackProblem)、斐波那契数列
[贪心法](https://zh.wikipedia.org/wiki/%E8%B4%AA%E5%BF%83%E6%B3%95)|一种在每一步选择中都采取在当前状态下最好或最优（即最有利）的选择，从而希望导致结果是最好或最优的算法|旅行推销员问题（最短路径问题）、最小生成树、哈夫曼编码

### Leetcode Problems
* [Github . haoel/leetcode](https://github.com/haoel/leetcode)
* [Github . pezy/LeetCode](https://github.com/pezy/LeetCode)

### 剑指 Offer
* [Github . zhedahht/CodingInterviewChinese2](https://github.com/zhedahht/CodingInterviewChinese2)
* [Github . gatieme/CodingInterviews](https://github.com/gatieme/CodingInterviews)

### Cracking the Coding Interview 程序员面试金典
* [Github . careercup/ctci](https://github.com/careercup/ctci)
* [牛客网 . 程序员面试金典](https://www.nowcoder.com/ta/cracking-the-coding-interview)

### 牛客网
* [牛客网 . 在线编程专题](https://www.nowcoder.com/activity/oj)

# 设计模式
* [C++设计模式](https://github.com/youngyangyang04/DesignPattern)
    * TODO
* C++单例模式 
* 用C++设计一个不能被继承的类
* 如何定义一个只能在堆上定义对象的类?栈上呢
    * 只能在堆上：析构函数设为私有，类对象就无法建立在栈上(编译器会监测析构函数可用性，但是new是可以的，但是有个缺陷，继承问题无法解决，好的解决方法是下面这种)

```
class  A {
protected :
    A(){}
    ~A(){}
public :
    static  A* create() {
        return new A();
    }
    void  destory() {
        delete   this ;
    }
};
```
    * 只能在栈上：构造函数和析构函数私有化就可以了

# 操作系统 

* linux的内存管理机制，内存寻址方式，什么叫虚拟内存，内存调页算法，任务调度算法 
    * 寻址方式：逻辑地址->线性地址->物理地址，内存管理单元 MMU 用来翻译虚拟内存地址。
    * 虚拟内存：解决多进程对内存操作的冲突问题
    * 分页：操作系统将整个物理内存以 4K
    为单位，划分为各个页。之后进行内存分配时，都以页为单位，减小虚拟内存页对应物理内存页的映射表，如 4G 内存只需要 8M
    的映射表，进程访问的物理地址还没有被分配，系统则会产生一个缺页中断，在中断处理时，系统切到内核态为进程虚拟地址分配物理地址，分页很好的保证了内存完整性
    （进程认为自己申请的都是连续的内存空间）和安全（页表权限）；并且有利于数据的共享，例如只加载一份共享库，用页表去指向正确加载的位置；并且虚拟内存得意支持SWAP。
* 锁：互斥锁，乐观锁，悲观锁 
    * 自旋锁与互斥锁：互斥锁用「线程切换」来应对，自旋锁则用「忙等待」来应对，所以如果锁内上下文很短，最好用自旋锁来防止线程上下文切换
    * 读写锁：读多写少的时候，优势比较大
    * 乐观锁与悲观锁：乐观锁先做操作，再看有没有冲突，若发现冲突，就放弃操作；适合加锁成本很高，冲突概率很低的场合。
    * 死锁必要条件及避免算法：互斥，占有等待，不可抢占，循环等待，避免的方式是破坏死锁生成的必要条件即可
* 常见的信号、系统如何将一个信号通知到进程
    * 进程即将从内核态返回用户态时，检查维护的未决信号的链表，系统其实就是把该信号加入到这个未决信号链表当中
    * 常见信号：SIGINT, SIGSEGV, SIGTERM, SIGCLD
* linux系统的各类同步机制、linux系统的各类异步机制
    * 同步机制：互斥锁，条件变量（生产者消费者问题），读写锁
    * 异步机制：信号
* mutex使用注意
    * 尽量使用不可重入的mutex：防止同一个线程内外层函数都能拿到锁修改
    * 每次调用Guard对象的时候考虑下调用栈上有没有调用过
    * 不用跨进程的mutex，进程通信使用socket
    * 加锁解锁保持在同一个线程
    * PTHREAD_MUTEX_ERROR_CHECK可以用来排查错误
    * 一般一个程序的线程只会等在condition variable或者epoll_wait上，如果等在了类似__lll_lock_wait，一般就发生了死锁
    * void print() const __attribute__ ((noinline))可以禁止编译器inline函数，从而看清调用栈：查看线程的bt，先gdb然后thread apply all bt
    * 了解一下什么是false sharing & cpu cache效应：
* 条件变量的使用：
    * 对wait端：
        * 和mutex一起使用
        * mutex已经上锁的情况下才能调用wait
        * 把判断的bool条件和wait放到while循环里面（必须是while循环，防止spurious wakeup）
    * 对single/broadcast端：
        * 不一定要在mutex上锁的情况下调用singal
        * signal之前要修改bool表达式，切要先加锁
        * signal一般标示资源可用，broadcast一般标示状态变化
    * 例子：
```
void enqueue(int x)
{
    MutexLockGuark(lock_);
    queue.push(x);
    cond.notify();
}
int dequeue()
{
    MutexLockGuark(lock_);
    while (queue.empty())
    {
        cond.wait();
    }
    assert (!queue.empty());
    return queue.pop();
}
```

# linux 服务器
* 五种I/O 模式:阻塞I/O,非阻塞 I/O,I/O 多路复用,信号驱动 I/O,异步 I/O
    * select 模型和 poll 模型，epoll模型
    * socket服务端的实现，select和epoll的区别(必问)
    * epoll哪些触发模式，有啥区别？
* 用户态和内核态的区别
    * 用户态到内核态条件：设备中断，异常（例如缺页），系统调用；
    * 主要区别在于特权级别的不同，能访问的内存空间及对象区别，处理器是否可以抢占等等。

* 进程之间的通信方式以及优缺点
    * 管道（PIPE）
        * 有名管道：一种半双工的通信方式，它允许无亲缘关系进程间的通信
            * 优点：可以实现任意关系的进程间的通信
            * 缺点：
                1. 长期存于系统中，使用不当容易出错
                2. 缓冲区有限
        * 无名管道：一种半双工的通信方式，只能在具有亲缘关系的进程间使用（父子进程）
            * 优点：简单方便
            * 缺点：
                1. 局限于单向通信 
                2. 只能创建在它的进程以及其有亲缘关系的进程之间
                3. 缓冲区有限
    * 信号量（Semaphore）：一个计数器，可以用来控制多个线程对共享资源的访问
        * 优点：可以同步进程
        * 缺点：信号量有限
    * 信号（Signal）：一种比较复杂的通信方式，用于通知接收进程某个事件已经发生
    * 消息队列（Message Queue）：是消息的链表，存放在内核中并由消息队列标识符标识
        * 优点：可以实现任意进程间的通信，并通过系统调用函数来实现消息发送和接收之间的同步，无需考虑同步问题，方便
        * 缺点：信息的复制需要额外消耗 CPU 的时间，不适宜于信息量大或操作频繁的场合
    * 共享内存（Shared Memory）：映射一段能被其他进程所访问的内存，这段共享内存由一个进程创建，但多个进程都可以访问
        * 优点：无须复制，快捷，信息量大
        * 缺点：
            1. 通信是通过将共享空间缓冲区直接附加到进程的虚拟地址空间中来实现的，因此进程间的读写操作的同步问题
            2. 利用内存缓冲区直接交换信息，内存的实体存在于计算机中，只能同一个计算机系统中的诸多进程共享，不方便网络通信
    * 套接字（Socket）：可用于不同计算机间的进程通信
        * 优点：
            1. 传输数据为字节级，传输数据可自定义，数据量小效率高
            2. 传输数据时间短，性能高
            3. 适合于客户端和服务器端之间信息实时交互
            4. 可以加密,数据安全性强
        * 缺点：需对传输的数据进行解析，转化成应用级的数据。

* 线程之间的通信方式
    * 锁机制：包括互斥锁/量（mutex）、读写锁（reader-writer lock）、自旋锁（spin lock）、条件变量（condition）
        * 互斥锁/量（mutex）：提供了以排他方式防止数据结构被并发修改的方法。
        * 读写锁（reader-writer lock）：允许多个线程同时读共享数据，而对写操作是互斥的。
        * 自旋锁（spin lock）与互斥锁类似，都是为了保护共享资源。互斥锁是当资源被占用，申请者进入睡眠状态；而自旋锁则循环检测保持者是否已经释放锁。
        * 条件变量（condition）：可以以原子的方式阻塞进程，直到某个特定条件为真为止。对条件的测试是在互斥锁的保护下进行的。条件变量始终与互斥锁一起使用。
    * 信号量机制(Semaphore)
        * 无名线程信号量
        * 命名线程信号量
    * 信号机制(Signal)：类似进程间的信号处理
    * 屏障（barrier）：屏障允许每个线程等待，直到所有的合作线程都达到某一点，然后从该点继续执行。

    线程间的通信目的主要是用于线程同步，所以线程没有像进程通信中的用于数据交换的通信机制  


* 进程之间私有和共享的资源

    * 私有：地址空间、堆、全局变量、栈、寄存器
    * 共享：代码段，公共数据，进程目录，进程 ID

* 线程之间私有和共享的资源

    * 私有：线程栈，寄存器，程序计数器
    * 共享：堆，地址空间，全局变量，静态变量


* 对比

对比维度 | 多进程 | 多线程 | 总结
---|---|---|---
数据共享、同步|数据共享复杂，需要用 IPC；数据是分开的，同步简单|因为共享进程数据，数据共享简单，但也是因为这个原因导致同步复杂|各有优势
内存、CPU|占用内存多，切换复杂，CPU 利用率低|占用内存少，切换简单，CPU 利用率高|线程占优
创建销毁、切换|创建销毁、切换复杂，速度慢|创建销毁、切换简单，速度很快|线程占优
编程、调试|编程简单，调试简单|编程复杂，调试复杂|进程占优
可靠性|进程间不会互相影响|一个线程挂掉将导致整个进程挂掉|进程占优
分布式|适应于多核、多机分布式；如果一台机器不够，扩展到多台机器比较简单|适应于多核分布式|进程占优

* 优劣

优劣|多进程|多线程
---|---|---
优点|编程、调试简单，可靠性较高|创建、销毁、切换速度快，内存、资源占用小
缺点|创建、销毁、切换速度慢，内存、资源占用大|编程、调试复杂，可靠性较差

* Linux 内核的同步方式

    * 原因：在现代操作系统里，同一时间可能有多个内核执行流在执行，因此内核其实像多进程多线程编程一样也需要一些同步机制来同步各执行单元对共享数据的访问。尤其是在多处理器系统上，更需要一些同步机制来同步不同处理器上的执行单元对共享的数据的访问。

* 同步方式
    * 原子操作
    * 信号量（semaphore）
    * 读写信号量（rw_semaphore）
    * 自旋锁（spinlock）
    * 大内核锁（BKL，Big Kernel Lock）
    * 读写锁（rwlock）
    * 大读者锁（brlock-Big Reader Lock）
    * 读-拷贝修改(RCU，Read-Copy Update)
    * 顺序锁（seqlock）

* 协程
    * 协程是一种轻量级的线程，本质上协程就是用户空间下的线程，如果把线程/进程当作虚拟“CPU”，协程即跑在这个“CPU”上的线程。
    * 特点：占用的资源更少，所有的切换和调度都发生在用户态，协程叫法来源于多个协程相互协作，是在用户态显示控制切换

#### 字节序
##### 概念

主机字节序又叫 CPU 字节序，其不是由操作系统决定的，而是由 CPU 指令集架构决定的。主机字节序分为两种：

* 大端字节序（Big Endian）：高序字节存储在低位地址，低序字节存储在高位地址，又叫网络字节序
* 小端字节序（Little Endian）：高序字节存储在高位地址，低序字节存储在低位地址

##### 存储方式

32 位整数 `0x12345678` 是从起始位置为 `0x00` 的地址开始存放，则：

内存地址 | 0x00 | 0x01 | 0x02 | 0x03
---|---|---|---|---
大端|12|34|56|78
小端|78|56|34|12

大端小端图片

![大端序](https://gitee.com/huihut/interview/raw/master/images/CPU-Big-Endian.svg.png)
![小端序](https://gitee.com/huihut/interview/raw/master/images/CPU-Little-Endian.svg.png)

##### 判断大端小端

判断大端小端

可以这样判断自己 CPU 字节序是大端还是小端：

```cpp
#include <iostream>
using namespace std;

int main()
{
	int i = 0x12345678;

	if (*((char*)&i) == 0x12)
		cout << "大端" << endl;
	else	
		cout << "小端" << endl;

	return 0;
}
```

### 页面置换算法

在地址映射过程中，若在页面中发现所要访问的页面不在内存中，则产生缺页中断。当发生缺页中断时，如果操作系统内存中没有空闲页面，则操作系统必须在内存选择一个页面将其移出内存，以便为即将调入的页面让出空间。而用来选择淘汰哪一页的规则叫做页面置换算法。

#### 分类

* 全局置换：在整个内存空间置换
* 局部置换：在本进程中进行置换

#### 算法

全局：
* 工作集算法
* 缺页率置换算法

局部：
* 最佳置换算法（OPT）
* 先进先出置换算法（FIFO）
* 最近最久未使用（LRU）算法
* 时钟（Clock）置换算法


# 计算机网络

* TCP和UDP区别

| | UDP | TCP |
|--|-|-|
| 是否连接| 无连接  | 面向连接  |
| 是否可靠|不可靠传输，不使用流量控制和拥塞控制  | 可靠传输，使用流量控制和拥塞控制 |
| 连接对象个数|支持一对一，一对多，多对一和多对多交互通信  | 只能是一对一通信 |
| 传输方式|面向报文	| 面向字节流 |
| 首部开销|首部开销小，仅8字节	| 首部最小20字节，最大60字节 |
| 适用场景|适用于实时应用（视频会议、直播等）	| 适用于要求可靠传输的应用，例如文件传输 |

* TCP和UDP三次握手和四次挥手状态及消息类型 
* time_wait，close_wait状态产生原因，keepalive
* 什么是滑动窗口，超时重传
* 列举你所知道的tcp选项：MSS, SO_RCVBUF
* connect会阻塞检测及防止，socket什么情况下可读？ 
* socket什么情况下可读？

| |可读| 可写 |
|--|-|-|
| 有数据可读| yes| |
| 是关闭连接的读一半|yes| |
| 是listen socket|yes| |
| 有用于写数据的空间|| yes|
| 是关闭连接的写一半|| yes|
| 有错误待处理|yes| yes|

* connect会阻塞，怎么解决?(必考必问) 
    * socket设置为非阻塞，用select或epoll检查描述符是否可写，
    * 使用signal定时器设置定时处理函数，超时会跳过阻塞的connect
* keepalive是什么？
    * 为了让在一次TCP连接中多次发送http请求，减少tcp连接建立次数，减少TIME_WAIT状态，提高性能和吞吐
* 正向代理和反向代理的区别
    * 正向代理代理的对象是客户端，反向代理代理的对象是服务端，典型的例子就是ssr以及nginx反向代理服务器，正向代理隐藏真实客户端，反向代理隐藏真实服务端
* 为什么服务器一般都会屏蔽SIGPIPE信号
    * TCP是全双工的信道, 可以看作两条单工信道, TCP连接两端的两个端点各负责一条。当对端调用close时, 虽然本意是关闭整个两条信道, 但本端只是收到FIN包。按照TCP协议的语义, 表示对端只是关闭了其所负责的那一条单工信道, 仍然可以继续接收数据. 也就是说, 因为TCP协议的限制, 一个端点无法获知对端的socket是调用了close还是shutdown。对一个已经收到FIN包的socket调用read方法, 如果接收缓冲已空, 则返回0, 这就是常说的表示连接关闭. 但第一次对其调用write方法时, 如果发送缓冲没问题, 会返回正确写入(发送)。但发送的报文会导致对端发送RST报文, 因为对端的socket已经调用了close, 完全关闭, 既不发送, 也不接收数据. 所以, 第二次调用write方法(假设在收到RST之后), 会生成SIGPIPE信号, 导致进程退出.
    


# 数据库 

* 谈谈数据库中索引的理解，索引和主键区别
    * 优点：
        * 可以大大加快数据的检索速度
        * 可以加速表和表之间的连接
        * 在使用分组和排序子句进行数据检索时，显著减少查询中分组和排序的时间。
    * 缺点：
        * 创建索引和维护索引要耗费时间，随着数据量的增加而增加
        * 索引需要占物理空间，除了数据表占数据空间之外，每一个索引还要占一定的物理空间
        主键与索引区别：
        * 主键属于索引的一种，索引一般分为，唯一索引，聚簇索引，主键索引 
* 关系型数据库和非关系数据库的特点 
    * TODO
* 数据库范式：
    * 列不可再分
    * 属性完全依赖于主键
    * 属性不依赖于其它非主属性，属性直接依赖于主键
* 数据库binlog日志：用于主从复制，实现主从同步
    * 主库打开binlog
    * 当有增删改操作时，必须记录到主库的binlog
    * 主库通过IO线程把binlog里的内容传给从库relay log
    * 从库sql线程负责读取relay log里的信息并应用到数据库
* B TREE 和B+TREE的区别 
    * B树：
        * 一种二叉搜索树。
        * 除根节点外的所有非叶节点至少含有（M/2（向上取整）-1）个关键字，每个节点最多有M-1个关键字，以升序排列。M阶B树的除根节点外的所有非叶节点的关键字取值区间为[M/2-1(向上取整),M-1]。
        * 每个节点最多有M-1个关键字
    * B+ 树：
        * 有n棵子树的非叶子结点中含有n个关键字（b树是n-1个），关键字不保存数据，只用来索引，所有数据保存在叶子节点（b树是每个关键字都保存数据）。
        * 所有的叶子结点中包含了全部关键字的信息，及指向含这些关键字记录的指针，且叶子结点本身依关键字的大小自小而大顺序链接（叶子节点组成一个链表）。
        * 所有的非叶子结点可以看成是索引部分，结点中仅含其子树中的最大（或最小）关键字。
        * 通常在b+树上有两个头指针，一个指向根结点，一个指向关键字最小的叶子结点。
        * 同一个数字会在不同节点中重复出现，根节点的最大元素就是b+树的最大元素。
    * 区别：
        * B树每个节点都存储数据，B+树只有叶子节点存储数据（B+树中有两个头指针：一个指向根节点，另一个指向关键字最小的叶节点），叶子节点包含了这棵树的所有数据，所有的叶子结点使用链表相连，便于区间查找和遍历，所有非叶节点起到索引作用。
        * B树中叶节点包含的关键字和其他节点包含的关键字是不重复的，B+树的索引项只包含对应子树的最大关键字和指向该子树的指针，不含有该关键字对应记录的存储地址。
        * B树中每个节点（非根节点）关键字个数的范围为[m/2(向上取整)-1,m-1](根节点为[1,m-1])，并且具有n个关键字的节点包含（n+1）棵子树。B+树中每个节点（非根节点）关键字个数的范围为[m/2(向上取整),m](根节点为[1,m])，具有n个关键字的节点包含（n）棵子树。
        * B+树中查找，无论查找是否成功，每次都是一条从根节点到叶节点的路径。
    * 优缺点：
        * B树的每一个节点都包含key和value，因此经常访问的元素可能离根节点更近，因此访问也更迅速。
        * 所有的叶子结点使用链表相连，便于区间查找和遍历。B树则需要进行每一层的递归遍历。相邻的元素可能在内存中不相邻，所以缓存命中性没有B+树好。
        * b+树的中间节点不保存数据，能容纳更多节点元素。
        * 数据库索引是存储在磁盘上的，考虑到磁盘IO，当数据量大时，不能把整个索引全部加载到内存，只能逐一加载每一个磁盘页（对应索引树的节点）。所以我们要减少IO次数，对于树来说，IO次数就是树的高度，而“矮胖”就是b树的特征之一，m的大小取决于磁盘页的大小。
* 存储引擎
    * InnoDB
        * InnoDB 是 MySQL 默认的事务型存储引擎，只要在需要它不支持的特性时，才考虑使用其他存储引擎。
        * InnoDB 采用 MVCC 来支持高并发，并且实现了四个标准隔离级别(未提交读、提交读、可重复读、可串行化)。其默认级别时可重复读（REPEATABLE READ），在可重复读级别下，通过 MVCC + Next-Key Locking 防止幻读。
        * 主索引时聚簇索引，在索引中保存了数据，从而避免直接读取磁盘，因此对主键查询有很高的性能。
        * InnoDB 内部做了很多优化，包括从磁盘读取数据时采用的可预测性读，能够自动在内存中创建 hash 索引以加速读操作的自适应哈希索引，以及能够加速插入操作的插入缓冲区等。
        * InnoDB 支持真正的在线热备份，MySQL 其他的存储引擎不支持在线热备份，要获取一致性视图需要停止对所有表的写入，而在读写混合的场景中，停止写入可能也意味着停止读取。
    * MyISAM
        * 设计简单，数据以紧密格式存储。对于只读数据，或者表比较小、可以容忍修复操作，则依然可以使用它。
        * 提供了大量的特性，包括压缩表、空间数据索引等。
        * 不支持事务。
        * 不支持行级锁，只能对整张表加锁，读取时会对需要读到的所有表加共享锁，写入时则对表加排它锁。但在表有读取操作的同时，也可以往表中插入新的记录，这被称为并发插入（CONCURRENT INSERT）。
        * 可以手工或者自动执行检查和修复操作，但是和事务恢复以及崩溃恢复不同，可能导致一些数据丢失，而且修复操作是非常慢的。
        * 如果指定了 DELAY_KEY_WRITE 选项，在每次修改执行完成时，不会立即将修改的索引数据写入磁盘，而是会写到内存中的键缓冲区，只有在清理键缓冲区或者关闭表的时候才会将对应的索引块写入磁盘。这种方式可以极大的提升写入性能，但是在数据库或者主机崩溃时会造成索引损坏，需要执行修复操作。
* InnoDB 和 MyISAM 的比较
        * 事务：InnoDB 是事务型的，可以使用 Commit 和 Rollback 语句。
        * 并发：MyISAM 只支持表级锁，而 InnoDB 还支持行级锁。
        * 外键：InnoDB 支持外键。
        * 备份：InnoDB 支持在线热备份。
        * 崩溃恢复：MyISAM 崩溃后发生损坏的概率比 InnoDB 高很多，而且恢复的速度也更慢。
        * 其它特性：MyISAM 支持压缩表和空间数据索引。
* 数据库相关：https://juejin.cn/post/6883270227078070286
            https://juejin.cn/post/6869532756498448392

* 秒杀系统
    * 秒杀系统的问题
        * 高并发
        * 超卖
        * 恶意请求：黄牛
        * 数据库
    * 解决
        * 资源静态化
        * 秒杀连接加密，随机md5,使url动态化，前端代码获取后端url校验
        * 库存预热，加入到redis中，事务相关的东西可以依赖redis的事务，LUA，乐观锁等等，这些都能防止超卖
        * 限流&降级&熔断&隔离：
        * 流量削峰：消息队列
        * 分布式事务：2PC，3PC，TCC（https://www.cnblogs.com/jajian/p/10014145.html）等

# 分布式
* zookeeper：ZooKeeper是一个分布式应用程序协调服务，它包含一个简单的原语集，分布式应用程序可以基于它实现同步服务，配置维护和命名服务等。
* 基础知识：https://www.cnblogs.com/luxiaoxun/p/4887452.html
* zk分布式锁：https://juejin.im/post/6844904117563817991
* 使用场景：
    * 服务注册与订阅（共用节点）
    * 分布式通知（监听znode）
    * 服务命名（znode特性）
    * 数据订阅、发布（watcher）
    * 分布式锁（临时节点）
* 节点类型：
    * 持久化节点（zk断开节点还在）
    * 持久化顺序编号目录节点
    * 临时目录节点（客户端断开后节点就删除了）
    * 临时目录编号目录节点

# Redis
* redis分布式锁：https://juejin.cn/post/6844904126288150542

# 海量数据处理 

* bitmap 
* Map-Reduce原理 
* BloomFilter原理 
* Trie树原理 
* LSM树原理 

## 📏 设计模式

> 各大设计模式例子参考：[CSDN专栏 . C++ 设计模式](https://blog.csdn.net/liang19890820/article/details/66974516) 系列博文

[设计模式工程目录](DesignPattern)

### 单例模式

[单例模式例子](DesignPattern/SingletonPattern)

### 抽象工厂模式

[抽象工厂模式例子](DesignPattern/AbstractFactoryPattern)

### 适配器模式

[适配器模式例子](DesignPattern/AdapterPattern)

### 桥接模式

[桥接模式例子](DesignPattern/BridgePattern)

### 观察者模式

[观察者模式例子](DesignPattern/ObserverPattern)

### 设计模式的六大原则

* 单一职责原则（SRP，Single Responsibility Principle）
* 里氏替换原则（LSP，Liskov Substitution Principle）
* 依赖倒置原则（DIP，Dependence Inversion Principle）
* 接口隔离原则（ISP，Interface Segregation Principle）
* 迪米特法则（LoD，Law of Demeter）
* 开放封闭原则（OCP，Open Close Principle）

# 程序员求职

* [简历模板](https://github.com/youngyangyang04/Markdown-Resume-Template)
* [程序员要如何写简历](https://mp.weixin.qq.com/s/PkBpde0PV65dJjj9zZJYtg)
* [适合新手练习的Github小项目（代码简单，功能实用）](https://mp.weixin.qq.com/s/Bc8Co6TiYxhbrzGLfSrISA)
* [一线互联网公司技术面试的流程以及注意事项](https://mp.weixin.qq.com/s/1VMvQ_6HbVpEn85CNilTiw)
* [如何使用markdown来制作一份自己的简历](https://mp.weixin.qq.com/s/ejvKML-NmEzok15GOzs62A)
* [深圳原来有这么多互联网公司，你都知道么？](https://mp.weixin.qq.com/s/Yzrkim-5bY0Df66Ao-hoqA)

# 程序员的工具

工欲善其事必先利其器

* [vim配置](https://github.com/youngyangyang04/PowerVim)
* [程序员为什么要使用Markdown](https://mp.weixin.qq.com/s/IYbHXABVsFETXW66DYd5nA)
* [程序员应该常逛哪些资讯类网站](https://mp.weixin.qq.com/s/ScMTuJ4WnlQTAbYk0_jeXA)


# 适合新手的开源项目

* [fileHttpServer(go语言实现)](https://github.com/youngyangyang04/fileHttpServer)
* [Sqlgen（shell脚本实现的批量操作mysql）](https://github.com/youngyangyang04/PowerSqlgen)
* [NosqlAttack （python实现）](https://github.com/youngyangyang04/NoSQLAttack)

## 📝 面试题目经验

* [牛客网 . 2020秋招面经大汇总！（岗位划分）](https://www.nowcoder.com/discuss/205497)
* [牛客网 . 【备战秋招】2020届秋招备战攻略](https://www.nowcoder.com/discuss/197116)
* [牛客网 . 2019校招面经大汇总！【每日更新中】](https://www.nowcoder.com/discuss/90907)
* [牛客网 . 2019校招技术类岗位面经汇总【技术类】](https://www.nowcoder.com/discuss/146655)
* [牛客网 . 2018校招笔试真题汇总](https://www.nowcoder.com/discuss/68802)
* [牛客网 . 2017秋季校园招聘笔经面经专题汇总](https://www.nowcoder.com/discuss/12805)
* [牛客网 . 史上最全2017春招面经大合集！！](https://www.nowcoder.com/discuss/25268)
* [牛客网 . 面试题干货在此](https://www.nowcoder.com/discuss/57978)
* [知乎 . 互联网求职路上，你见过哪些写得很好、很用心的面经？最好能分享自己的面经、心路历程。](https://www.zhihu.com/question/29693016)
* [知乎 . 互联网公司最常见的面试算法题有哪些？](https://www.zhihu.com/question/24964987)
* [CSDN . 全面整理的C++面试题](http://blog.csdn.net/ljzcome/article/details/574158)
* [CSDN . 百度研发类面试题（C++方向）](http://blog.csdn.net/Xiongchao99/article/details/74524807?locationNum=6&fps=1)
* [CSDN . c++常见面试题30道](http://blog.csdn.net/fakine/article/details/51321544)
* [CSDN . 腾讯2016实习生面试经验（已经拿到offer)](http://blog.csdn.net/onever_say_love/article/details/51223886)
* [cnblogs . C++面试集锦( 面试被问到的问题 )](https://www.cnblogs.com/Y1Focus/p/6707121.html)
* [cnblogs . C/C++ 笔试、面试题目大汇总](https://www.cnblogs.com/fangyukuan/archive/2010/09/18/1829871.html)
* [cnblogs . 常见C++面试题及基本知识点总结（一）](https://www.cnblogs.com/LUO77/p/5771237.html)
* [segmentfault . C++常见面试问题总结](https://segmentfault.com/a/1190000003745529)
