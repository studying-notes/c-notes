---
date: 2022-10-27 15:23:00+08:00  # 创建日期
author: "Rustle Karl"  # 作者

title: "数组"  # 文章标题
url:  "posts/cpp/interview/language/array"  # 设置网页永久链接
tags: [ "C", "C++" ]  # 标签
series: [ "C 学习笔记" ]  # 系列
categories: [ "学习笔记" ]  # 分类

toc: true  # 目录
draft: false  # 草稿
---

## `int a[2][2]={{1},{2,3}}`，则 `a[0][1]` 的值是多少

原则上来说，二维数组在内存中既可以按行存储，也可以按列存储，但一般是按行存放的，即先存储第一行的数组元素的值，再按顺序存放第二行的数组元素的值，以此类推。

```c++
#include <iostream>

int main() {
  int a[2][2] = {{1}, {2, 3}};
  std::cout << a[0][0] << a[0][1] << a[1][0] << a[1][1] << std::endl;
  return 0;
}
```

```c++
1023
```

本题中，由于数组 a 是一个二维数组，第一行第一个数是 1，第二行第一个数是 2，第二个数是 3，**其他位置数的值自动补 0**，所以第一行第二个数也就是 `a[0][1]`，它的值是 0。

二维数组的初始化一般有两种方式，第一种方式是按行来执行，而第二种方式是把数值写在一块。

```c++
#include <iostream>

int main() {
  int a[2][2] = {{1,2}, {3, 4}};
  int b[2][2] = {1, 2, 3, 4};
  return 0;
}
```

有一种情况比较特殊，如果对二维数组的所有元素都赋值，则数组的第一维的长度可以不指定，但第二维的长度不能省。

```c++
#include <iostream>

int main() {
  int a[][2] = {1, 2, 3, 4};
  return 0;
}
```

需要注意的是，只有在进行带有初始化数据的数组说明时，才允许省略第一维长度，在仅仅进行说明数组而未进行初始化数组元素时，省略长度是错误的，因为编译系统无法预知数组大小，如 `int a[][4]` 就是错误的。

## 如何合法表示二维数组

对于数组 `a[3][4]`，`*(a[1]+1)`、`*(&a[1][1])`、`(*(a+1))[1]` 和 `*(a+5)` 四种表示方法中，哪个不能表示 `a[1][1]`？

第一个可以，因为 `a[1]` 是第 2 行的地址，`a[1]+1` 偏移一个单位（得到第 2 行第 2 列的地址），然后解引用取值，得到 `a[1][1]`。

第二个也可以，`[]` 优先级高，`a[1][1]` 取地址再取值。

第三个 `a+1` 相当于 `&a[1]`，所以 `*(a+1)=a[1]`，因此 `*(a+1)[1]=a[1][1]`。

第四个 `a+5` 相当于 `&a[5]`，单从这里看就已经越界了，所以不可以。指针类型的大小不一样，a 是 int[4] 大小，5 代表了 5 个 sizeof(int[4])，所以越界了，除非 5 代表 5 个 sizeof(int)。

```c++
#include <iostream>

int main() {
  int a[3][4] = {1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12};
  std::cout << a[1][1] << std::endl;
  std::cout << *(a[1] + 1) << std::endl;
  std::cout << *(&a[1][1]) << std::endl;
  std::cout << (*(a + 1))[1] << std::endl;
  std::cout << *(*(a + 1) + 1) << std::endl;
  std::cout << *(a + 5) << std::endl;
  std::cout << a[5] << std::endl;
  return 0;
}
```

## a 是数组，(int*)(&a+1) 表示什么意思

对于数组而言，一个数组名代表的含义是数组中第一个元素的位置，即地址，通过数组名可以访问该数组。

一般而言，对指针进行加 1 操作，得到的将是下一个元素的地址。因此，只要确定了指针所指向的类型，就可以很容易地求出指针 +1 的结果。也就是说，一个类型为 T 的指针移动，是以 `sizeof(T)` 为移动单位的。

对于数组 `int a[5];`，a 表示数组首元素的地址，而 `&a` 表示数组的首地址。

a 有两层含义：一是 a 表示数组首元素的地址（类型为 `int*`）；二是 a 表示一个大小为 5 的 int 数组。数组名本身在计算中会自动转化成第一个元素的地址，但 sizeof 测试的时候不做计算，不转化。由此可以看出，在计算 `*(a+1)` 的时候，可以把 a 看成 `int*` 类型的指针，`a+1` 表示的是数组第二个元素的指针，由此 `*(a+1)` 的值为 2。但是，`sizeof(a)` 是把 a 作为一个数组类型来测试大小，结果是数组占用的字节数，由此 `sizeof(a)=20`。

`&a` 是一个指向 `int(*)[5] `的指针。由于 `&a` 是一个指针，那么在 32 位机器上，`sizeof(&a)=4` ，但是 `&a+1` 的值取决于 `&a` 指向的类型，由于 `&a` 指向 `int(*)[5]` ，所以 `&a+1=&a+sizeof(int(*)[5])=&a+20`。`&a` 是数组的首地址，由此 `&a+1` 表示 a 向后移动了 5 个 int 从而指向 `a[5]`，所以 ptr 指向的是 `a[5]`，由于 ptr 是 `int*` 类型，那么 `ptr-1` 就指向 `a[4]`，所以 `*(ptr-1)=a[4]=5`。

由此可知 `(int*)(&a+1)` 指向了数组 a 最后一个元素之后的下一个内存单元开始的 4 个字节 `(int*)` 内存。

## 数组下标可以为负数吗

可以，因为下标只是给出了一个与当前地址的偏移量而已，只要根据这个偏移量能定位得到目标地址即可。

```c++
#include <iostream>

int main() {
  int a[5] = {1, 2, 3, 4, 5};
  int *p = &a[4];
  for (int i = -4; i <= 0; i++) {
    std::cout << i << "=" << p[i] << std::endl;
  }
  return 0;
}
```

```
-4=1
-3=2
-2=3
-1=4
0=5
```

从上例可以发现，在 C 语言中，数组的下标并非不可以为负数，当数组下标为负数时，编译可以通过，而且也可以得到正确的结果，只是它表示的意思却是从当前地址向前寻址，即为当前地址减去 sizeof（类型）的地址值。

## 不使用流程控制语句，如何打印出 1～1000 的整数

递归方式：

```c++
#include <iostream>

void range(int start, int stop) {
  if (start <= stop) {
    std::cout << start << std::endl;
    range(start + 1, stop);
  }
}

int main() {
  range(1, 1000);
  return 0;
}
```

结构体数组声明方式：

```c++
#include <iostream>

struct print {
  static int a;
  print() {
    std::cout << a++ << std::endl;
  }
};

int print::a = 1;

int main() {
  print tt[1000];
  return 0;
}
```

宏定义方式：

```c++
#include <iostream>

#define A(x) \
  x;         \
  x;         \
  x;         \
  x;         \
  x;         \
  x;         \
  x;         \
  x;         \
  x;         \
  x;

int main() {
  int n = 1;
  A(A(A(printf("%d\n", n++);)));
  return 0;
}
```

## 求 1+2+…+n，要求不能使用乘除法、for、while、if、else、switch、case 等关键字以及条件判断语句（A?B:C）

通常针对此类题目除了用公式 n(n+1)/2 之外，还可以用循环和递归两种思路，由于不能使用 for 和 while，循环已经不能再用了。同样，一般的递归函数也需要用 if 语句或者条件判断语句来判断是继续递归下去，还是终止递归，所以一般的递归函数也无法满足本题的需要。

根据本题的思路，可以采用构造函数与静态变量结合的方式，程序示例如下：

```c++
#include <iostream>

struct print {
  static int a, b;
  print() {
    std::cout << (a += b++, a) << std::endl;
  }
};

int print::a = 0;
int print::b = 1;

int main() {
  print tt[10];
  return 0;
}
```

不使用条件判断语句来进行递归条件的判断，则此种递归方法可取，可以利用逻辑与的短路特性来实现递归终止的条件判断，程序示例如下：

```c++
#include <iostream>

int sum(int n) {
  int ans = n;
  ans && (ans += sum(n - 1));
  return ans;
}

int main() {
  std::cout << sum(10) << std::endl;
  return 0;
}
```

## `char str[1024]; scanf("%s",str)` 是否安全

上述代码会存在安全风险，程序的潜在问题是如果用户输入了超过 1024 个长度的字符，那么就会有数组越界的问题。

## 行存储与列存储中哪种存储效率高

```c++
#include <iostream>

int main() {
  const int M = 3;
  const int N = 4;

  int a[M][N];
  int b[M][N];

  // 行存储策略
  for (int i = 0; i < M; i++) {
    for (int j = 0; j < N; j++) {
      a[i][j] = i + j;
    }
  }

  // 列存储策略
  for (int j = 0; j < N; j++) {
    for (int i = 0; i < M; i++) {
      b[i][j] = a[i][j];
    }
  }

  return 0;
}
```

C++ 采用的是行存储策略，数组是一行一行地存的。

```c++

```