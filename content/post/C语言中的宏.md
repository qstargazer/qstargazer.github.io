---
title: C语言中的宏
mathjax: true
date: 2018-07-21 20:33:48
tags:
  - 计算机
categories:
  - 技艺
---

C/C++中有些特殊的宏定义，面试时候被问到，写个短文总结下。
<!--more-->

## 宏定义中的\#和\#\#连字符

这两个字符在宏定义中代表连接和替换，

- `#`紧跟字母表示对应字符的字符串化，将对应的字符转换成对应的字符串，比如`#hello`就是`"hello"`
- `##`表示将宏定义中的两个标识符连接在一起，组成一个新的标识符，类似胶水。它首先查看`##`两边的字符，是否有宏定义可以替换的字符串，替换之，之后将两个连接在一起。

下面举例说明。

```c
#include <stdio.h>
#define trace(x, format) printf(#x " = %" #format "\n", x)
#define trace2(i) trace(x##i, d)

int main(int argc, _TCHAR* argv[])
{
    int i = 1;
    char *s = "three";
    float x = 2.0;

    trace(i, d);                // 相当于 printf("x = %d\n", x)
    trace(x, f);                // 相当于 printf("x = %f\n", x)
    trace(s, s);                // 相当于 printf("x = %s\n", x)

    int x1 = 1, x2 = 2, x3 = 3;
    trace2(1);                  // 相当于 trace(x1, d)
    trace2(2);                  // 相当于 trace(x2, d)
    trace2(3);                  // 相当于 trace(x3, d)

    return 0;
}
```

又比如

```c
#define STACK_ADD_TASK_IRAM_POOL1(TASK_NAME, task_name, task_type){\
task_info_g[INDEX_##TASK_NAME].task_entry_func = \
    (osa_task_func_ptr) stack_##task_type##_task;\
}
```

上面的语句中，我们会将其中的`TASK_NAME`和`task_type`替换掉，并与前后的标识符相连接生成新的标识符。

## 宏定义中的 do{} while(0)

经常在宏定义中会看到如下的语句

```c
#define STACK_ADD_MULTI_TASK(multi_sys_max, TASK_NAME, task_name, task_type)  \
    do{                                                                       \
       if(multi_sys_max >= 1)                                                 \
       {                                                                      \
           STACK_ADD_TASK_1_CARD(TASK_NAME, task_name, task_type);            \
       }                                                                      \
    }while(0)
```

这个语句的特点是宏定义之后紧跟一个`do{...}while(0)`的结构，看起来颇为繁琐，**那这样的定义有什么好处呢？**

首先，C 中的宏定义，在预编译阶段就会将宏定义的结构替换掉，使用宏定义定义函数。在代码替换中肯定希望像使用定义的函数使用宏定义，比如上面的语句在代码中肯定是下面这样的

```c
STACK_ADD_MULTI_TASK(multi_sys_max, TASK_NAME, task_name, task_type);
```

所以注意到**宏定义的while(0)后面没有分号**。

然后，这个结构和`if..else...`的控制结构可以完美结合，对照上面的宏定义，一般能想到的宏定义的结果修改如下

```c
#define STACK_ADD_MULTI_TASK(multi_sys_max, TASK_NAME, task_name, task_type)  \
    {                                                                         \
       if(multi_sys_max >= 1)                                                 \
       {                                                                      \
           STACK_ADD_TASK_1_CARD(TASK_NAME, task_name, task_type);            \
       }                                                                      \
    }
```

如果正常替换之前的语句，替换之后的结果就是

```c
{                                                                         \
   if(multi_sys_max >= 1)                                                 \
   {                                                                      \
       STACK_ADD_TASK_1_CARD(TASK_NAME, task_name, task_type);            \
   }                                                                      \
};
```

注意最后的分号，**这一行编译不通过**，但是很明显如果换成`do{}while(0)`结构就不存在这个问题。

其次，如果使用宏定义多行语句，那么使用大括号的宏定义嵌套在`if...else...`的结构中会遇到问题，比如定义

```c
#define BAR(X) f(x); g(x)
```

如果用在下面的程序中就会出现语法错误

```c
if(true)
    BAR(x)
else
   {
       //do nothing
   }
```

但是使用`do{...}while(0)`结构就不会有这样的问题。

## `__DATE__,__TIME__,__FILE__,__LINE__`特殊宏定义
-   \__LINE__：在源代码中插入当前源代码行号；
-   \__FILE__：在源文件中插入当前源文件名；
-   \__DATE__：在源文件中插入当前的编译日期
-   \__TIME__：在源文件中插入当前编译时间；
-   \__STDC__：当要求程序严格遵循ANSI C标准时该标识被赋值为1；
-   \__cplusplus：当编写C++程序时该标识符被定义

具体的使用方式参考如下的代码
```cpp
#include <stdio.h>

int main(void) {
    int answer;
    short x = 1;
    long y = 2;
    float u = 3.0;
    double v = 4.4;
    long double w = 5.54;
    char c = 'p';;

    // __DATE__, __TIME__, __FILE__, __LINE__ 为预定义宏
    printf("Date : %s\n", __DATE__);
    printf("Time : %s\n", __TIME__);
    printf("File : %s\n", __FILE__);
    printf("Line : %d\n", __LINE__);
    printf("Enter 1 or 0 : ");
    scanf("%d", &answer);

    // 这是一个条件表达式
    printf("%s\n", answer?"You sayd YES":"You said NO");

    // 各种数据类型的长度
    printf("The size of int %d\n", sizeof(answer));
    printf("The size of short %d\n", sizeof(x));
    printf("The size of long %d\n", sizeof(y));
    printf("The size of float %d\n", sizeof(u));
    printf("The size of double %d\n", sizeof(v));
    printf("The size of long double %d\n", sizeof(w));
    printf("The size of char %d\n", sizeof(c));
}
```

## `__VA_ARGS__`宏定义
当前主干代码的一个典型的用法如下
```cpp
#define DEFINE_KERNEL(gobalFunc, ...) \
 extern "C" __global__ __aicore__ void gobalFunc(__gm__ half* tableAddr, TABLE_SIZE_INT64 tableSize) \
 { \
 __VA_ARGS__(tableAddr, tableSize); \
 } \
\
 DLL_PUBLIC void run_##gobalFunc(uint32_t coreDim, void* l2ctrl, void* stream, \
 void* tableAddr, uint32_t tableSize) \
 { \
 gobalFunc<<<coreDim, (rtL2Ctrl_t*)l2ctrl, stream>>>( \
 (half*)tableAddr, static_cast<TABLE_SIZE_INT64>(tableSize)); \
 }
#endif
```

上面代码中`...`对应`__VA_ARGS__`，表示在宏展开中将`__VA_ARGS__`替换成`...`的内容，此处的`...`可以表示表示一个或多个参数，但是常见的编译器也允许传递0个参数。再比如如下的代码
```cpp
# define MYLOG(FormatLiteral, ...)  fprintf (stderr, "%s(%d): " FormatLiteral "\n", __FILE__, __LINE__, __VA_ARGS__)
```
对于下面的代码
```cpp
MYLOG("Too many balloons %u", 42);
```
可以扩展为
```cpp
fprintf (stderr, "%s(%d): Too many balloons %u\n", __FILE__, __LINE__, 42);
```

## 参考资料

- [C/C++ 宏定义中 \#、##、\#@的区别 - hellokandy 的博客 - CSDN 博客](http://blog.csdn.net/hellokandy/article/details/50592971)
- [c - Why use do {} while (0) in macro definition? - Stack Overflow](https://stackoverflow.com/questions/9495962/why-use-do-while-0-in-macro-definition)
- [c++ - Do you consider this technique “BAD”? - Stack Overflow](https://stackoverflow.com/questions/243967/do-you-consider-this-technique-bad)
- [c++ - Why use apparently meaningless do-while and if-else statements in macros? - Stack Overflow](https://stackoverflow.com/questions/154136/why-use-apparently-meaningless-do-while-and-if-else-statements-in-macros)
- -[C/C++中几个预定义的宏：__DATE__,__TIME__,__FILE__,__LINE___C语言中文网](http://c.biancheng.net/cpp/html/2552.html)
- [可变参数宏 - Wikiwand](https://www.wikiwand.com/zh-cn/%E5%8F%AF%E5%8F%98%E5%8F%82%E6%95%B0%E5%AE%8F)