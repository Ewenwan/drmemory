# Dr. Memory: the memory debugger

     Dr. Memory是一个内存调试工具，它是一个开源免费的内存检测工具，它能够及时发现内存相关的编程错误，

     比如未初始化访问、内存非法访问、数组越界读/写、以及内存泄露等。

 
## About Dr. Memory

Dr. Memory is a memory monitoring tool capable of identifying
memory-related programming errors such as accesses of uninitialized memory,
accesses to unaddressable memory (including outside of allocated heap units
and heap underflow and overflow), accesses to freed memory, double frees,
memory leaks, and (on Windows) handle leaks, GDI API usage errors, and
accesses to un-reserved thread local storage slots.

Dr. Memory operates on unmodified application binaries running on Windows,
Linux, Mac, or Android on commodity IA-32, AMD64, and ARM hardware.

Dr. Memory is released under an LGPL license and binary packages are
[available for
download](https://github.com/DynamoRIO/drmemory/wiki/Downloads).

Dr. Memory is built on the [DynamoRIO dynamic instrumentation tool
plaform](http://dynamorio.org).

![DynamoRIO logo](http://www.burningcutlery.com/images/dynamorio/drlogo.png)

## Dr. Memory Performance

Dr. Memory is faster than comparable tools, including Valgrind, as shown in
our [CGO 2011](http://www.cgo.org) paper [Practical Memory Checking with
Dr. Memory](http://www.burningcutlery.com/derek/docs/drmem-CGO11.pdf),
where we compare the two tools on Linux on the SPECCPU 2006 benchmark
suite:

![Performance chart](http://dynamorio.org/images/drmem-spec2k6-sm.png)

(Valgrind is unable to run 434.zeusmp and 447.dealII).

## Documentation

Documentation is included in the release package.  We also maintain a copy
for [online browsing](http://drmemory.org/docs/).

## System call tracer for Windows

The Dr. Memory package includes [an "strace for Windows" tool called
`drstrace`](http://drmemory.org/strace_for_windows.html).

## Obtaining help

Dr. Memory has its own [discussion
list](http://groups.google.com/group/DrMemory-Users).

To report a bug, use the [issue
tracker](https://github.com/DynamoRIO/drmemory/issues).

See also [the Dr. Memory home page](http://drmemory.org/): [http://drmemory.org/](http://drmemory.org/)

# 例子
```c

#include <iostream>
 
void test1(); // 内存泄露，动态申请的内存未释放，就丢失了指向其内存块的指针，直到程序结束也未释放
void test2(); // 非法访问
void test3(); // 未初始化读
void test4(); // Heap 操作参数错误(Invalid Heap Argument)
 
int main()
{
	// reference: http://www.ibm.com/developerworks/cn/linux/1309_liuming_drmemory/
	test4();
 
	std::cout << "ok" << std::endl;
	return 0;
}

// 内存泄露，动态申请的内存未释放，就丢失了指向其内存块的指针，直到程序结束也未释放
void test1()
{
	char *ptr;// 字符型指针 
	for (int i = 0; i<100; i++) {
		ptr = (char*)malloc(i);// 在堆中动态申请 i个字节
 
		if (i % 2) free(ptr);// 只释放第偶数次 申请的内存， 奇数次申请的内存未释放
	}
}

// 非法访问, 越界访问数组，越界访问指针指向的空间，对已经释放的指针进行解引用等
void test2()
{
	char *x = (char*)malloc(8);
	char c = *(x + 8); // 超过申请的内存空间(0~7) buffer overlow
	free(x);           // 释放指针指向的空间
  // free 之后及时将 指针置空  x = null;
	c = *x;            // 读取未分配的内存read free memory
}

// 结构体 变量 T
typedef struct T_ {
	char a;
	char b;
}T;

// 未初始化读
void test3()
{
	T a, b;
	char x;
	a.a = 'a';
	a.b = 'b';
	b.a = x; // error C4700:使用了未初始化的局部变量x,若使vs2013能够正常编译，需将配置属性中的C/C++ SDL检查关闭
	if (b.a == 10)
		memcpy(&b, &a, sizeof(T));
}
// 双重释放内存   DFM
void test4()
{
	char * ptr = NULL;
	ptr = new char;
	free(ptr);
	free(ptr); // 双重释放内存 
}
```
