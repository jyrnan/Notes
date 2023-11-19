# 指针的类型信息保存在哪里

# [问题描述]

int* pnVar;和char* pcVar;这两个指针在做加法的时候，计算机怎么知道是加1个还是4个byte

对应的汇编代码：

```c
pnVar++;

00413031  mov         eax,dword ptr [pnVar]

00413034  add         eax,4

00413037  mov         dword ptr [pnVar],eax

pcVar++;

0041303A  mov         eax,dword ptr [pcVar]

0041303D  add         eax,1

00413040  mov         dword ptr [pcVar],eax
```

# [理解]

“类型” 的概念只存在于编译前，编译器会识别出各种数据类型，然后生成目标文件，obj里面就包含了每条语句的汇编实现，所以pnVar++或者pcVar++，在生成的PE文件.text节里面早已写死。在运行期不需要关心所谓的“类型”。

# [反思]

之前想不通此问题的原因：

认为汇编语言和C语言一样，变量在操作前需要知道数据类型。而没有切换到汇编的思维，在汇编眼里，或者说CPU眼里，取指(opcode)执行时，只认识bit流，opcode告诉bus一次取多少位数据，cpu就取多少位，压根不用知道所谓的类型

# [结论]

从汇编 reversing 到高级语言，需要自如的切换思维