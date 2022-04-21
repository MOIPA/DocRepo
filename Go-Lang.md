title: Go Lang Basic
author: tr
tags:
  - Go
categories:
  - Go
date: 2022-04-12 09:26:00
---
## GO

<!--more-->

> go语言学习，记录《Head First Go》的学习内容
>
> 前面几十页都在讲怎么更好的学习……
>
> 编译最快，运行效率最高，便捷分发任务，多线程效率高
>
> 07年google开始立项GO语言，因为每次测试新功能就要编译老版本，编译至少要一小时,所以开发全新的语言，快，不冗余，gc回收，好写多线程，支持多核cpu的语言。

## 基础语法

> package: go使用package来管理同一项目的所有文件，使用 package main作为主程序的入口，且方法名得是main作为入口代码
>
> import: go文件里想要使用其他文件内定义的方法需要使用import,import 包名，这样比起引用整个依赖库更灵活轻量，如果import的包没有被使用过，编译不通过
>
> 标准的go文件布局：
>
> + package `<name>`
>
> + import `"<name>"`
>  
> + actual code : 一般都是由各个方法组成的
>
> 可以用 `;`结尾 随意，以下代码可以 `go fmt`格式化，`go run hello.go`执行

```go
package main
import "fmt"
func main() {
 fmt.Println("hello go from tr")
}
```

> import 的另一种形式

```go
package main

//import "fmt"
import (
 "math"
 "strings"
)

func main() {
 //fmt.Println("hello go from tr","!")
 math.Floor(2.75)
 strings.Title("head first go")
}
```

> import 导入的实际是包的路径，包名不一定需要等于路径名，例如这里，rand包路径在`math/rand`下，且包名是rand,所以调用方法是`rand.Intn(100)`生成随机数，通常情况下包路径的最后一段就和包名同名

```go
//import "fmt"
import (
 "math/rand"
)

```

> 转义符 这个不多说，有意思的是fmt.Print方法里面传入转义符或者其他字符`'`单引号为单字符，会输出这些字符的原始unicode编码
>
> true & false
>
> `+` `-` `*` `/` `==` `>=` `<=` `!=` 都和其他语言一样来，返回`true/flase`
>
> go提供了查看数据类型的工具如下：

```go
package main

import (
 "fmt"
 "reflect"
)

func main() {
 fmt.Println(reflect.TypeOf(42)) // int
 fmt.Println(reflect.TypeOf(3.14)) //float64
 fmt.Println(reflect.TypeOf(true)) //bool
 fmt.Println(reflect.TypeOf("hello,go")) //string
}
```

### go的变量定义和声明

> go的变量有很多
>
```go
int8
int16
int32
int64

unit: 和int一样 只不过只能存储正数，所以占用空间少一半
unit8
unit16
unit32
unit64

float32 :除了float64还有32位的版本
```

```go
package main

import (
 "fmt"
)

func main() {
 var quantity = 4 //可以忽略int
 var length,width float64 = 1.2，2.4
 var name string
 fmt.Println(quantity) // 初始化都是0
 fmt.Println(length)
 fmt.Println(width)
 fmt.Println(name)
}

```

> 定义可以简写 `:=`

```go
package main

import (
 "fmt"
)

func main() {
 //var quantity int = 12
 //var length,width float64 = 1.1,1.2
 //var name string = "test"
 // 简写形式
 quantity := 12
 length, width := 1.1, 1.2
 name := "tr"
 fmt.Println(quantity)
 fmt.Println(length)
 fmt.Println(width)
 fmt.Println(name)
}

```

> go的命名规则
>
> + 变量/方法若以大写字母开头，则被认为是暴露出去的，其他包文件可以访问，反之是私有的只有包内的文件可以访问
> + 不能以数字开头
> + 遵循驼峰法
>
> 注意：数学操作或者比较都需要输入的类型为同一类型，举例int和float64的不能做乘除，需要做转换`float64(变量)`，如果是float64转int，小数会被截掉
>
> go的if判断也和其他语言一样

```go
if true && true {
}else if !true{}
```

### go 的cli工具

> go是这么编译的 一个大概流程

![upload successful](/images/pasted-174.png)

> go build ： 编译，可以发给其他人执行
> go run ： 编译执行，不输出编译结果文件
> go fmt ：格式化
> go version
> go install `<文件夹的名字>`：需要文件夹内有go代码和src目录，且go代码处于main包下，才能在bin文件夹下生成可执行文件
> go get ：想要导入其他人的包就需要用这个，在后面会学到导入其他人的包的组织方式，社区的代码托管在github上，可以通过go get获取，将自动下载到工作区的src下 例子：`go get github.com/headfirstgo/greeting`
> go doc ：导入了他人的包后想要使用，go doc可以查看说明,比如，`go doc strconv`查看包，`go doc strconv ParseFloat`查看包内方法

### 方法调用 时间 字符

> 有意思的是 go可以返回多个值，go规定定义的变量必须要被用到，所以如果返回的多个值有不需要的，可以用下划线`_`代表废弃`name,_ := reader.ReadString()`

```go
package main

import (
 "fmt"
 "time"
 "strings"
)

func main() {
 // 时间调用
 var now time.Time = time.Now()
 fmt.Println(now.Year())
 // 字符调用

 broken := "G# r#cks"
 replacer := strings.NewReplacer("#","o")
 fixed := replacer.Replace(broken)
 fmt.Println(fixed)
}
```


### GO 在if中初始化

> `err := myFunc()` 错误时常会有，有的时候我们会忘记`err`的定义是否要加上`:`，但是实际上`err`很快只用来if判断是否出错，随后就不在使用，这时我们可以在if中直接初始化`err`
>

```go
if err:=myFunc(); err!=nil{
	log.Fatal(err)
}
```

### GO 的switch

> switch 自动加上break，如果要继续往下，加上`fallthrough`

```go
	val := 10

	switch val {
	case 1:
		fmt.Println("1")
	case 2:
		fmt.Println("2")
		fallthrough
	case 3:
		fmt.Println("3")
	}
```

### GO 字符

> GO 使用`utf8`编码，当我们使用`len(myStr)`的时候，返回的是字符串的字节数，而不是字符个数，需要反映字符个数用：`utf8.RuneCountInString(myStr)`
>
> 如果想要处理字符串，处理里面的单个字符，GO提供了字符串到字符slice的转换
>

```go
 strRunes := []rune(myStr) // 字符串转slice
 str := string(strRunes) // slice 转字符串
```

> 用for遍历字符串

```go
for position,c := range myStr{
}
```

### 错误处理 用户输入

> go的方法都会返回一个error，如果有错，这个值不为`Nil`，且提供了日志`log`让我们用来输出
>
> 注意的是这里的err不能定义为error，因为有系统类型error，如果手动定义了error会遮盖系统的

```go
package main

import (
 "fmt"
 "os"
 "bufio"
 "log"
)

/**
 * 用户输入和判断
*/
func main() {
 fmt.Print("Enter a grade: ")
 reader := bufio.NewReader(os.Stdin)
 // input,_ := reader.ReadString('\n') 使用blank identifier忽视错误
 input,err := reader.ReadString('\n')  // 接收错误
 if err!=nil{
  log.Fatal(err)
 }
 fmt.Println(input)
}

```

### 文件读取

```go
package main

import (
 "bufio"
 "fmt"
 "log"
 "os"
)

func main(){
 file,err := os.Open("./data")
 if err != nil{
  log.Fatal(err)
 }
 scanner := bufio.NewScanner(file)
 for scanner.Scan(){
  fmt.Println(scanner.Text())
 }
    err = file.Close()
    if err != nil{
  log.Fatal(err)
 }
    if scanner.Err() != nil{
  log.Fatal(scanner.Err())
 }
}

```

### 返回异常

> go的存在返回多个值的特性，所以一个方法如果执行参数有问题，可以返回error类型的参数，通过调用error包的New方法

```go
package main

import (
 "errors"
 "fmt"
 "log"
)

func main() {
 sayHi()
}

func sayHi() {
 fmt.Println("hi")
 fmt.Printf("%d\n", calc(-3, 5))
}
func calc(height int, width int) int {
 if height < 0 || width < 0 {
  errA := errors.New("can not be negative") // 原始用法，如果需要格式化参数用下面这种
  errB := fmt.Errorf("can not be negative,value is %d", (height * width))
  // 可以直接打印效果等同于：err.Error()
  log.Fatal(errA)
  msg := errB.Error()
  fmt.Println(msg)

 }
 return height * width
}

```

### 字符转数字

```go
func main() {
 fmt.Print("Enter a grade: ")
 reader := bufio.NewReader(os.Stdin)
 // input,_ := reader.ReadString('\n') 使用blank identifier忽视错误
 input, err := reader.ReadString('\n') // 接收错误
 if err != nil {
  log.Fatal(err)
 }
 fmt.Println(input)
 input = strings.TrimSpace(input)
 grade, err := strconv.ParseFloat(input, 64)
 if err != nil {
  log.Fatal(err)
 }

 if grade > 60 {
  fmt.Println("qualified")
 }else{
  fmt.Println("failed")
 }
}

```

> 这里我们看到err似乎被定义了两次，在go中，同一个变量名被定义两次是不可以的，但是如果简写形式`:=`下，定义多个变量，有至少一个变量是未曾出现过的，那么是合法的

### scope 域

> 需要注意的是go的域是通过{}来划分的，{}内部定义的变量无法被外部读取

![upload successful](/images/pasted-175.png)

### 循环

基本和其他语言一致

> continue 和 break 还是一样的用法

```go
for x:=0;x<10;x++ {
 fmt.Pintln(x)
}

// 也可以这样
x:=0
for x<10 {
 fmt.Pintln(x)
    x++
}
```

> 猜数游戏

```go
package main

import (
 "bufio"
 "fmt"
 "log"
 "math/rand"
 "os"
 "strconv"
 "strings"
 "time"
)

func main() {
 seconds := time.Now().Unix()
 rand.Seed(seconds) // 如果不给时间种子，每次随机结果一样
 target := rand.Intn(100) + 1
 fmt.Println("start gussing")
 fmt.Println(target)
 // 循环10次
 for x := 0; x < 10; x++ {
  fmt.Println("u have", 10-x, "times to guss")
  // 下一步 用户输入
  reader := bufio.NewReader(os.Stdin)
  input, err := reader.ReadString('\n')
  if err != nil {
   log.Fatal(err)
  }
  input = strings.TrimSpace(input)
  guss, err := strconv.Atoi(input)
  if err != nil {
   log.Fatal(err)
  }
  if guss == target {
   fmt.Println("right")
   return
  } else if guss < target {
   fmt.Println("lower")
  } else {
   fmt.Println("higher")
  }

 }
 fmt.Println("u failed")

}
```

### Printf,Sprintf以及占位符

和c的用法一样：`Printf("number: %0.2f\n,1.0、3.0)`

Sprintf 和 Printf一样，不同的是Sprintf打印，而是返回格式化的字符串

![upload successful](/images/pasted-176.png)

> %v 和 %#v 很有意思，后者不转义，代码里啥样，显示就是啥样

### 方法

> 和其他语言差不多，没啥好说，命名不能数字开头，大写字母开头表示暴露
>
> 方法内 类型和变量名是倒着的，方法返回值类型也在最后和scala一样
>
> func calc(width int) 这里是值拷贝
>
> 如果方法有返回类型，最后一行不是reture编译不通过

```go
import (
 "fmt"
)
func main() {
 sayHi()
}
func sayHi() {
 fmt.Println("hi")
}

```

#### 返回多个值

> 返回多个值需要在方法定义内加上`()`

```go
func getParam(number int) (int,string){
 return 1,"test"
}
```

> 甚至可以给每个返回值取名

```go
func getParam(number int) (vale int,name string){
 return 1,"test"
}
```

> 一般这个特性用来返回方法是否出错

```go
func calc(height int, width int) (int, error) {
 if height < 0 || width < 0 {
  errA := errors.New("can not be negative") // 原始用法，如果需要格式化参数用下面这种
  errB := fmt.Errorf("can not be negative,value is %d", (height * width))
  // 可以直接打印效果等同于：err.Error()
  //log.Fatal(errA)
  msg := errB.Error()
  fmt.Println(msg)
  // 也可以抛出
  return 0, errA

 }
 return height * width, nil
}
```

#### 可变函数

> `fmt.Println(1,2)`如同这个函数，有些函数可以接收若干个参数
>
> 想要函数可变接收参数如下

```go
func get(number int , numbers ...int){
 //numbers是个slice
}

// slice 无法直接传递进去，需要转换
mysli :=[]int{1,2,3,4}
get(1,mysli...)
```

### 指针

> 学过了scala后还是觉得指针不好，虽然很灵活单也很容易出问题，且go支持了多值返回，更不推荐指针了
>
> go 是值传递语言，所有传入的值都只拷贝，可以通过指针将变量的地址传入，这里还是值传递，只不过拷贝的是地址。
>
> 书里的解释很有趣，指针类型用`*`表示，go的定义和其他语言是反的，所以指针类型其他语言是`int *` ，go是`*int`表示`pointer to int`：指向int类型的指针
>
> 通过`fmt.Pintln(reflect.TypeOf(&num))`查看类型
>
> 特别注意的是：其他语言的指针，只能在自己的访问域内使用，举例，假如方法内生成了局部变量且将局部变量的指针返回，其他语言在方法执行结束后，指针指向的地址空间会被释放，但是go不会，只要有人用就不会释放

```go
package main

import (
 "fmt"
 "reflect"
)

func main() {
 // 查看地址
 var amount int = 19
 fmt.Println(&amount)
 // 查看类型
 fmt.Println(reflect.TypeOf(&amount))

 // 定义指针和使用
 var intPointer *int = &amount
 fmt.Println(intPointer, *intPointer)
 // 另一种形式
 floatVal := 1.1
 floatPointer := &floatVal
 fmt.Println(floatPointer)

 // 修改指针指向的值
 strVal := "hello"
 strPointer := &strVal
 *strPointer = "hello from tr"
 fmt.Println(strVal, strPointer)

 // 获取方法内的指针
 var receivePointer *float64 = genPointer()
 fmt.Println(receivePointer, *receivePointer)

 // 解决值传递的问题，传递的数据可被修改
 h := 10
 changeVal(&h)
 fmt.Println(h)
}

// 方法内的指针在方法结束后如果被引用，空间不会被释放
func genPointer() *float64 {
 val := 1.4
 return &val
}

// 修改传递的数据
func changeVal(val *int) {
 *val *= 2
}
```

### 包管理 package

> go定义了工作区，文件夹名字就叫`go`,安装了go后默认在home目录下会有go这个文件夹，这就是默认的工作区，包含`bin`,`pkg`,`src`
>
> 一般包名和包的文件夹名是一致的,但是`main`package是例外，main文件的package 必须是main，但是所在文件夹的名字随意，所有的文件引用都是去工作区引用的
>
> 如果运行失败无法找到自己写的模块，可能需要改下环境变量

```shell
go env -w GO111MODULE="off"
go env -w GOPATH=/home/tr/go
# 将main文件所在目录改为main
```

![upload successful](/images/pasted-177.png)

### 命名规则

1. 小写

2. 简写：fmt ==> format

3. 最好一个单词表达，如果超过一个单词用 `_`

4. 最好不常用，不然容易和本地变量命名冲突

### 常量 constants

1. const 定义符

2. 定义的时候必须赋值

3. 常量不可用 `:=` 简写 但是还是可以省略类型

4. 推荐程序内的所有“魔数”都用常量替换

5. 最好定义在包层，而不是方法内部

```go
const quantity = 10
```

### cmd传递参数

```go
func main(){
 fmt.Println(os.Args)
}
```

### 修改go的配置

#### 修改GOPATH

> go的工作目录永远在默认的home目录下，如果想要把工作区修改到其他目录，需要修改环境变量
>
> go tool永远会去GOPATH下寻找我们引用的模块，假如我们自己写的模块放入了`/code/src`目录下，如果想要go编译的时候去寻找，需要`export GOPATH="/code"`
>
> `go env -w GO111MODULE="off"` `go env -w GOPATH="/code"`  如果`export`不起作用，可以试试这样

#### 导入包

> 假如导入github上其他人的包，一般是将其他人的包以URI的形式，放入src下的目录，如下，导入的时候只需输入路径`"github.com/headfirstgo/keyboard"`

![upload successful](/images/pasted-178.png)

> 导入他人包就是cli里的，`go get <URL>`，查看文档：`go doc <URL>`

#### 添加包说明

> `go doc`可以查看包的信息，包给其他人用的时候需要添加一些说明
>
> 所有`Package`和方法语句之前的注释文档都会被`go doc`输出
>
> go可以自动生成在线文档服务，`godoc -http=:6060`

## GO数据结构

### 数组

> 基本和其他语言一样
>
> 不过初始化的时候，int型的默认0，string型的默认空

```go
// 基础类型数组
var notes [7]string
notes[0] = "test"

// 对象/结构体数组 默认值是utc 0的时间
var dates [3]time.Time

// 快速初始化数值
var names [3]string = [3]string{"1","2","3"}

// 简写快速初始化
names := [2]string {"1","2"}

// 如果要分多行，一定要用逗号作为格式化字符(最后一行也要逗号)
names := [2]string{
 "1",
    "2",
}

// fmt可以快速打印数据接口包括 array,map,slices等
fmt.Println(names)
fmt.Printf("%#v\n",names)
```

> 数组的循环

```go
// loop
for i := 0; i < len(names); i++ {
  fmt.Println(names[i])
}
```

> loop的另一种形式 和python的 `for key in`很像

```go
// loog another
for index, value := range names {
  fmt.Println(index, value)
}
```

> 以上这种情况，index如果没被用到，是会报错的，可以使用空定义避免

```go
for _, value := range names {
  fmt.Println(value)
}
```

### slices

>类似链表，用于解决数组长度固定问题
> 定义和数组一样，只不过不需要提供长度
> <b>本身是基于数组的，只是数组的一个限制镜像，若slice来自数组，修改数组就会导致slice变动</b>

```go
// 声明，因为slice基于数组，没有底层数组单声明slice无用，这时候的slice是个nil
var mycli []string
// 初始化一个数组给slice
mycli = make([]string,7)
// 使用
mycli[0] = "test"
```

> 可以快速简写，且遍历和数组一样

```go
mycli := make([]string,7)

// 快速定义赋值
mycli := []string{
  "1",
        "2",
}
```

> slice可以从现有数组中创建

```go
// 从数组的0到3截取创建slice，左闭右开
myarr := [5]int{1,2,3,4,5}
mysli := myarr[0:3]
// 且可以忽略开始或者结束
myslic := myarr[0:]
myslid := myarr[:3]
```

> 修改数组会导致slice变化

```go
myarr := [5]int{1,2,3,4,5}
mysli := myarr[0:3]
myarr[0] = 10
fmt.Println(mysli)
```

> 所以使用slice的时候，还是推荐make单独给slice一个数组
>
> slice 有append方法可以在尾部添加数据，前面说过slice是基于数组的，如果底层的数组空间不足，append会生成新的数组，将原来值拷贝过去，这时append给一个新的slice无法确定这两个slice是否共享一个底层数组，所以建议append给原来的slice

```go
// 安全的调用slice
mysli = append(mysli,1)
// 无法确定是否属于同一数组
mycli2 := append(mycli,1)
```

> 注意：虽然slice初始化的时候不确定数组是个nil，但是其他方法调用的时候不用判断是否为nil，默认为nill的slice是个空slice，比如以下就是合法的操作

```go
var sc []string
sc = append(sc,"test")
```

> 一个读取文件内的文本转为float型数组的例子：

```go
package main

import (
 "bufio"
 "fmt"
 "log"
 "os"
 "strconv"
 "strings"
)

func main() {
 arr, err := readFloatFromFile("./data")
 if err == nil {
  fmt.Println(arr)
 }
}

func readFloatFromFile(filePath string) ([]float64, error) {
 var numbers []float64
 file, err := os.Open(filePath)
 if err != nil {
  log.Fatal(err)
 }
 scanner := bufio.NewScanner(file)
 for scanner.Scan() {
  txt := strings.TrimSpace(scanner.Text())
  number, err := strconv.ParseFloat(txt, 64)
  if err != nil {
   // 出错返回数组和错误信息
   return nil, err
  }
  numbers = append(numbers, number)
 }
 err = file.Close()
 if err != nil {
  return nil, err
 }
 if scanner.Err() != nil {
  return nil, err
 }
 // 无错 返回数组和空
 return numbers, nil
}
```

> slice 传入可变函数需要添加`...`
>
> slice 排序

```go
sort.Strings(mySli)
```

### Map

> Map几乎可以用任意类型作为key（但是必须是可以使用`==`来比较的类型）
>
> Map定义：

```go
var myMap map[string]float64
myMap = make(map[string]float64)
```

> 快速定义：

```go
myMap := make(map[string]int)
```

> 使用：

```go
myMap["first"] = 100
```

> map字面量（快速赋值）：

```go
myMap := map[string]int{"a":1.2,"b":1.3}
```

> `map[something]` 如果从未定义过，也可以获取结果是0，想要区分一个key是否被赋值过如下：

```go
// map访问可以多带一个返回参数，是否接收都可以用于判断是否赋值过
// myMap["tzq"] # 这是直接访问
value,ok := myMap["tzq"]
// ok 为true表示赋值过
// 若只想知道是否赋值过
_,ok := myMap["tzq"]
```

> 删除key

```go
// 从myMap中删除key为name的值
delete(myMap,"name")
```

> map打印的时候是乱序打印的，可以将key塞到一个slice，然后排序slice打印

```go
package main

import (
 "fmt"
 "sort"
)

func main(){
 grades := map[string]float64{
  "tr":100,
  "tzq":100,
  "sxy":90,
 }
 var mySlic []string
 // 可以不写_
 for k := range grades{
  mySlic = append(mySlic,k)
 }
 // 不返回值
 sort.Strings(mySlic)
 for _,name := range mySlic{
  fmt.Printf("%5s || %5.2f\n",name,grades[name])
 }
}
```

> 一个使用map的例子，统计选票

```go
package main

import (
 "bufio"
 "fmt"
 "log"
 "os"
 "strings"
)

func main(){
 fmt.Println("reading")
 names,err := readStringFromFile("./names")
 if err != nil{
  log.Fatal(err)
 }
 // 统计名字出现次数
 res := map[string]int{}
 for _,name := range names{
  res[name]++
 }
 fmt.Println(res) 
}

func readStringFromFile(path string)([]string,error){
 res := []string{}
 file,err := os.Open(path)
 if err != nil{
  return nil,err
 }
 scanner := bufio.NewScanner(file)
 for scanner.Scan(){
  res = append(res,strings.TrimSpace(scanner.Text())) 
 }
 err = file.Close()
 if err != nil {
  return nil,err
 }
 if scanner.Err()!=nil{
  return nil,err
 }
 return res,nil 
}
```

### 结构体 structs（和C很像）

> 单独的结构体定义变量

```go
// 单独定义一个结构体（类似对象）
var person struct{
 name string
    age int
}
person.name = "tr"
fmt.Println(person.name)
```

> 自定义数据类型 配合结构体(类似实现java的类)

```go
type Person struct{
  name string
  age int
 }
 var tr Person
 tr.name = "te"
 tr.age = 24
 fmt.Println(tr)
```

> 结构体类型可以作为方法出入参，但是要记住go永远是值拷贝，传递的结构体，方法内修改不会影响方法外的结构体，所以建议所有对原始数据修改的方法，都用指针。且如果结构体很大，也用指针，不然拷贝很久消耗内存和更多cpu

```go
package main

import "fmt"

 type Person struct{
  name string
  age int
 }
func main(){
 //var person struct{
 // name string
 // age int
 //}
 //person.name = "tr"
 //person.age = 24
 //fmt.Println(person)
 var tr Person
 tr.name = "te"
 tr.age = 24
 tr = changeName(tr)
 fmt.Println(tr)
}

func changeName(person Person)Person{
 person.name = "changed"
 return person
}
```

> 指针访问结构体

```go
 var pointer *Person = &tr
 // 访问指针的内容
 fmt.Println((*pointer).name) // (*pointer)这样的写法很冗杂
 fmt.Println(pointer.name) // 简化形式
```

> go里面，只有大写开头的结构体才可以被其他包使用！<b>并且，结构体内的字段也要大写开头，才能访问！</b>

```go
Package MyStru
type Person struct{
  Name string
  Age int
}
```

```go
// 其他包内容调用
import MyStru

func main(){
 var p MyStru.Person 
}
```

> 这样通过`包名.类型`的方式有些啰嗦，go提供了结构体字面量可以同时创建和初始化字段

```go
// 不必所有的字段都要赋值
p := MyStru.Person{Name:"tr",Age:24}
```

> 结构体的嵌套

![upload successful](/images/pasted-179.png)

> 匿名变量（结构体链式调用简写），有时候如上图emp.Addr.City看起来很冗杂，go提供了匿名变量：只保留变量类型，去掉名字
> 访问的时候可以去掉名字作为内置变量访问，也可以通过类型名访问

![upload successful](/images/pasted-180.png)

## Go的高级特性

### 自定义类型

> 上一节讲了`type`，自定义类型基于结构体，现在可以试试基于任何其他类型并且定义方法
>
> 为什么要用这个，这里提供书上的一个场景：
> 一加仑=3.78升，如果一个程序定义了油耗是10:`var fuel float64 = 10`，那么他是10升还是10加仑，所以这时候我们可以将`float64`转为自定义类型，用可阅读的英语代替

```go
type Liter float64 // 定义升
type Gallon float64 // 定义加仑

func main{
 var carFuel Liter // 这里明显可以得知是升
    var busFuel Gallon // 同理
    carFuel = Liter(10) // 10升油
    
    motorCycle := Gallon(10)
    
    carFuel = Gallon(10) // 这里会编译错误，因为一开始声明的类型不一样，即使这些类型底层都是float64
}
```

> 但有时我们会想要混用转换这两种类型，go提供了两种自定义类型（基于同一种类型）互相转化的解决方案
>
> 定义自己的类型和对应操作符（+ - * / == > <），一般go是不允许不同类型加减乘除的，需要自定义操作符，这里我们希望`Liter`和`Gallon`单位混合运算
> PS:这里定义每个类型的对应方法的时候，方法指的就是方法(Function),函数(Method)指的是不属于类型的那些方法，比起函数，方法要多加一个入参(写在方法名左边)，其他和函数没有区别

```go
type MyType string

// 一旦MyType的方法定义了，任何MyType的value都可以调用
// 一般自定义的入参取名单个开头字母小写
func (m MyType) sayHi(){
}

value := MyType("some thing")
value.sayHi() // 这里value就是第一个入参
```

```go
package main

import "fmt"

type MyType string

// 如果不加sayHi左边的就是一个普通的方法而已
func (t MyType)sayHi(name string)string{
 fmt.Println(name,t)
 return name
}

func main(){

 // a := "test"
 // 这里是无法调出 a.sayHi的
 value := MyType("tr")
 value.sayHi("hello from ")
 // 结果是 hello from tr
}
```

> 哪些类型可以被自定义方法？ 在同一包下自定义的类型可以，int等系统类型不行
>
> 方法的第一入参可以接收指针，场景：需要给自己的基于int的类型加个Double倍增的方法，但是不使用指针无法修改，不过不用担心，对于首要入参，go会自动的在需要的时候转为指针，如下

```go
type Number int

// n如果不是指针类型 无法实现*2效果
func (n *Number) DoubleNumber(){
 (*n) *= 2;
}

func main(){
 val := Number(12)
 val.DoubleNumber() // 调用的时候go自动传入了地址
 fmt.Println(val)
}

```

> 这时候我们就可以解决第一个场景的问题了：给升和加仑转为对方的转换方法

```go
type Liter float64 // 定义升
func (l liter) toGallon()Gallon{
 return Gallon(l* 0.264)
}
```

### 封装和嵌入

#### Set

> 其实go的封装很类似java的一个实体类，对于go来说，一个struct内部的方法，如果可以不经过校验随意输入参数，那么代码很容易出错，需要给一个setter方法，让输入的参数经过校验
>
> 其实真的很像java的实体类，我们需要定义一个对象，对象再调用setter方法设置值，在go中我们得通过上节的自定义类型方法，例子如下

```go
// 定义日期结构体
type Date struct{
 Year int
 Month int
 Day int
}

// 定义类型方法 如果不用指针 结果不会改变
func (d *Date)SetYear(year int){
 d.Year = year
}
func main(){
 var d Date
 // 显然直接修改结构体内的值不安全，万一设置负的呢
 //d.Year = -1

 // 我们想要调用d.setYear 但是让类型调用方法我们需要自定义类型方法
 d.SetYear(1997)
 fmt.Println(d)

}
```

> 以上这个例子结构体内的域变量还是可以被访问到，其实这时候只要将结构体移到其他包内，且域内变量都小写不开放即可。这样我们就只能通过开放的`Setxxx`方法去修改不开放的`域内值`

#### Get

> Get方法实际上和java的作用也一样，建议类型首参用指针且方法名不用`GetYear`而是`Year`，如下

```go
func (d *Date)Year()int{
 return d.year
}
```

> 在其他语言中，封装是通过`class`实现的，而go是通过`package`实现的

#### 完整的GetSet例子

> date目录下

```go
package date

import "errors"

// 定义日期结构体,全都是小写为了防止被其他包读取
type Date struct {
 year  int
 month int
 day   int
}

// 定义Set方法
func (d *Date) SetYear(year int) error {
 if year < 0 {
  return errors.New("Year can not be negative")
 }
 d.year = year
 return nil
}

func (d *Date) SetMonth(month int) error {
 if month < 1 || month > 12 {
  return errors.New("Month must between 1-12")
 }
 d.month = month
 return nil
}

func (d *Date) SetDay(day int) error {
 if day > 31 || day < 1 {
  return errors.New("Day must between 1-31")
 }
 d.day = day
 return nil
}

// Get方法,最好还用指针不然拷贝数据浪费内存
func (d *Date) Year() int {
 return d.year
}
func (d *Date) Month() int {
 return d.month
}
func (d *Date) Day() int {
 return d.day
}
```

> 使用日期包的主文件

```go
package main

import (
 "date"
 "fmt"
)

func main(){

 var myDate date.Date
 myDate.SetYear(1997)
 myDate.SetMonth(11)
 myDate.SetDay(21)

 fmt.Printf("%v || %v || %v",myDate.Year(),myDate.Month(),myDate.Day())
}
```

#### 嵌入

> 一个结构体可以嵌套另一个结构体，但是访问的时候可以将子结构体的内容当做自己的属性访问（这些内容包括方法和属性）
>
> 不过需要外部结构体定义的子结构体是匿名的，即只有类型
>
> 当然子结构体内部不暴露的（小写开头）的变量是不会嵌入/或者说弹出给外部结构体的

### 接口 Interface

> go的接口指的永远是变量的接口，因为go没有类的概念，一个自定义类型可以基于任何类型，可以实现方法。那么接口就是约定这个自定义类型需要包含的方法

#### 接口实现和对应类型实现

> 和java的接口概念一样，约定我需要哪些方法，具体怎么来的不关心，不过go里面，接口必须和类型结合使用，类型可以有多个方法，但是必须有符合接口的方法才满足
>
> `You don’t care whether
you have a Pen or a Pencil, you just need something with a Draw method`
>
> 定义接口示例

```go
type MyInterface interface{
 sayHi()
    getSomething(float64)
    returnSomething()string
}
```

> 定义满足接口的类型

```go
type MyType int

//给这个类型定义方法
func (m *MyType) sayHi(){
  ...
}
func (m *MyType) getSomething(f float64){
  ...
}
func (m *MyType) returnSomething()string{
  ...
}
// 可以有其他方法
.......
```

> 这里定义了接口和类型，需要注意的是不同于`Java`的`implement`go是自动的，不需要手动声明，如下使用

```go
func main(){
  // 定义变量类型为接口
  var value MyInterface
  // 因为MyType定义的方法是符合接口的，所以这么做合法
  value = MyType(5)
  value.sayHi();
}
```

> 一个接口类型不确定内容到底是什么，可能是int，可能是其他，取决于自定义类型的基础

```go
package main
import "fmt"

type Whistle string
type Horn string

func (w Whistle) MakeSound(){
 fmt.Println("whistle")
}
func (h Horn) MakeSound(){
 fmt.Println("whistle")
}

type NoiseMaker interface{
 MakeSound()
}
func main(){
 var toy NoisMaker
    toy = Whistle("toy")
    toy.MakeSound()
    toy = Horn("toy")
    toy.MakeSound()
}
```

> 接口变量也可以作为入参，总结一下，go的接口实现方式是基于自定义类型的，一个自定义类型，包含了数据，和类型可以调用的方法，这就相当于Java的类
>
> 当生成了自定义类型的变量，就相当于java内的对象，可以作为参数传递，如果这个变量的方法满足接口，那么入参类型为接口变量即可
>
> 需要注意的是：如果实现类型方法的时候，首参是指针，那么在给一个接口变量赋值的时候，只能赋值类型变量的指针，下面是错误的示例和修正代码

![upload successful](/images/pasted-181.png)

![upload successful](/images/pasted-182.png)

#### 接口类型转为具体类型

> 如同java的接口一样，当使用接口的时候，那么只能使用接口定义的方法，假如实现接口的具体类有更多方法，需要转为具体实现类
>
> go也是同理，假如一个接口变量定义了并且赋值了一个实现接口的接口自定义类型，如果需要将接口变量转为自定义类型变量，只需要`.(具体自定义类型)`即可
>

```go
var noiseMaker NoiseMaker = Robot("test")
var robot Robot = noiseMaker.(Robot)
// 更推荐用下面这种带判断是否转换成功的
robot,ok := noiseMaker.(Robot)
if ok{
}else{}
```

#### Error接口

> 我们之前常用error类型，但是实际上error只是个接口，只要我们实现了`Error()string`的自定义类型都可以视为error的实现
>

```go
package main

import "fmt"

// 自定义类型
type ComputerError string

// 自定义类型实现Error()string 方法
func (c ComputerError)Error() string{
 return "a computerError occured"
}

func main(){
 // 我们自定义的ComputerError符合接口
 var err error = ComputerError("test")
 err.Error()
 fmt.Println(err)
}
```

> error接口是小写开头，那么我们为啥可以调用，毕竟小写开头都是私有不暴露的，这是因为error是作为`universe block`存在的，不论在哪个包都可以调用到。
>
> 有些接口是定义在方法下的，有些是在包下的，所以他们有限制，必须大写才能暴露，且使用需要引入，但是`error`和一些其他预定于的是在"所有包下的"

#### String接口

> 对于`fmt.Println`或者其他方法来说，这些print方法会去寻找入参的变量是否有`String()string`方法，如果有，会去调用这个方法
>
> 即，fmt存在`Stringer`接口，输入的任何变量其实是作为`Stringer`接口变量存在的:`fmt.Println(myVariable)`，`myVariable`会作为接口变量存在，当然如果不满足接口方法那就不是了
>
> 可以`go doc fmt Println`查看具体接口定义

```go
import "fmt"


// 自定义类型实现string方法满足Stringer接口
type CoffeePot string

func (c CoffeePot)String() string{
 return string(c) + "coffee"
}

func main(){
 coffee := CoffeePot("100")
 fmt.Println(coffee)
}
```

#### 空接口

> 一个接口定义了以后，只要任何变量满足这个接口内的方法，那么变量就可以被传入或者赋值
>
> 那么假如接口内无任何方法定义，则可以接收任何变量

```go
// 单独命名一个空接口
type emptInterface interface{}
// 任何值都可以被传入 {}表示接收的接口变量无方法
func AcceptThing(thing interface{}){
......
}
```

> 但是问题是空接口那么也没有方法可被调用，所以这时候要转换空接口为具体变量
> 了解这个再看fmt包的Println就明白了`func Println(a ...any) (n int, err error)` 这么定义表明可以接收若干个空接口即任何参数
> 随后转换这些参数

```go
func AcceptThing(thing interface{}){
  whistle,ok :=thing.(Whistle)
  if ok{
  }
}
```

### 代码例子

> 利用接口，自定义类型，结构体，封装等概念完成一个自定义链表，以及基于链表的栈
>
> 代码实现

```go
package main

import (
 "fmt"
 "list"
 "log"
)

// 测试main

func main(){
 fmt.Println("********")
 // 测试节点 节点元素可以是任何类型
 var n list.Node
 ele := list.StringEle("13.1")
 n.SetEle(ele) 
 n.SetEle(13)
 n.PrintList()

 // 测试链表
 fmt.Println("*************")
 // 生成的新节点不能用指针 除非malloc手动分配内存
 var newStartNode list.Node
 newStartNode.SetEle(11)
 var myList list.List
 myList = &newStartNode
 myList.AppendNode(22)
 myList.InsertNode(33,1)
 myList,_ = myList.DeleteNodeByIndex(1)
 err := myList.UpdateNodeByIndex(100,3)
 if err!=nil {
  log.Fatal(err)
 }
 myList.PrintList()
}
```

> 链表实现

```go
package list
import "fmt"

/**
 * @author tr
 * @content 实现链式链表
 * @date 2022-4-19
 */

//*************** 定义链式链表的基础节点***************
// 自定义错误类型(不能用指针！！！)
type nodeErr string
func (n nodeErr) Error()string{
 // 需要类型转换
 return fmt.Sprintf("node error %s:\n",string(n))
}
// 定义节点元素的基础类型
type IntEle int
type StringEle string
type FloatEle float64
type Node struct{
 // 节点元素，可以是任何类型
 ele interface{}
 // 节点的下一个指针 初始化的时候默认为nil
 next *Node
}
// 封闭私有元素，开放set,get
func (n *Node)SetEle(ele interface{}){
 n.ele = ele
}
func (n *Node)SetNext(next *Node){
 n.next = next
}
func (n *Node)Ele()interface{}{
 return n.ele
}
func (n *Node)Next()*Node{
 return n.next
}
//*****************************************************


//************************* 定义链式链表***************
// 定义一个链表的接口
type List interface{
 // 清楚元素
 Clear()
 // 链表长度
 GetLength()int
 // 插入节点
 InsertNode(interface{},int)(*Node,error)
 // 尾部插入节点
 AppendNode(interface{})
 // 删除节点-通过节点位置且返回头结点
 DeleteNodeByIndex(int)(*Node,error)
 // 删除节点-通过节点元素且返回被删除元素
 //DeleteNodeByEle(interface{})(interface{},error)
 // 更新节点-通过index
 UpdateNodeByIndex(interface{},int)error
 // 打印链表
 PrintList()
}

// 实现链表接口 如果实现的时候首参数用了 * 赋值要用 & 注意区别
func (n *Node)Clear(){
 n.next = nil
}

// 实现打印，因为元素类型不同,可以转为自定义的元素类型或者原始元素类型，通过转换分辨节点类型
func doPrint(ele interface{}){
 var ok bool
 _,ok = ele.(IntEle)
 if ok {
  fmt.Printf("int element %d\n",ele)
  return
 }
 _,ok = ele.(StringEle)
 if ok {
  fmt.Printf("string element %s\n",ele)
  return
 }
 _,ok = ele.(FloatEle)
 if ok {
  fmt.Printf("float element %f\n",ele)
  return
 }
 _,ok = ele.(int)
 if ok {
  fmt.Printf("original int element %d\n",ele)
  return
 }
 fmt.Println("unknown type",ele)
}
func (n *Node)PrintList(){
 for n!=nil{
  // fmt.Println(n.ele) 这样打印显示不出类型转换
  ele := n.Ele()
  doPrint(ele)
  n = n.Next()
 }
}
func (n *Node)GetLength()int{
 // 计数器
 counter := 0
 for n!=nil{
  counter++
  n = n.Next()
 }
 return counter
}

// 插入节点 从0开始计数，0表示在第一个，1表示在第一个节点后...
// 且只有为0的时候需要返回新的头部节点
func (n *Node)InsertNode(ele interface{},index int)(*Node,error){
 // 若大于链表长度 返回错误
 if index>n.GetLength(){
  return n,nodeErr("index out of bounds")
 }
 // 若等于 调用append
 if index == n.GetLength(){
  n.AppendNode(ele)
  return n,nil
 }
 // 若为0 成为头部
 if index == 0{
  var newHead Node
  newHead.SetEle(ele)
  newHead.SetNext(n)
  return &newHead,nil
 }
 // 其他情况下要做处理
 for ;index>1;index--{
  n = n.Next()
 }
 var newN Node
 newN.SetEle(ele)
 newN.SetNext(n.Next())
 n.SetNext(&newN) 

 return n,nil
}

func (n *Node)AppendNode(ele interface{} ){
 var newN Node
 newN.SetEle(ele)
 n.SetNext(&newN)
}

// 删除从1开始计数 如果删除第一个需要返回新的头结点
func(n *Node)DeleteNodeByIndex(index int)(*Node,error){
 if index>n.GetLength()||index<1{
  return nil,nodeErr("index out of bounds in deleting")
 }
 if index == 1{
  return n.Next(),nil
 }
 for ;index>2;index--{
  n = n.Next()
 }
 n.SetNext(n.Next().Next())
 return n,nil
}
//*****************************************************

func (n *Node)UpdateNodeByIndex(ele interface{},index int)error{
 if index>n.GetLength()||index<1{
  return nodeErr("index out of bounds in updating")
 }
 for ;index>1;index--{
  n = n.Next()
 }
 n.SetEle(ele) 
 return nil
}
```