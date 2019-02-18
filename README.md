# 《GO 语言学习笔记》学习笔记

### 第一章 概述

1.  可以用  `defer` 定义延迟调用，无论函数是否出错，它都会确保结束前调用。

-   常用来释放资源、解除锁定、或执行一些清理操作

-   可以定义多个 defer，按照 FILO （先进后出，栈）执行

```go
package main

func test(a, b int) {
	defer println("dispose>>>")

	println(a / b)
}

func main() {
	test(10, 0)
}
```



2.  `ok-idiom`模式，是指在多返回值中用一个名为 ok 的布尔值来表示操作是否成功。因为很多操作默认返回**零值**，所需需要额外说明。
3.  结构体 `struct` 可以匿名嵌入其他类型，也可以调用匿名字段的方法，多用于实现与继承类似的功能

```go
package main

import "fmt"

type user struct {
	name string
	age  byte
}

func (u user) ToString() string {
	return fmt.Sprintf("%+v", u)
}

type manage struct {
	user
	title string
}

func main() {
	var m manage
	m.name = "tsingwong"
	m.age = 29
	fmt.Println(m.ToString())
}
```

4.  **并发** ：整个运行时完全并发化设计，几乎都是在以 `goroutine`方式运行。这是一种比普通协程或线程更加高效的并发设计，能轻松创建和运行成千上万的并发任务。

### 第二章 类型

1.  Go 是一门静态语言，类型决定了变量内存的长度和储存格式。一经创建只能修改变量的值，无法改变类型。
2.  关键词 `var` 用于定义变量，类型放在变量名后，运行时内存分配操作会确保变量自动初始化为二进制零值，避免出现不可预期的行为。未使用的局部变量会引发编译错误。

简短模式的弊端：

-   定义变量，同时显示初始化
-   不能提供数据类型
-   只能用于函数内部（`var`可以用在全局作用域）

```go
var a int
var b, c int
var d, e = 100, "tsingwong"
var (
	f, g int
    h, i = 100, "tsingwong"
)

// 简短模式
j := 100
h, i = 1, "tsingwong"
```

3.  多变量赋值操作时，首先会计算出所有右边值，然后再依次完成赋值操作。赋值操作，必须确保左右值类型一致。
4.  变量命名：以字母或下划线开始，可以由多个字母、数字和下划线组成；区分大小写；使用驼峰（camel case）拼写。首字母大小写决定了其作用域，首字母大写即为可导出成员，可以被包外引用，反之小写只能在包内使用。[推荐命名查询网站](https://unbug.github.io/codelf/)
5.  `_` 空标识符：通常用作忽略用的占位符，可以作为表达式左值，无法读取其内容。多用来临时规避编译器对未使用变量和导入包的错误检查。
6.  常量：必须是编译器可确定的字符、字符串、数字或布尔值。不曾使用的常量不会引发编译错误。显式指定类型时，必须确保常量左右值类型一致。常量组中不指定类型和初始值，则与上一行的非空常数右值相同。
7.  Go 语言中没有明确意义上的 `enum` 定义，即枚举，可以借助 `iota` 标识符来实现自增常量的枚举类型。 `iota` 中断自增后，需要显式恢复，且后续自增值按行序递增。

```go
const(
	a = iota 	// 0
    b		 	// 1
    c = 100		// 100
    d			// 100
    e = iota	// 4
    f			// 5
)
```

8. 变量在运行期分配存储内存，而常量通常会被编译器在**预处理**阶段直接展开，作为指令数据使用。
9. 有如下预定义的基本类型：

| 类型          | 长度 | 默认值 |           说明            |
| :------------ | ---- | ------ | :-----------------------: |
| bool          | 1    | false  |                           |
| byte          | 1    | 0      |           unit8           |
| Int, uint     | 4, 8 | 0      |       默认整数类型        |
| int8, uint8   | 1    | 0      |     -128~127, 0 ~255      |
| int16, uint16 | 2    | 0      |  -321768~32767, 0 ~65535  |
| int32, uint32 | 4    | 0      |    -21亿~21亿，0~42亿     |
| int64, uint64 | 8    | 0      |                           |
| float32       | 4    | 0.0    |                           |
| float64       | 8    | 0.0    |     默认浮点数的类型      |
| complex64     | 8    |        |                           |
| complex128    | 16   |        |                           |
| rune          | 4    | 0      | Unicode Code Point, int32 |
| uintptr       | 4, 8 | 0      |    足以存储指针的 uint    |
| string        |      | ""     |  字符串，默认是空字符串   |
| array         |      |        |           数组            |
| struct        |      |        |          结构体           |
| function      |      | nil    |           函数            |
| interface     |      | nil    |           接口            |
| map           |      | nil    |      字典，引用类型       |
| slice         |      | nil    |      切片，引用类型       |
| channel       |      | nil    |      通道，引用类型       |

10. 标准库 `strconv` 可以用于转换不同进制的字符串。浮点数需要注意小数位的有效精度。
11. Go 中只有三种引用类型，slice、map、channel。引用类型的数据除了需要分配内存，他们还需要初始化指针、长度等等属性，甚至包括哈希分布、数据队列等等。
12. 值类型结构使用 `new` 按照指定类型长度分配零值内存，返回指针，并不关心类型内部的构造和初始化方式；而引用类型必须要使用 `make` 函数创建，编译器会将 `make` 转换为目标类型专门的创建函数，以确保完成全部内部分配和相关属性初始化。
13. 除了变量、别名类型以及未命名的类型外， Go 强制要求使用显式类型转换。
14. 如果转换的目标是指针、单向通道或没有返回值的函数类型，那么必须使用括号，以避免造成语法分解错误。
15. 可以使用关键词 `type` 来定义用户自定义类型，包括基于现有基础类型创建，或是结构体、函数类型等。

### 第三章 表达式

1. Go 语言仅有 **25** 个保留关键字，保留关键字不能用作常量、变量、函数名、以及结构字段等标识符：

```go
break		default		func	interface	select
case		defer		go		map			struct
chan		else		goto	package		switch
continue	for			import	return		var
```

2. Go 语言的全部运算符及分隔符如下：

```go
+		&		+=		&=		&&		==		!=		(		)	
-		|		-=		|=		||		<		<=		[		]
*		^		*=		^=		<-		>		>=		{		}
/		<<		/=		<<=		++		=		:=		,		;
%		>>		%=		>>=		--		!		...		.		:
		*^				&^=
```

3. 指针与内存地址不能混为一谈。内存地址是内存中每个字节单元的唯一编号，而指针是一个实体。指针会分配内存空间，相当于一个专门用来保存地址的整型变量。

- 取址运算符 `&`：用于获取对象地址
- 指针运算符 `*`：用于间接引用目标对象
- 二级指针 `**T` ：如包含包名则写成 `*pacakge.T`

4. 指针没有专门的指向成员 `->` 运算符，统一使用 `.` 操作符。
5. 对于复合类型（数组、切片、字典、结构体等）初始化时有以下限制：

- 初始化表达式必须含类型标签
- 左花括号必须在类型尾部，不能另起一行
- 多个成员初始值以逗号分隔
- 允许多行，但每行须以逗号或右花括号结束

```go
package main

import "fmt"

type data struct {
	x int
	s string
}

func main() {
	var a = data{1, "tsingwong"}
	b := data{
		1,
		"abc",
	}

	c := []int{
		1, 2, 3,
		4, 5}
	fmt.Println(a, b, c)
}
```

6. `if...else` 中条件表达式必须是布尔值，可以省略括号，且左花括号不能另起一行，支持块局部变量或执行初始化函数，初始化可以用于错误处理和后续正常操作。注意Go语言中没有条件运算符 `a>b?a:b`。

```go
package main

import "fmt"

func main() {
	var m = map[string]string{
		"name": "tsingwong",
		"age":  "26",
	}

	if ch, ok := m["name1"]; ok {
		fmt.Println(ch)
	} else {
		fmt.Println("not find")
	}
}
```

7. `swtich` 同样支持初始化语句，按从上到下，从左到右顺序匹配 `case` 语句，只有全部不匹配才会执行 `default` 块。相邻的 `case` 语句不构成多条件匹配，不可以出现重复的 `case` 常量；无需要显示执行 `break` 语句，`case` 完毕后自动中断。如果需要连贯后面的 case 语句，需要执行 `fallthrough` 语句，但是不会匹配后续条件表达式。注：`fallthrough` 语句会被 `break` 语句阻止，所以要放在 `case` 块的结尾。

```go
package main

import "fmt"

func main() {
	switch x := 5; x {
	case 5:
		x += 10
		fmt.Println(x)

		// if x >= 15 {
		// 	break 
		// }

		fallthrough
	case 6:
		x += 20
		fmt.Println(x) // 35
	case 7:
		x += 20
		fmt.Println(x) // 这个不会输出哦
	}
}
```

8. 

```go

```

