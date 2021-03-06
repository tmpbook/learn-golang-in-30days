# 第二章 程序结构

### 2.3 变量
``` golang
var s string
fmt.Println(s)
// ""

var i, j, k int
var b, f, s = true, 2, 3, "four"

var f, err = os.Open(name)
// os.Open returns a file and an error
```

#### 2.3.1 简短变量声明
``` golang
// 变量类型根据表达式来自动推导
anim := gif.GIF{LoopCount: nframes}
freq := rand.Flout64() * 3.0
t := 0.0

i := 100
var boiling flout64 = 100
// 应该是字符串列表
var names []string
var err error
var p Point

// 和 var 声明语句一样，简短变量声明语句也可以用来声明和初始化一组变量；
i, j := 0, 1
// 加深记忆：:= 是变量声明语句，而 = 是变量赋值操作

i, j = j, i
// 交换 i 和 j 的值

// 简短变量声明语句也可以用函数的返回值来声明和初始化变量：
f, err := os.Open(name)
if err != nil {
    return err
}
// use file
f.close()
这里和 Python 一样
```

> 这里有一个比较微妙的地方：简短变量声明左边的变量可能并不完全都是刚刚声明的。如果有一些已经在相同的语法域声明过了，那么简短变量声明语句对这些已经声明过的变量就只有赋值行为了。

``` golang
// 在下面代码中，第一个语句声明了 in 和 err 两个变量。
// 第二个语句只声明了 out 一个变量，然后对已经声明的 err 进行了赋值操作。
in, err := os.Open(infile)
out, err := os.Create(outfile)

// 简短变量声明语句中必须至少要声明一个新的变量
f, err := os.Open(infile)
f, err := os.Create(outfile)
// complie error: no new variables
// 解决方法是第二个简短变量语句改用普通的多重赋值

// 当然，这些是在同级词法域的基础上
```

#### 2.3.2. 指针
> 一个指针的值是另一个变量的地址。一个指针对应变量在内存中的存储位置。并不是每一个值都会有一个内存地址，但是对于每一个变量必然有对应的内存地址。通过指针，我们可以直接读或者更新对应变量的值，而不需要知道该变量的名字（如果变量有名字的话）。
```golang
var x int
// &x 将产生一个指向该整数变量的指针，指针对应的数据类型是 *int
// 被称为『指向 int 类型的指针』

// 如果指针名字为 p，那么可以说『 p 指针指向变量 x 』，或者
// 『p 指针保存了 x 变量的内存地址』

// *p 表达式对应 p 指针指向的变量的值

// 例子：
x := 1
p := &x         // p, of type *int, points to x
fmt.Println(*p) // "1"
*p = 2          // equivalent to x = 2
fmt.Println(x)  // "2"
```

> 任何指针的零值都是 nil。如果 p 指向某个有效变量，那么 p != nil 测试为真。指针之间也可以进行测试的，只有当它们指向的同一个变量或全部是 nil 时才相等。

```golang
var x, y int
fmt.Println(&x == &x, &x == &y, &x == nil) // "true false false"
```

> 在 go 语言中，返回函数中局部变量的地址也是安全的，调用 f 函数时创建局部变量 v，在局部变量地址被返回之后依然有效，因为指针 p 依然引用这个变量。
```golang
var p = f()

func f() *int {
    v := 1
    return &v
}
// 每次调用 f 函数都将返回不同的结果：
fmt.Println(f() == f()) // "false"
```

> 通过指针更新变量值（模拟 ++i )
```golang
func incr(p *int) int {
    *p++
    return *p
}

v := 1
incr(&v)              // side effect: v is now 2
fmt.Println(incr(&v)) // "3" (and v is 3)
```

#### new 函数
另一个创建变量的方法是调用内建的new函数。表达式new(T)将创建一个T类型的匿名变量，初始化为T类型的零值，然后返回变量的地址，返回的指针类型为*T。
```golang
p := new(int)   // p，*int类型，指向匿名的int变量
fmt.Println(*p) // "0"
*p = 2          // 设置 int 匿名变量的值为 2
fmt.Println(*p) // "2"
```
用new创建变量和普通声明语句方式创建变量没什么区别，除了不需要声明一个临时的变量名字外，我们还可以在表达式中使用new(T)。换言之，new函数类似一种语法糖，而不是新的基础概念。
下面是例子：
```golang
func newInt() *int {
    return new(int)
}

func newInt() *int {
    var dummy int
    return &dummy
}
```
每次调用new返回一个新的变量地址，下面是不同的地址：
```golang
p := new(int)
q := new(int)
fmt.Println(p == q) // false
```
注意：也有特殊情况，如果两个类型都是空的，类型大小是0，例如struct{}和[0]int，有可能有相同的地址

#### 变量的生命周期
```golang
for t := 0.0; t < cycles*2*math.Pi; t += res {
    x := math.Sin(t)
    y := math.Sin(t*freq + phase)
    img.SetColorIndex(
        size+int(x*size+0.5), size+int(y*size+0.5),
        blackIndex, // 最后插入的逗号不会导致编译错误，这是Go编译器的一个特性
    )
}
```
在每次循环开始会创建临时变量t，然后在每次循环迭代中创建临时变量x和y。

那么Go语言的自动垃圾收集器是如何知道一个变量是何时可以被回收的呢？这里我们可以避开完整的技术细节，基本的实现思路是，从每个包级的变量和每个当前运行函数的每一个局部变量开始，通过指针或引用的访问路径遍历，是否可以找到该变量。如果不存在这样的访问路径，那么说明该变量是不可达的，也就是说它是否存在并不会影响程序后续的计算结果。

因为一个变量的有效周期只取决于是否可达，因此一个循环迭代内部的局部变量的生命周期可能超出其局部作用域。同时，局部变量可能在函数返回之后依然存在。

编译器会自动选择在栈上还是在堆上分配局部变量的存储空间，但可能令人惊讶的是，这个选择并不是由用var还是用new声明变量的方式决定的。
```golang
var global *int

func f() {
    var x int
    x = 1
    global = &x
}

func g() {
    y := new(int)
    *y = 1
}
```
f函数里的x变量必须在堆上分配，因为它在函数退出后依然可以通过包一级的global变量找到，虽然它是在函数内部定义的；用Go语言的术语说，这个x局部变量从函数f中逃逸了。相反，当g函数返回时，变量`*y`将不可达，也就是说可以马上被回收。因此， `*y`并没有从函数g中逃逸，编译器可以选择在栈上分配`*y`的存储空间（也可以选择在堆上分配，然后由go语言的GC回收这个变量的内存空间），虽然这里用的是new方式。

### 2.4 赋值
使用赋值语句可以更新一个变量的值，最简单的赋值语句是将要被赋值的变量放在=的左边，新值的表达式放在=的右边。
```golang
x = 1                       // 命名变量的赋值
*p = true                   // 通过指针间接赋值
person.name = "bob"         // 结构体字段赋值
count[x] = count[x] * scale // 数组、slice或map的元素赋值
```
特定的二元算数运算符和赋值语句的符合操作有一个间接形式，例如上面的最后的语句可以重写为
```golang
count[x] *= scale
```
#### 2.4.1 元组赋值
```golang
x, y = y, x
a[i], a[j] = a[j], a[i]
```

或者是计算两个整数值得最大公约数（GCD）- greatest common divisor
欧几里得的GCD是最早的非平凡算法：
```golang
func gcd(x, y int) int {
    for y != 0 {
        x, y = y, x%y
    }
    return x
}
```
或者是计算斐波那契数列（Fibonacci）的第N个数：
```golang
func fib(n int) int {
    x, y := 0, 1
    for i := 0; i < n; i++ {
        x, y = y, x+y
    }
    return x
}
```
元组赋值也可以这样（同Python）
```golang
i, j, k = 2, 3, 5
f, err = os.Open("foo.txt") // function call returns two values
```
通常这类函数会用额外的返回值来表达某种错误类型，例如`os.Open`是用额外的返回值返回一个error类型的错误，还有一些是用来返回bool值，通常被称为ok。
```golang
v, ok = m[key]  // map lookup
v, ok = x.(T)   // type assertion
v, ok = <-ch    // channel receice

v = m[key]      // map 查找，失败返回零值
v = x.(T)       // type断言，失败时panic异常
v = <-ch        // 管道接收，失败是返回零值（阻塞不算失败）

_, ok = m[key]  // 丢弃字节数
_, ok = x.(T)   // 只检测类型，忽略具体值
```
#### 2.4.2 可赋值性
> 赋值语句是显示的赋值形式，但是程序中还有很多地方会发生隐式的赋值行为：函数调用会隐式的将调用的参数的值赋给函数的参数变量，一个返回语句会隐式的将返回操作的值赋值给结果变量，一个符合类型的字面量也会产生赋值行为。例如：

```golang
medals := []string{"gold", "silver", "bronze"}
// 类似
medals[0] = "gold"
medals[1] = "silver"
medals[2] = "bronze"
```
规则：类型必须完全匹配，`nil` 可以赋值给任何指针或引用类型的变量

值得注意的是，对于两个值是否可以用`==`或`!=`进行比相等比较的能力也和可赋值能力有关系：对于任何类型的值的相等比较，第二个值必须是对第一个值类型对应的变量是可赋值的，反之亦然。

### 2.5 类型
```golang
gopl.io/ch2/tempconv0

// Package tempconv performs Celsius and Fahrenheit temperature computations. package tempconv

import "fmt" 

type Celsius float64 // 摄氏温度 
type Fahrenheit float64 // 华氏温度

const ( 
    AbsoluteZeroC Celsius = -273.15 // 绝对零度 
    FreezingC Celsius = 0 // 结冰点温度 
    BoilingC Celsius = 100 // 沸水温度 
)

func CToF(c Celsius) Fahrenheit { return Fahrenheit(c*9/5 + 32) } 
func FToC(f Fahrenheit) Celsius { return Celsius((f - 32) * 5 / 9) }
```

比较运算符`==`和`<`也可以用来比较一个命名类型的变量和另一个有相同类型的变量，或有着相同底层类型的未命名类型的值之间做比较。但是如果两个值有不同的类型，则不能直接进行比较：
```golang
var c Celsius
var f Fahrenheit
fmt.Println(c == 0)         // "true"
fmt.Println(f >= 0)         // "true"
fmt.Println(c == f)         // compile error: type mismatch
fmt.Println(c == Celsius(f))// "true"
// Celsius(f)是类型转换操作，并不会改变值，仅仅是改变值的类型而已
```

```golang
func (c Celsius) String() string { return fmt.Sprintf("%g℃", c) }
// 许多类型都会定义一个String方法，因为当使用fmt包的打印方法时，将会优先使用该类型对应的String方法返回的结果打印（作者注，这里和 Python 一样，类型有内置函数，可以重写）
```

```golang
c := FToC(212.0)
fmt.Println(c.String()) // "100℃"
fmt.Printf("%v\n", c)   // "100℃"; no need to call String explicitly
fmt.Printf("%s\n", c)   // "100℃"
fmt.Println(c)          // "100℃"
fmt.Printf("%g\n", c)   // "100"; does not call String
fmt.Printf(float64(c))  // "100"; does not call String
```

### 2.6 包和文件

每个包都对应一个独立的名字空间。例如：在image包中的Decode函数和在unicode/utf16包中的Decode函数时不同的，要在外部引用该函数，必须显式的使用image.Decode或utf16.Decode形式访问。

在Go语言中，一个简单的规则是：如果一个名字是大写字母开头的，那么该名字是导出的。

#### 2.6.1 导入包
在Go语言程序中，每个包都是有一个全局唯一的导入路径。导入语句中类似"gopl.io/ch2/tempconv"的字符串对应包的导入路径。Go语言的规范并没有定义这些字符串的具体含义或者包来自哪里，他们是由构建工具来解释的。当使用Go语言自带的go工具箱时，一个导入路径代表一个目录中的一个或多个Go源文件。

```golang
package main

import (
    "fmt"
    "os"
    "strconv"

    "gopl.io/ch2/tempconv"
)

func main() {
    for _, arg := range os.Args[1:] {
        t, err := strconv.ParseFloat(arg, 64)
        if err != nil {
            fmt.Fprintf(os.Stderr, "cf: %v\n", err)
            os.Exit(1)
        }
        f := tempconv.Fahrenheit(t)
        c := tempconv.Celsius(t)
        fmt.Printf("%s = %s, %s = %s\n",
            f, tempvonc.FToC(f), c, tempconv.CToF(c))
    }
}
```

#### 包的初始化

包的初始化首先是解决包级变量的依赖顺序，然后按照包级变量声明出现的顺序依次初始化：

```golang
var a = b + c // a 第三个初始化，为3
var b = f()   // b 第二个初始化，为2，通过调用f（依赖c）
var c = 1     // c 第一个初始化，为1

func f() int { return c + 1 }
```
如果包中含有多个.go源文件，它们将按照发给编译器的顺序进行初始化，Go语言的构建工具首先会将.go文件根据文件名排序，然后依次调用编译器编译。

对于在包级别声明的变量，如果有初始化表达式则用表达式初始化，还有一些没有初始化表达式的，例如某些表格数据初始化并不是一个简单的赋值过程。在这种情况下，我们可以用一个特殊的init初始化函数来简化初始化工作。每个文件都可以包含多个init初始化函数
```golang
func init() { /* ... */}
```
这样的init初始化函数除了不能被调用或引用外，其他行为和普通函数类似。程序开始执行时按照它们的声明顺序被自动调用。

每个包在解决依赖的前提下，以导入声明的顺序初始化，每个包只会被初始化一次。因此，如果p包导入了q包，那么在p包初始化的时候可以认为q包必然已经初始化过了。初始化工作是自上而下进行的，main包最后被初始化。以这种方式，可以确保在main函数执行之前，所有依赖的包都已经完成初始化工作。

```golang
// 下面代码定义了一个PopCount函数，用于返回一个数字中含二进制 1bit 的个数。它使用 init 初始化函数来
// 生成辅助表格 pc，pc 表格用于处理每个 8bit 宽度的数字含二进制的 1bit 的 bit 个数，这样的话在处理
// 64bit 宽度的数字时就没有必要循环 64 此， 只需要8次查表就可以了。
```

#### 作用域
```golang
func main() {
    x := "hello!"
    for i := 0; i < len(x); i++ {
        x := x[i]
        if x != [!] {
            x := x + 'A' - 'a'
            fmt.Printf("%c", x) // "HELLO" (one letter per iteration)
        }
    }
}

func main() {
    x := "hello"
    for _, x := range x {
        x := x + 'A' - 'a'
        fmt.Printf("%c", x)
    }
}

if x := f(); x == 0 {
    fmt.Println(x)
} else if y := g(x); x == y {
    fmt.Println(x, y)
} else {
    fmt.Println(x, y)
}
fmt.Println(x, y) // compile error: x and y are not visible here
```
> 第二个if语句嵌套在第一个内部，因此第一个if语句条件初始化词法域声明的变量在第二个if中也可以访问。switch语句的每个分支也有类似的语法域规则：条件部分为一个隐式语法域，然后每个是每个分支的语法域。

在看一个例子：

在这个程序中
```golang
if f, err := os.Open(fname); err != nil { // compile error: unused: f
    return err
}
f.ReadByte() // compile error: undefined f
f.Close()    // compile error: undefined f
```
变量f的作用域只有if语句内，因此后面的语句将无法引入它，这将导致编译错误。你可能会收到一个局部变量f没有生命的错误提示，具体错误信息依赖编译器的实现。

通常需要再if之前声明变量，这样可以确保后面的语句依然可以访问变量：
```golang
f, err := os.Open(fname)
if err != nil {
    return err
}
f.ReadByte()
f.Close()
```

你可能会考虑通过将`ReadByte`和`Close`移动到if的else块来解决这个问题：
```golang
if f, err := os.Open(fname); err != nil {
    return err
} else {
    // f and err are visible here too
    f.ReadByte()
    f.Close()
}
```
!!!但这不是Go语言推荐的做法，Go语言的习惯是在if中处理错误然后直接返回，这样可以确保正常执行的语句不需要代码缩进（这让我想到了JavaScript中的promise）

有许多方式可以避免出现类似潜在的问题。最直接的方法是通过单独声明err变量，来避免使用`:=`的简短声明方式
```golang
var cwd string
func init() {
    var err error
    cwd, err = os.Getwd()
    if err != nil {
        log.Fatalf("os.Getwd failed: %v", err)
    }
}
```
