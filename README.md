# PL/0 Compiler without error diagnosis processing
[视频解释 detailed explanation video](https://space.bilibili.com/595418829/channel/series)
### Lexer：Lexical analysis is realized in two ways
1. [a normal way](https://juejin.cn/post/6929689435437006862)
2. [use DFA](https://juejin.cn/post/6935074305780744223)
### version1：Relatively complete code just without error diagnosis processing.
没有错误诊断处理，遇到错误只是直接报错

## 常见问题
_存储组织上无法回避的问题：_
### 1. 如何实现嵌套定义
用栈。每一次过程调用都会生成一个栈帧，在栈上开辟空间时，必须先开辟3个空间保存“静态链、动态链、返回地址”。
- 静态链：定义关系中上一层的过程在栈中基地址
- 静态链：执行关系中上一个栈帧的基地址
- 返回地址：结束本次调用，返回到上层过程后下一条需要指令的地址
### 2. 如何分配不同作用域、生命周期的数据（PL/0允许嵌套定义，在一个过程中再定义其他过程，C语言是不允许的，所以PL/0涉及非局部量的访问的问题，而对于C语言非局部量就是全局量）

内存中设计不同的存储区，比如heap,stack,static data,program...
- stack：存储调用结束即可被销毁的局部量
- static data：存储全局变量/常量等生命周期长的 <br/>

使用静态链，对应base()，沿着静态链寻找非局部量的存储位置。
