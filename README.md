# PL/0 Compiler without error diagnosis processing
[视频解释 detailed explanation video](https://space.bilibili.com/595418829/channel/series)
## Lexer：Lexical analysis is realized in two ways
### 1. A normal way using only if-else
- 遍历字符流，不需回溯
- 没用DFA，全程if-else，思路很好理解
- 识别单词的时候分为：1-标识符关键字,2-整数,3-符合运算符,4-单独字符
- 类别码是1,2,3...，用enum在头文件里定义了。
- 单独字符的类型码统一定义在ssym数组里了，关键字的类型码统一定义在wsym数组里了。因为他们是一一对应的，用数组比较简洁。
- 识别到token就直接输出了，万事从简（懒）...理解思想和方法就可再改进😃
- C 库函数 `int isalpha(int c)`：判断字符是否是字母，`int isdigit (int c)`：判断字符是否是数字。当然也直接用ASCII码。
- C 库函数 `int strcmp(const char *str1, const char *str2)` 把 str1 所指向的字符串和 str2 所指向的字符串进行比较。

### 2. Using DFA
- 遍历字符流，遇到不能识别的字符而结束本次识别时，回退一个字符，让它能被继续识别
- 重点：有限状态机三段式：（1）定义状态（2）根据现态和当前字符计算次态（3）更新现态   （和数字逻辑里的时序电路FSM设计差不多）
- 对于细节要耐心，注释和空格回车等字符的处理，获取下一个字符的时机🥰

## version1：Relatively complete code just without error diagnosis processing.
没有错误诊断处理，遇到错误只是直接报错
<hr/>

## 常见问题
_存储组织上无法回避的问题：_
### 1. 如何实现嵌套定义（PL/0允许嵌套定义，在一个过程中再定义其他过程，C语言是不允许的）
用栈。每一次过程调用都会生成一个栈帧，在栈上开辟空间时，必须先开辟3个空间保存“静态链、动态链、返回地址”。
- 静态链：定义关系中上一层的过程在栈中基地址
- 静态链：执行关系中上一个栈帧的基地址
- 返回地址：结束本次调用，返回到上层过程后下一条需要指令的地址
### 2. 如何分配不同作用域、生命周期的数据（PL/0涉及非局部量的访问的问题，而对于C语言来说，非局部量就是全局量）

内存中设计不同的存储区，比如heap,stack,static data,program...，具体划分与CPU的体系结构和OS有关
- stack：存储调用结束即可被销毁的局部量
- static data：存储全局变量、Pl/0的常量中的等生命周期长的 <br/>
- heap：不符合上述两种，可以手动申请和释放的，比如C语言中的malloc

内存分配策略：
- 静态分配（编译时就分配好）：比如分配大小固定、可全程访问的“全局变量、静态变量、常量、类的虚函数表”
- 动态分配（运行时才分配）：包括栈式分配/堆式分配
使用静态链，对应base()，沿着静态链寻找非局部量的存储位置。
