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

8.  `for` 循环只有一种形式。初始化语句仅会被执行一次。注意初始化语句定义变量是局部变量。

```go
package main

func main() {
	for index := 0; index < count; index++ {
		// 初始化表达式支持函数调用或定义局部变量
	}

	for x < 10 {
		// 类似 "while x < 10" 或 "for ; x < 10; {}"
	}

	for {
		break
	}
}
```

9. `for...range` 完成数据迭代，支持字符串、数组、数组指针、切片、字典、通道类型，返回索引、键值数据，允许返回单值，或者用 `_` 来忽略前者。

| 数据类型    | 1st value | 2st value | 备注          |
| ----------- | --------- | --------- | ------------- |
| string      | index     | s[index]  | unicode, rune |
| array/slice | index     | v[index]  |               |
| map         | key       | value     |               |
| channel     | element   |           |               |

10. 不管是普通的 `for` 循环还是 `for......range` 迭代，其定义的局部变量都会重复使用。
11. 使用 `goto` 前，必须先定义标签。标签区分大小写，且未使用的标签会引发编译错误。`goto` 不能用于跳转到其他函数，或者内部代码块内。

```go
package main

import (
	"fmt"
)

func test() {
test:
	println("test")
	fmt.Println("test Exit.")
}

func main() {
	for index := 0; index < 3; index++ {
	loop:
		print(index)
	}

	goto test // label test not defined
	goto loop // goto loop jumps into block starting at src/test/main.go:14:37
}
```

12. 与 `goto`  定点跳转不同，break、continue 用于中断代码块执行，他们配合标签使用可以实现多层嵌套中指定目标层级。

- `break`：用于 `switch`、`for`、`select` 语句，终止整个语句执行；
- `continue`：仅用于 `for` 循环，终止后续逻辑，进入下一次循环。

```go
package main

import "fmt"

func main() {
outer:
	for x := 0; x < 5; x++ {
		for y := 0; y < 10; y++ {
			if y > 2 {
				println()
				continue outer
			}
			if x > 2 {
				break outer
			}

			fmt.Println(x, y)
            // 0 0 0 1 0 2
            // 1 0 1 1 1 2
            // 2 0 2 1 2 2
		}
	}
}
```

### 第四章 函数

1. 函数是结构化编程的最小模块单元，函数是代码复用和测试基本单位。使用关键词 `func` 来定义。有以下优点：

- 无需前置声明；
- 不支持命名嵌套定义（nested）// 即函数里面不能在定义函数
- 不支持同名函数重载（overload）
- 不支持默认参数
- 支持不定长变参
- 支持多返回值
- 支持命名返回值
- 支持匿名函数和闭包

2. 函数属于第一类对象（可以在运行期间创建，可以用作函数的参数或返回值，可以存入变量的实体。常见的是匿名函数），具有相同签名（参数及返回值列表）的视为同一类型。从阅读和代码维护角度来说，使用命名类型更加方便。函数只能判断其是否等于 `nil`，不支持其他比较。

3. 从函数中返回局部变量指针是安全的，编译器会通过 **逃逸分析** 来决定是否在堆上分配内存。

```go
package main

import (
	"fmt"
)

func test() *int {
	a := 100
	return &a
}

func main() {
	var a *int = test()
	fmt.Println(a, *a)
}
```

4. 函数命名规范，在避免冲突的情况下，尽可能的本着精简短小、望文知意的原则，（函数与方法命名规则有所不同，方法通过选择符调用，具备状态上下文，可使用更加简短的动词命名）：

- 动词 + 介词 + 名词， 如 scanWords
- 避免不必要的缩写，如 printError 比 printErr 更好一些
- 避免使用类型关键词，如 buildUserStruct 看上去很别扭
- 避免歧义，不能有多重用途的解释容易造成误解
- 避免只能通过大小写区分的同名函数
- 避免使用数字，除非是特定专有名词，例如 UTF-8
- 避免添加作用域提示前缀
- 统一使用  camel/pascal case 拼写风格
- 使用相同的术语，保持统一性
- 使用习惯语法，比如 init 表示初始化， is/has 返回布尔值结果
- 使用反义词组命名行为相反的函数，如 get/set，min/max 等

5. 对于参数，Go 语言不支持有默认值的可选参数，不支持命名实参。调用时，必须按照签名顺序传递指定类型和数量的实参，就算是 `_` 命名的参数也不能省略。参数列表中，相邻的同类型参数可以合并。参数可是为函数局部变量，因此不能在相同层级定义同名变量。（形参是函数定义中的参数，实参是函数调用是传入的参数。形参类似函数局部变量，而实参是函数外部对象，可以是常量、变量、表达式或函数等）

```go
package main

import (
	"fmt"
)

func test(x, y int, s string, _ bool) *int {
	x := 100 // no new variables on left side of :=
	y := 25
	var s string // s redeclared in this block
	z := x + y
	return &z
}

func main() {
	test(1, 2, 'tsingwong') // not enough arguments in call to test
}
```

6. 函数中不管是指针、引用类型，还是其他类型参数，都是**值拷贝传递**（pass-by-value），函数调用之前，会为形参和返回值分配内存空间，并将实参拷贝到形参内存。

```go
package main

import (
	"fmt"
)

func test(x *int) {
	fmt.Printf("pointer: %p, target: %v\n", &x, x)
    // pointer: 0xc000082028, target: 0xc00006c008
}

func main() {
	a := 0x100
	p := &a
	fmt.Printf("pointer: %p, target: %v\n", &p, p)
	// pointer: 0xc000082018, target: 0xc00006c008    
	test(p)
}
```

7. 要实现传出参数，通常建议使用返回值，当然也可以使用二级指针。参数过多时，通常建议将其重构成一个复合结构类型，也算是变相实现可选参数和命名实参功能。

```go
package main

import (
	"fmt"
	"log"
	"time"
)

type serverOption struct {
	address string
	port    int
	path    string
	timeout time.Duration
	log     *log.Logger
}

func newOption() *serverOption {
	return &serverOption{
		address: "0.0.0.0",
		port:    8080,
		path:    "/var/test",
		timeout: time.Second * 5,
		log:     nil,
	}
}

func server(option *serverOption) {
	fmt.Println(option.port)
}

func test(p **int) {
	x := 100
	*p = &x
}

func main() {
	var p *int
	test(&p)
	fmt.Printf("%v \n", *p) // 100

	opt := newOption()
	opt.port = 8085
	server(opt) // 8085
}
```

8. 变参实质上是切片，只能接收一到多个同类型参数，且必须放在列表尾部。将切片作为变参时，必须进行展开操作。如果是数组，需要先将其转为切片。变参复制的仅仅是切片自身，不包括底层数组，因此可以修改原始数据。如果需要的话，可以使用内置函数 `copy` 复制底层数据。

```go
package main

import (
	"fmt"
)

func test(s string, a ...int) {
	fmt.Printf("%T, %v\n", a, a) // 显示类型和值 []int
}

func test2(a ...int) {
	for i := range a {
		a[i] += 100
	}
}

func main() {
	test("tsingwong", 1, 2, 3, 4, 5, 6)
	a := [3]int{100, 200, 300}
	test("tsingwong", a[:]...) // 展开 slice
	// var a2 []int
	var a2 []int = make([]int, 3)
	copy(a2, []int{1, 2, 3})
	test2(a[:]...)
	fmt.Println(a)  // [200, 300, 400]
	fmt.Println(a2) // [1, 2, 3]
}
```



