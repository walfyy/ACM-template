# C++

**拷贝构造函数**

1.浅拷贝：默认拷贝构造函数是浅拷贝（不能用于动态分配）
~~~
class A{
private:
    int *p;
public:
    A(){p=new int();}
    ~A(){delete p;}
};
int main()
{
    A a;
    A b(a);
    return 0;
}
~~~
A a执行时p指向堆，执行浅拷贝时，只对成员赋值b.p=a.p,执行析构函数时会删除同一个堆空间，就会出错（即b.p指针悬挂）

2.深拷贝
~~~
class A{
private:
    int *p;
public:
    A(){p=new int();}
    A(const A&x){p=new int();*p=*x.p;}
    ~A(){delete p;}
};
int main()
{
    A a;
    A b(a);
    return 0;
}
~~~
完成拷贝后，a.p和b.p各指向一段内存空间，但是指向的空间内容相同！

对于一个类X, 如果一个构造函数的第一个参数是下列之一:

   a) X&
   
   b) const X&
   
   c) volatile X&
   
   d) const volatile X&
   
且没有其他参数或其他参数都有默认值,那么这个函数是拷贝构造函数.

## 右值引用，移动语义，完美转发，通用引用

**移动构造函数（移动语义的关键）**

区分左值和右值：能不能取地址！

定义移动构造函数后编译器会默认将右值进行移动构造，可以提升效率（函数传参为类对象，函数返回值为类对象）
~~~
class A{
private:
    int *p;
public:
    A(){p=new int();}
    A(A &&rhs)
    {
        p=rhs.p;
        rhs.p=NULL;
        puts("!!!");
    }
    ~A()
    {
        if(p!=NULL)delete p;
        else puts("&&&");
    }
};
int main()
{
    A a;
    A b=move(a);
    return 0;
}
~~~
右值引用，避免a的拷贝，直接将a给b，同时删除a

**完美转发**

forward保留参数的左右值类型
~~~
void f1(int &a)
{
    puts("&");
}
void f1(int &&a)
{
    puts("&&");
}
int main()
{
    int &&a=1;
    f1(forward<int>(a));
    return 0;
}
~~~

**通用引用**

1.必须满足T&&这种形式

2.类型T必须是通过推断得到的（函数模板，auto）

即可是左值引用，也可是右值引用

**常量函数**

注意非成员函数不能有CV[const, volatile]限定，常量成员函数不能调用

值传递，引用传递，指针传递

值传递：实参的拷贝，不改变实参的值

引用传递：实参的地址，改变实参的值

指针传递：指针指向实参的地址，改变实参的值

**引用和指针传递的区别：**

1.引用不能改变引用地址，只是别名不是实体，而指针是实体

2.sizeof计算变量大小时，得到引用变量大小，指针大小4

3.可以const指针，没有const引用

4.引用传递不能为空。

5.可以多级指针

**sizeof**

sizeof指针：指针大小

sizeof数组：数组内存大小* 数据bit大小

当数组名作为实参传入函数中，形参是指针所以不能用sizeof计算

## 虚函数

子类函数覆盖基类函数（即多态）

当基类指针指向基类对象时，调用的是基类的函数。

当基类指针指向子类对象时，调用的是子类的函数。

纯虚函数virtual void f()=0

基类中没有定义，但是任何派生类中都要实现该方法

目的：使派生类只是继承函数的接口

带有纯虚函数的类称为抽象类

**虚函数表（存放在全局区，每个类共用一个）建立在编译阶段，而虚函数表指针是在构造函数调用时被初始化的！**

**虚函数可以inline吗**

inline是在编译器将函数内容替换到函数调用处，是静态编译的。而虚函数是动态调用的，在编译器并不知道需要调用的是父类还是子类的虚函数，所以不能够inline声明展开，所以编译器会忽略

**虚继承**

虚继承是用来解决菱形继承的问题，对于D中和B，C相同的变量，还是要通过域限定来确定。
~~~
class A{
public:
    int a;
    void f(){puts("!!");}
};
class B:virtual public A{
public:
    int a;
};
class C:virtual public A{
public:
    int a;
};
class D:public B,public C{

};
int main()
{
    D d;
    d.f();
    d.A::f();
    d.C::f();
    return 0;
}
~~~


**程序编译的四个阶段：**

1.预处理阶段（.c—.i）：编译器将C程序的头文件编译进来，还有宏的替换。

2.编译阶段（.i—.s）：分析检查无误后，将代码翻译成汇编

3.汇编阶段（.s—.o）：汇编器将汇编程序翻译成机器语言

4.链接阶段：链接器负责处理合并目标代码，生成一个可执行目标文件，可以被加载到内存中，由系统执行

**GCC常用编译选项**

-c：只激活预处理,编译,和汇编,也就生成obj文件

-S：只激活预处理和编译，就是指把文档编译成为汇编代码。

-E：只激活预处理，不生成文档，需要把他重定向到一个输出文档里。

-o：定制目标名称，缺省的时候gcc 编译出来的文档是a.out

-ansi：关闭gnu c中和ansi c不兼容的特性，激活ansi c的专有特性。

-Dmacro：相当于C语言中的#define macro

-Dmacro=defn：相当于C语言中的#define macro=defn

-Umacro ：相当于C语言中的#undef macro

-Idir：指定头文件路径。

-llibrary：指定库

-Ldir：定制编译的时候，搜索库的路径。

-g：指示编译器，在编译的时候，产生调试信息。

-static：此选项将禁止使用动态库，所以，编译出来的东西，一般都很大。

-share：此选项将尽量使用动态库，所以生成文档比较小，但是需要系统由动态库。

-O0 -O1 -O2 -O3：编译器的优化选项的4个级别，-O0表示没有优化,-O1为缺省值，-O3优化级别最高

-Wall：会打开一些很有用的警告选项，建议编译时加此选项。

-std：指定C标准，如-std=c99使用c99标准，-std=gnu99，使用C99 再加上 GNU 的一些扩展。

**访问权限**

public: 能被类成员函数、子类函数、友元访问，也能被类的对象访问。 

private: 只能被类成员函数及友元访问，不能被其他任何访问，本身的类对象也不行。 

protected: 只能被类成员函数、子类函数及友元访问，不能被其他任何访问，本身的类对象也不行

使用private继承，父类的protected和public属性在子类中变为private 

使用protected继承，父类的protected和public属性在子类中变为protected 

使用public继承，父类中的protected和public属性不发生改变

**面向对象和面向过程的区别**

面向过程就是分析出解决问题所需要的步骤，然后用函数把这些步骤一步一步实现，使用的时候一个一个依次调用就可以了
面向对象是把构成问题事务分解成各个对象，建立对象的目的不是为了完成一个步骤，而是为了描叙某个事物在整个解决问题的步骤中的行为

## STL

vector扩容：如果当前位置内存已满，新开更大的内存，**把内容复制过来**，释放之前的内存，然后加入新元素（两倍或者1.5倍）

**为什么要成倍扩容，而不是增长固定容量**

二倍扩容时：当插入n个数时，大约需要log2(n)次扩容，每次需要复制2^i个元素，一共需要复制2^1+...+2^{log2(n)}=2(n-1),平均每次pushback需要2(n-1)/n,约O(1)复杂度

固定容量为k时：当插入n个数，需n/k次扩容，每次ki个元素，一共复制k+2k+...+n/k * k=(1+n/k)* n,当k为2时，复杂度O(n^2),均摊为O(n)

**为什么是两倍或者1.5倍，不是3，4倍**

如果以大于2倍的方式扩容，下一次申请的内存会大于之前分配内存的总和，导致之前分配的内存不能再被使用。

**vector vs list效率**

插入时，vector比list快，遍历时差不多

**memcpy源码**

memcpy内未解决内存覆盖问题，下面是优化过的

~~~
void *memcpy(void *dst,const void *src,size_t len)
{
    if(dst==NULL||src==NULL)return NULL;
    void *ret=dst;
    if(dst<=src||(char*)dst>=(char*)src+len)
    {
        while(len--)
        {
            *(char*)dst=*(char*)src;
            dst=(char*)dst+1;
            src=(char*)src+1;
        }
    }
    else
    {
        src=(char*)src+len-1;
        dst=(char*)dst+len-1;
        while(len--)
        {
            *(char*)dst=*(char*)src;
            dst=(char*)dst-1;
            src=(char*)src-1;
        }
    }
}
~~~

**share_ptr简单实现**
~~~
template<typename T>
class Shared_ptr{
public:
    Shared_ptr():count(new int(1)),val(new T()){}
    Shared_ptr(T *p):count(new int(1)),val(p){}
    Shared_ptr(const Shared_ptr&p):count(p.count),val(p.val){
        ++*count;
    }
    ~Shared_ptr(){
        del();
    }
    T& operator *(){
        return *val;
    }
    Shared_ptr& operator =(const Shared_ptr&p){
        ++*p.count;
        del();
        count=p.count;
        val=p.val;
        return *this;
    }
    void del(){
        if(--*count==0)
        {
            delete count;
            delete val;
        }
    }
    int use_count(){return *count;}
private:
    int *count;
    T *val;
};
int main()
{
    Shared_ptr<int>a;
    printf("%d\n",a.use_count());
    Shared_ptr<int>b=a;
    printf("%d %d\n",a.use_count(),b.use_count());
    return 0;
}
~~~


# 计网

rtt：报文往返时间

**tcp保证可靠性**

1.序列号，确认应答，超时重传

2.窗口控制与高速重发控制、快速重传

（窗口大小：无需确认即可继续发送数据的最大值）

快速重传：发送端收到三次相同确认应答，会立即重发

3.拥塞控制

慢启动：开始窗口为1，每次收到确认应答（一个rtt），窗口×2

拥塞避免：设置慢启动阀值（65536）窗口达到阀值，不再指数增加，而是每次+1

ACK：确认比特，ACK=1，即确认号字段有效

SYN：同步比特，SYN=1，即连接请求

FIN：终止比特，FIN=1，即数据发送完毕请求释放连接

**序列号seq**
**三次握手：**

1.客户端：发送SYN连接报文,seq=x

2.服务器：发送SYN确认连接报文,seq=y，ACK=1

3.客户端：发送ACK确认报文,ACK=y+1

**四次挥手：**

客户端：发送FIN终止请求，seq=x+2，ACK=y+1

服务器：发送ACK=X+3确认

客户端收到ACK后会关闭从客户端到服务器的连接，服务器会继续发送没发完的数据

服务器：发送FIN，seq=y+1，表示没有数据了

客户端：发送ACK=y+2，关闭从服务器到客户端的连接

**为什么TCP建立连接需要三次，而释放连接需要四次？**

TCP释放连接时服务器的ACK和FIN是分开发送的（中间隔着数据传输），

而TCP建立连接时服务器的ACK和SYN是一起发送的（第二次握手），

所以 TCP 建立连接需要三次，而释放连接则需要四次。

**TCP为什么要四次挥手？**
因为TCP是全双工模式

**为什么TCP释放连接后要等待2MSL**

MSL：报文最大生存时间

为了确认客户端发送的最后一个ACK报文，能到达服务器，如果服务器未收到，则会超时重传FIN+ACK报文，客户端即可在2MSL时间内收到，同时保证旧的数据过期

**TCP和UPD区别：**

1.TCP面向连接，UDP无连接

2.TCP可靠，UDP尽力交付，不可靠

3.TCP点到点，UDP可多对多

4.UDP没有拥塞控制

5.TCP面向字节流，UDP面向报文

6.TCP首部开销20字节，UDP8字节

**OSI七层模型**
物理层，数据链路层，网络层，传输层，会话层，表示层，应用层

**TCP/IP 四层模型**
网络接口层，网络层，传输层，应用层

**区别TCP/UDP报文**
看ip头中的协议标识字段，17是UDP，6是TCP

**http/https区别**

1.https明文，https有TLS加密

2.http协议端口80，https443

3.https协议需要服务端申请证书，浏览器安装对应的根证书

**http返回码：**

1xx:指示信息，表示请求已接受，继续处理

2xx：成功，请求已成功接受，理解，接受

3xx：重定向，请求必须进一步操作

4xx：客户端错误，请求有语法错误或请求无法实现

5xx：服务端错误，服务器不能实现合法请求

**在浏览器输入url，按回车**

1.浏览器在DNS服务器请求解析URL中域名对应IP地址

2.解析出IP后，根据IP地址和默认端口80，和服务器建立TCP链接

3.浏览器发出读取文件的http请求，该请求报文作为TCP三次握手的的第三个数据发送给服务器

4.服务器对浏览器请求做出相应，并把html文本发送给浏览器

5.释放tcp链接

6.浏览器将该html显示内容
