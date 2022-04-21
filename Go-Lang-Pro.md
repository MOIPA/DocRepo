title: Go Lang Pro
author: tr
tags:
  - Go
categories: []
date: 2022-04-19 16:59:00
---
# GO Lang Pro

<!--more-->


# 延迟执行 defer (虽迟但到)

> 之前的代码处理错误都是简单的打印`log.Fatal()`，但是有些情况是必须要处理错误问题的，比如读取文件出错需要关闭文件流等
>
> 这时候就需要我们延迟方法返回执行完错误处理再返回
>
> go提供了`defer`关键字，在普通方法或者函数调用前加上go会推迟执行，但是必定执行，即使方法调用了return，被`defer`修饰的语句还是执行
>
> 如下代码，即使return了返回错误还是会执行defer修饰的语句，输出`GoodBye`
>
> 注意！ `defer`只能修饰方法或者函数调用

```go
func main(){
	Socialize()	
}

func Socialize()error{
	defer fmt.Println("GoodBye")
	fmt.Println("hello")

	return fmt.Errorf("I don't want to talk")
}

```

# 错误恢复以及处理

> 这里以递归为例
> 
> GO的递归没给什么特别的关键字或者语法糖，和其他语言一样用就行了

## 打印目录树

```go
func main(){
	PrintFireTree("/home/tr/go/src")
}

func PrintFireTree(dirPath string)error{
	files,err := ioutil.ReadDir(dirPath)
	if err != nil{
		log.Fatal(err)
	}
	for _,file := range files{
		if !file.IsDir(){
			fmt.Println("  "+file.Name())
		}else{
			// 可以直接用+拼接 也可以用path/filepath的join方法
			fmt.Println("printing dir:",dirPath+"/"+file.Name())
			err :=PrintFireTree(dirPath+"/"+file.Name())
			if err != nil{
				return err
			}
		}
	}
	return nil
}
```

## 错误处理和恢复

> 就如同上面那个递归一样，方法返回了error信息，每次调用都需要处理error信息，这样比较复杂，我们可以用更简洁的方式：`panic`和`recover`
>
> 手动生成一个panic错误 `panic("program going down")`
>
> 如同其他语言一样，go在出错的时候也提供了一个`stack trace`用于回溯出错的所有点
>
> `panic()` 只能用于程序bug的时候，不能因为用户输入错误数据终止程序
>
> `panic()`执行的时候会停止程序，输出错误栈，但是如果用户调用了`recover()`，那么`panic()`就只会打印错误消息
> 
> 要注意的是：一个方法内执行了`panic()`再执行`recover()`是无效的，因为recover只能在panic中执行，如果panic执行完了，那么轮不到revocer程序就终止了，所以要将`recover()`方法放入一个函数，使用`defer`修饰的函数，如例
>

```go
func main(){
	fmt.Println("running")
	Stop()
	fmt.Println("running")

}

func Stop(){
	// 如果发生了panic，执行恢复操作
	defer DoRec()
	fmt.Println("paniced")
	panic("stopping program")
	// panic发生后的所有方法都不再执行
	fmt.Println("after pannic")
}

func DoRec(){
	// fmt.Println(recover()) 可以打印panic的值
    recover()
}
```

> 这样可以改造我们之前的打印目录树的程序，将里面的error去掉，修改为panic，将error丢入panic：`panic(err)`，随后恢复和报告错误
>
> 也同样适用于其他任何程序

```go
func reportPanic(){
  // 若在panic时传入了一个error
  errInterface := recover()
  if errInterface == nil{
    return
  }
  err,ok = errInterface.(error)
  if ok{
  	fmt.Println(err)
  }
}
```

## recover的问题

> recover 会从panic状态中恢复过来，但是recover会恢复任何panic，这就会导致问题
>
> recover会返回panic的值，那么我们可以判断这个值是不是一个error或者其他我们定义的，如果决定不恢复可以增加一个`panic()`
>

```go
func reportPanic(){
  // 若在panic时传入了一个error
  errInterface := recover()
  if errInterface == nil{
    return
  }
  err,ok = errInterface.(error)
  if ok{
  	fmt.Println(err)
  }else {
    // 说明传递的不是error类型，不处理就继续panic吧
    panic(errInterface)
  }
}
```
 
# GO 并发执行

> GO 并发执行，或许在web应用中最为常见，所以书上举例了web，这里同样
>

## web

> go的 `net/http` 库提供了http请求用于发送和请求http报文数据
>
> `http.Get`：发送Get请求
>
> `http.Response`：结构体响应
>
> 以下是一个请求多次的程序，顺序执行

```go
import (
	"fmt"
	"io/ioutil"
	"log"
	"net/http"
)

func main(){
	ResponseSize("https://example.com")
	ResponseSize("https://example.com")
	ResponseSize("https://example.com")
	ResponseSize("https://example.com")
}
func ResponseSize(url string){
	response,err := http.Get(url)
	if err != nil{
		log.Fatal(err)
	}
	// 最后一定要关闭网络连接
	defer response.Body.Close()
	// 读取响应数据
	body,err := ioutil.ReadAll(response.Body)
	if err != nil{
		log.Fatal(err)
	}
	// fmt.Println(body) body是一个slice存储了十进制数，需要手动转为其他类型
	// fmt.Println(string(body)) // 这样就输出了一个html文档
	fmt.Println(len(body))
}
```

## 并发 goroutines

> 上面的代码顺序请求，go提供了并发执行的方式，其实类似java的线程
>
> 只需要使用`go`：`go myFunction()` 只能用于方法调用，作用相当于开启了一个线程
>
> go认为主方法就是一个`goroutine`，当主方法的`goroutine`结束，进程就退出了，所以即使有其他的`go routine`在执行，一旦main的结束了就会退出
>
> 所以要等待其他`goroutine`执行完毕：`channel`
>
> 以下代码是修改后的结果，这里暂时不用channel

```go
func main(){
	go ResponseSize("https://example.com")
	go ResponseSize("https://example.com")
	go ResponseSize("https://example.com")
	go ResponseSize("https://example.com")
	time.Sleep(2*time.Second)
}
func ResponseSize(url string){
	response,err := http.Get(url)
	if err != nil{
		log.Fatal(err)
	}
	defer response.Body.Close()
	body,err := ioutil.ReadAll(response.Body)
	if err != nil{
		log.Fatal(err)
	}
	fmt.Println(len(body))
}
```

> 这并行执行的时候并不保证谁是优先的，需要用channel控制
>
> `go <func>`不可以配合`return`

## channel

> 可以用于`goruntine`的控制，传递参数
>

```go
var myChannel chan float64
myChannel = make(chan float64)

// 简写
myChannel := make(chan float64)
```

> 赋值：`myChannel <- 3.14`
>
> 取值：`<-myChannel`
>
> 取值和赋值是同步的一个操作，一次赋值对应一次取值，如果没有一次赋值，执行取值会阻塞直到下次赋值,简单的例子

```go
func greeting(myChannel chan string){
  myChannel <- "hi"
}
func main(){
  myChannel := make(chan string)
  go greeting(myChannel)
  fmt.Println(<-myChannel)
}
```

> 利用channel 完成进程间的同步：当一个线程`X`生成了一个`channel`，将他传递给其他线程时，其他线程每次执行赋值操作，都需要`X`线程运行才能继续执行，如果`X`此时阻塞，被传递的线程也陷入阻塞态
>
> 使用channel 同步的例子

```go
import (
	"fmt"
	"io/ioutil"
	"log"
	"net/http"
)

func main(){
	size := make(chan Page)
	go ResponseSize("https://example.com",size)
	go ResponseSize("https://baidu.com",size)

	// 主线程直接执行到这里，但是会在第一个 <-size 这里阻塞,直到size被执行了一次赋值
	fmt.Println(<-size)
	fmt.Println(<-size)
}
// channel 可以接收struct类型
type Page struct{
	Url string
	Size int
}
func ResponseSize(url string,channel chan Page){
	response,err := http.Get(url)
	if err != nil{
		log.Fatal(err)
	}
	defer response.Body.Close()
	body,err := ioutil.ReadAll(response.Body)
	if err != nil{
		log.Fatal(err)
	}
	p := Page{Url:url,Size:len(body)}
	channel <- p
}
```

## 生产者消费者

> 目前所有的channel都是`unbuffered`类型，即子线程send数据给channel后立即阻塞，直到这个channel的内容被取出
>
> `buffered channel`意思是，可以先往channel放一部分数据
> `channel := make(chan string,5) // 这个channel可以放五个数据`且缓存的数据实际是队列模式，其他线程取数据每次都取最早放入的数据，先进先出
>
> 这时候`channel <- "a"` 不会阻塞子线程，直到执行了5次后channel满了才阻塞，这样就可以用来编写一个生产者消费者
>
> 同时为了防止主线程直接结束，主线程需要等子线程传递来消息才结束

```go
package main

import (
	"fmt"
	"time"
)

func main() {
	fmt.Println("start test")
	channel := make(chan string, 3)
	// 表示两个子线程都结束的chan，每个线程结束的时候往里面写一个数据
	endChan := make(chan string, 2)
	go producer(channel, endChan)
	go consumer(channel, endChan)
	fmt.Println(<-endChan, <-endChan)
}

func producer(channel chan string, endChan chan string) {
	for i := 0; i < 10; i++ {
		time.Sleep(time.Second * 2)
		fmt.Println("producer made product")
		channel <- fmt.Sprint("product", i)
	}
	// 线程结束，给主线程一个消息 因为endChan容量是2所以两个线程都写了主线程才能读取
	endChan <- "producer end"
}

func consumer(channel chan string, endChan chan string) {
	for i := 0; i < 10; i++ {
		time.Sleep(time.Second * 6)
		product := <-channel
		fmt.Println(product, "consumed")
	}
	endChan <- "consumer end"
}
```

# GO 的自动化测试

> 在给程序新增功能后，需要测试老的以往的功能是否正常，可以使用go提供的自动测试工具
>
> 假设有以下代码，用于英语环境下的连续物件`... , ... and ..`

```go
import (
	"fmt"
	"strings"
)

func main(){
	phrases := []string{"apple","orange","pie"}
	fmt.Println(JoinWithCommas(phrases))
}

func JoinWithCommas(phrases []string)string{
	result := strings.Join(phrases[:len(phrases)-1],", ")
	result += " and " + phrases[len(phrases)-1]
	return result
}
```

> 在新增功能之前，我们可以开始编写自动测试了，提供一组输入和输出，如果代码的结果不匹配则`fail`
>
> 自动文件的命名规则，如果主文件是`join.go`那么测试文件：`join_test.go`
>
> 测试文件不一定要和被测文件一个包，但要是想测的东西是私有的，那么只能在一个包下
>
> 测试方法名必须`Test`开头
>
> 执行测试文件：`go test ...` 被测文件必须包含`_test.go`否则无法找到
```go
import (
	"testing"
	"fmt"
)

// 下面这种是推荐的模板
func TestTwoElements(t *testing.T){
	// 失败
	//t.Error("no test yet")
	list := []string{"apple","orange"}
	want := "apple and orange"
	got := JoinWithCommas(list)
	if got != want {
		t.Error(errorString(list,got,want))
	}
}
func TestMoreElements(t *testing.T){
	list := []string{"apple","orange","pear"}
	want := "apple, orange, and pear"
	got := JoinWithCommas(list)
	if got != want {
		t.Error(errorString(list,got,want))
	}
}

// 因为不是Test开头的，所以不会执行测试
func errorString(list []string,got string,want string)string{
	return fmt.Sprintf("JoinWithCommas(%#v) = \"%s\", want \"%s\"",list,got,want)
}
```

> 这是文件结构 `go test join` 自动去`~/go/src`下面找`join`包，执行里面的`Test`开头的方法

![upload successful](/images/pasted-183.png)

> 默认的`go test <...>`会测试全部，可以添加配置测试部分
>
> `go test <...> -v` : 查看测试详情
>
> `go test <...> -run Two`：测试名字里面带`Two`的方法

## 表格驱动测试

> 测试的很多代码是重复的，我们可以生成一个表格，输入数据和期望数据的表格

```go
import (
	"testing"
	"fmt"
)

type testData struct{
	list []string
	want string
}

// 下面这种是推荐的模板
func TestJoinWithCommas(t *testing.T){
	testList := []testData{
		{list:[]string{"apple"},want:" and apple"},
		{list:[]string{"apple","orange"},want:"apple and orange"},
		{list:[]string{"apple","orange","pear"},want:"apple, orange, and pear"},
	}
	for _,test := range testList{
		got := JoinWithCommas(test.list)
		if got != test.want {
			t.Error(errorString(test.list,got,test.want))
		}
	}
}
// 因为不是Test开头的，所以不会执行测试
func errorString(list []string,got string,want string)string{
	return fmt.Sprintf("JoinWithCommas(%#v) = \"%s\", want \"%s\"",list,got,want)
}
```

# GO 函数传递

> GO 支持`first-class`函数，可以用于函数间传递，所谓`first-class`函数，意思是go的函数是可以被赋值给变量，由变量调用

```go
func main() {

	h := sayHi
	h()
}

func sayHi() {
	fmt.Println("hi from tr")
}
```

>
> 定义函数接口 传递函数作为参数，函数作为参数的时候不用预先声明

```go
func main() {
	double(sayHi)
}
func sayHi() {
	fmt.Println("hi from tr")
}
func double(hi func()) {
	hi()
	hi()
}
```

> 函数作为参数传递的时候，作为参数的这个函数格式必须符合定义的入参的函数的格式

```go
func main() {
	var s1 func() string
	var s2 func(string)

	s1 = sayHello // 换过来赋值就是错的
	s2 = sayHi
	s1()
	s2("tr")
}

func sayHi(name string) {}

func sayHello() string { return "hello" }
```

# GO 的web（请求响应）

> 这章节其实主要讲的是`net/http`包提供的响应`http`请求的内容
>
> 先简要概述下服务器软件的作用，首先客户端通过url和端口的方式访问物理机，物理机接收tcp或者udp报文（内部是http报文），将报文拆解后交给对应端口的服务器程序，程序收到http报文，可以看到报文头的访问路径`/hello`，找到对应的处理方法处理请求后封装http报文交给网络的下一次发出
>
> 这章能感受到作为后端服务，go比Java强的地方，Java相比太过笨重，从JavaEE到Spring生态，开发web是比较重的任务，go则更轻量快速，运行效率更高
>
> 先从一个简单的web应用开始

## simple web app

```go
package main

import (
	"log"
	"net/http"
)

// url 对应的处理器 类似java的controller
func viewHandler(writer http.ResponseWriter,request *http.Request){
	message :=[]byte("Hello from web powered by go!")
	_,err := writer.Write(message)
	if err != nil{
		log.Fatal(err)
	}
}

func main(){
	http.HandleFunc("/hello",viewHandler)
    // nil 是因为已经调用HandleFunc，给了一个处理方法了
	err := http.ListenAndServe("localhost:8080",nil)
	log.Fatal(err)
}
```

## 结合html页面

> 用Go来返回html页面，我觉得还是前后端分离更好，单纯用Go的高效性能负责后端请求处理
>
> Go提供了template模板，读取html文件内容，插入我们的动态内容返回给客户端
>
> 模板内需要插入的数据用`{{}}`来标识，称为一个`action`
>
> `{{.}}` 插入任何数据都会显示
>
> `{{if .}} 内容 {{end}}` 只有传递的数据为true才显示内容
>
> `{{range .}} 内容 {{.}} 其他内容 {{end}}` 传递一个slice，会遍历slice显示这段模板内容，将单个数据插入
>
> `{{.Name}}` 传递一个结构体，显示结构体的Name属性
>
> 注意：引入的包需要是 `html/template` 而不是`text/template`后者会显示任何东西即使是一段js代码
>
> guestbook.go

```go
package main

import (
	"fmt"
	"html/template"
	"log"
	"net/http"
	"os"
)

func main() {
	http.HandleFunc("/guestbook", ViewHandler)
	http.HandleFunc("/guestbook/new", NewHandler)
	http.HandleFunc("/guestbook/add", AddHandler)
	err := http.ListenAndServe("0.0.0.0:8080", nil)
	Check(err)
}

type Guestbook struct {
	SignaureCount int
	Signatures    []string
}

func ViewHandler(writer http.ResponseWriter, request *http.Request) {
	// 获取本地签名文件内容
	signatures := GetStrings("signatures.txt")
	// 将签名放入结构体
	guestbook := Guestbook{
		SignaureCount: len(signatures),
		Signatures:    signatures,
	}

	html, err := template.ParseFiles("view.html")
	Check(err)
	// html.Execute查看doc会发现接收的第一个参数是一个接口，之后会调用接口的write方法
	// 第二个参数是插入模板的数据 这里放入我们的签名结构体
	err = html.Execute(writer, guestbook)
	Check(err)
}

func NewHandler(writer http.ResponseWriter, req *http.Request) {
	htmlTemplate, err := template.ParseFiles("new.html")
	Check(err)
	// 单单返回new.html 不给页面插入参数
	err = htmlTemplate.Execute(writer, nil)
	Check(err)
}
func AddHandler(writer http.ResponseWriter, request *http.Request) {
	// 从request获取post数据
	signature := request.FormValue("signature")
	// 保存到本地文件 这个表示以只写，追加模式打开文件，若无则创建
	// go doc os O_WRONLY 查看具体信息
	options := os.O_WRONLY | os.O_APPEND | os.O_CREATE
	// 创建文件的时候给个权限
	file, err := os.OpenFile("signatures.txt", options, os.FileMode(0600))
	Check(err)
	// write data to file
	_, err = fmt.Fprintln(file, signature)
	Check(err)
	err = file.Close()
	Check(err)
	// 重定向到展示页面 req,resp 参数，重定向的地址，返回的状态码(302 找到)
	http.Redirect(writer, request, "/guestbook", http.StatusFound)
}

func Check(err error) {
	if err != nil {
		log.Fatal(err)
	}
}

func GetStrings(fileName string) []string {
	var lines []string
	file, err := os.Open(fileName)
	if os.IsNotExist(err) {
		return nil
	}
	Check(err)
	defer file.Close()
	scanner := bufio.NewScanner(file)
	for scanner.Scan() {
		lines = append(lines, scanner.Text())
	}
	Check(scanner.Err())
	return lines
}
```
> 两个html页面，一个展示，一个带个form用于新增
>
> view.html

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Document</title>
</head>
<body>
    <h1>Guest Book</h1>
    <div>
        {{.SignaureCount}} total signatures
        <a href="/guestbook/new">Add ur new signature</a>
    </div>
    <div>
        {{range .Signatures}}
        <p>{{.}}</p>
        {{end}}
    </div>
</body>
</html>
```

> new.html

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Document</title>
</head>
<body>
    <h1>add signature</h1>
    <form action="/guestbook/add" method="post">
        <div><input type="text" name="signature" id="" placeholder="signature"></div>
        <div><input type="submit" value="submit"></div>
    </form>
</body>
</html>
```

> 数据就存储在本地的文件内：`signatures.txt`

```txt
tr
oth
```