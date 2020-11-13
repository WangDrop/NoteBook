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

# 数据结构与算法

* [数据结构与算法学习攻略](https://github.com/youngyangyang04/leetcode-master)
    * TODO

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

# linux 服务器
* 五种I/O 模式:阻塞I/O,非阻塞 I/O,I/O 多路复用,信号驱动 I/O,异步 I/O
    * select 模型和 poll 模型，epoll模型
    * socket服务端的实现，select和epoll的区别(必问)
    * epoll哪些触发模式，有啥区别？
* 用户态和内核态的区别
    * 用户态到内核态条件：设备中断，异常（例如缺页），系统调用；
    * 主要区别在于特权级别的不同，能访问的内存空间及对象区别，处理器是否可以抢占等等。

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

# 海量数据处理 

* bitmap 
* Map-Reduce原理 
* BloomFilter原理 
* Trie树原理 
* LSM树原理 

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
