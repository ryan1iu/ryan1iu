---
title: Linux C 编程笔记
description: null
date: 2024-12-26T06:40:06.000Z
image: null
math: null
license: null
hidden: false
comments: true
draft: false
tags:
  - C
categories:
  - Notes
  - Programing Language
---
# C编程基础
## 基本注意事项

1. 头文件包含的重要性
2. 以函数为单位进行程序编写
3. 变量先定义再使用
4. return 0
5. 多用空格和空行
6. 大段函数注释可以通过#if 0  ---   #endif 进行注释
7. 没有单位的数值在计算机中是没有意义的

`算法`： 解决问题的方法。（流程图、NS图、有限状态机FSM）  
`程序`： 用某种语言实现算法   
`进程`： 防止写越界，防止内存泄露

## 数据类型
1. 所占字节数
2. float 类型
3. char型是否有符号
4. 不同类型的0值： 0，‘0’， “0”， ‘\0’
5. 数据类型与后序代码中所使用的输入输出要相匹配，防止自相矛盾

### 常量与变量
`常量`： 在程序执行过程中值不会发生变化的量  
分类： 整型常量、实型常量、字符常量、字符串常量、标识常量
- 整型常量： 1，123
- 实型常量： 3.14
- 字符常量： 由单引号引起来的单个的字符或转义字符
- 字符串常量： 由双引号引起来的一个或多个字符组成的序列
    - 特殊情况： 空串，“”，字符串末尾以'\0'结束，因此空串也会占据一个字节的存储空间
- 标示常量： #define 定义的常量，预处理阶段，占用编译时间，GNU C 对宏进行了拓展，Linux中使用的就是GNU C

`变量`：用来保存一些特定内容，并在执行过程中值随时会发生变化的量

```
[存储类型] 数据类型 标识符 = 值
```

`标识符`：由字母、数字、下划线组成且不能以数字开头的一个标识序列    
`存储类型`： auto static register extern(说明型)
- auto: 默认、自动分配空间、自动回收空间
- register： 寄存器类型， 建议型关键字，由gcc做最终决定
    - 只能用来定义局部变量，不能定义全局变量；
    - 大小有限制，数据的大小不能超出当前寄存器的长度
    - 寄存器没有地址，所以一个寄存器类型的变量无法打印地址查看或使用
- static： 静态型，自动初始化为零值或空值，并且其变量的值具有继承性
- extern: 说明型，意味着不能改变被说明的变量的值或类型

### 变量的生命周期与作用范围
变量的生命周期本质上取决于其存储在进程地址空间中的哪个区域当中。对于`auto`和`register`类型的变量，在运行过程中分别存储在栈和寄存器中，其生存周期为函数调用开始到结束阶段。 
对于`static`类型和全局变量，其在程序运行过程中存储在程序段中，因此其生命周期为程序整个运行期间。    

对于局部变量（auto\register\局部static）而言，其作用域为定义该变量的函数或复合语句内部  
对于定义在函数外部的变量（外部static\全局变量）而言，其中`外部static`变量的作用域仅限于当前文件，无法被其他文件访问。`全局变量`的作用域为整个程序，如何需要在当前文件中访问其他文件定义的全局变量，需要使用`extern`关键字。

## 运算符
![img](https://imagebed-1300955178.cos.ap-beijing.myqcloud.com/20241230211112.png?imageSlim)

## 标准I/O
区分标准I/O与系统调用I/O    
- 格式化输入输出scanf printf
- 字符输入输出 getchat putchar
- 字符串输入输出 gets puts

### 格式化输入输出
```c
int printf(const char *restrict format, ...);
int scanf(const char *restrict format, ...);
```
format格式：
```
% [修饰符] 格式字符
```
修饰符：    
![modify](https://imagebed-1300955178.cos.ap-beijing.myqcloud.com/20250101105411.png?imageSlim)

格式字符：  
![format](https://imagebed-1300955178.cos.ap-beijing.myqcloud.com/20250101084954.png?imageSlim)

scanf使用注意事项： 
1. 在scanf中使用%s接受字符串是一个非常危险的操作，可能导致内存越界，最好使用专门的字符串接受函数
2. 如果在循环中使用scanf函数，一定要对其返回值进行校验。例如
```c
void func(void) {
    int i;
    while(1) {
        scanf("%d", &i);
        printf("%d\n", i);
    }
}
```
如果输出的内容与%d不匹配，那么程序就会进入死循环
3. 抑制符的使用
```c
/*
* 如果连续调用两次scanf函数从终端读取内容，那么后一个scanf函数不能接收到正确的结果
*/
```

### 缓冲区刷新机制
C语言中，当使用标准I/O时，会有一个缓冲区来暂时存储输入输出的内容来提高效率。
行缓冲：在遇到\n时或者缓冲区已满时进行输出，终端使用行缓冲模式
全缓冲：缓冲区满时进行输出，文件使用全缓冲模式
标准错误流：标准错误流不缓冲


## 流程控制

## 数组

## 指针

## 构造类型
### struct内存对齐
1. 成员的起始地址必须是对齐值的整数倍，对齐值通常是数据类型的大小
2. 结构体的大小必须是最大对齐值的整数倍，否则需要在结构体末尾进行填充
3. 成员的排列顺序影响结构体的大小

### union的应用场景
1. 节省内存，当多个数据类型`不会`同时使用时，可以使用Union节省内存
2. 类型转换，通过Union可以实现相同位模式下的不同数据类型的解释
```c
union convert {
	int i;
	unsigned int j;
};

int main()
{
	union convert a;
	a.i = 0xffffffff;
	/**
     * a.i = -1
     * a.j = 4294967295
     */
	printf("a.i = %d\na.j = %u\n", a.i, a.j);

	return 0;
}
```
3. 硬件编程，在嵌入式开发中Union可以用来访问寄存器的不同部分
```c
union Register {
	unsigned int i;
	struct {
		unsigned char byte1;
		unsigned char byte2;
		unsigned char byte3;
		unsigned char byte4;
	} bytes;
};

int main()
{
	union Register r;
	r.i = 0x12345678;
	/*
     * byte1 = 78
     * byte2 = 56
     * byte3 = 34
     * byte4 = 12
     * 说明当前机器采用小端模式
     * */
	printf("byte1 = %x\nbyte2 = %x\nbyte3 = %x\nbyte4 = %x\n",
	       r.bytes.byte1, r.bytes.byte2, r.bytes.byte3, r.bytes.byte4);

	return 0;
}
```
## 函数










































