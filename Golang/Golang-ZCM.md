#喵
##基础知识
字符串

因为Golan中的字符串是不可变的，所以不能像其他语言那样很容易就修改字符串的内容。但是还是有至少下面两种方式来实现字符串内容的修改。

第一种：转成 []byte类型
<pre>
package main

func main() {
/*Go中字符串是不可变的,所以var s string = "hello" s[0] = 'c' println(s) 报错
*/
	var s string = "hello"
	c := []byte(s)  //将字符串 s 转换成 []byte 类型
	c[0] = 'c'
	s = string(c)  //再转回 string 类型
	println(s)
}
第二种：切片操作
<pre>
package main

import "fmt"

func main() {
	s := "hello"
	s = "c" + s[1:] //切片操作 加  字符串连接
	fmt.Println(s)
}
output==>
cello
</pre>
output==>
cello	
</pre>
数组  -- 值类型
<pre>
package main

import "fmt"

//数组
func main() {
	var a [10]int
	a = [10]int{2, 3, 4, 5}
	b := [5]string{"f", "d", "e"}
	c := [...]int{45, 56, 67, 78, 8, 89, 8900, 8} //不定长度
	fmt.Println(c)
	fmt.Println(b)
	fmt.Println(a)
}
output==>
[45 56 67 78 8 89 8900 8]
[f d e  ]
[2 3 4 5 0 0 0 0 0 0]
</pre>
切片 -- 引用类型

在很多应用场景中，数组并不能满足我们的需求。在初始定义数组时，我们并不知道需要多大的数组，因此我们就需要“动态数组”。在Go里面这种数据结构叫slice
<pre>
package main

import "fmt"

//数组
func main() {
	var ar = []byte{'a', 'b', 'c', 'd', 'e', 'f', 'g', 'h'}
	var a, b []byte
	a = ar[:5]
	b = ar[4:7]
	fmt.Println(string(a))
	fmt.Println(b)
}
output==>
abcde
[101 102 103]
</pre>
map 
<pre>
package main

import "fmt"

//数组
func main() {
	var numbers map[string]int = make(map[string]int)
	numbers["w"] = 3
	numbers["s"] = 2
	fmt.Println(numbers["w"])
}
ouput==>
3
</pre>
map 无序的，可能每次打印的map不是相同顺序的；通过 delete 删除 map 元素：
<pre>
package main 

import "fmt"

func main(){
	var a map[int]int = make(map[int]int)
	a[4] = 3
	a[2] = 1
	a[5] = 4
	a[6] = 5
	delete(a,6)   //删除map元素
	fmt.Println(a)
}
两次output==>
[ `go run main.go` | done: 2.5041432s ]
	map[5:4 4:3 2:1]
[ `go run main.go` | done: 2.2391281s ]
	map[4:3 2:1 5:4]        /*说明map是无序的*/
</pre>
<pre>
package main

import "fmt"

func main() {
	rating := map[string]float32{"c": 3, "d": 4, "f": 7, "h": 8}
	//map有两个返回值，分别是value与key，不存在key则是false
	resu, ok := rating["c"]
	if ok {
		fmt.Println(resu) //输出 value 值
	} else {
		fmt.Println("something error")
	}

}
output==>
3
</pre>
<pre>
package main

import "fmt"

//证明map是引用类型
func main() {
	m := make(map[string]string)
	m["hello"] = "Bonjour"
	m1 := m
	m1["hello"] = "salut"
	fmt.Println("m[\"hello\"]", m["hello"])
	fmt.Println("m1[\"hello\"]", m1["hello"])
}
output==>
m["hello"] salut
m1["hello"] salut
</pre>
需要注意的是：map和其他基本型别不同，它不是thread-safe，在多个go-routine存取时，必须使用mutex lock机制。

传值与传指针

先看一下下面两个例子，传值：
<pre>
package main

import "fmt"

func add1(a int) int {
	a = a + 1
	return a
}
func main() {
	x := 4
	fmt.Println("x =", x)

	x1 := add1(x)

	fmt.Println("x+1 =", x1)
	fmt.Println("x =", x)
}
output==>
x = 4
x+1 = 5
x = 4			
</pre>
看到了吗？虽然我们调用了add1函数，并且在add1中执行a = a+1操作，但是上面例子中x变量的值没有发生变化

理由很简单：因为当我们调用add1的时候，add1接收的参数其实是x的copy，而不是x本身。

那你也许会问了，如果真的需要传这个x本身,该怎么办呢？

传指针:
<pre>
package main

import "fmt"

func add1(a *int) int {
	*a = *a + 1
	return *a
}
func main() {
	x := 4
	fmt.Println("x =", x)

	x1 := add1(&x)

	fmt.Println("x+1 =", x1)
	fmt.Println("x =", x)
}
output==>
x = 4
x+1 = 5
x = 5
</pre>
这样，我们就达到了修改x的目的。那么到底传指针有什么好处呢？

- 传指针使得多个函数能操作同一个对象。
- 传指针比较轻量级 (8bytes),只是传内存地址，我们可以用指针传递体积大的结构体。如果用参数值传递的话, 在每次copy上面就会花费相对较多的系统开销（内存和时间）。所以当你要传递大的结构体的时候，用指针是一个明智的选择。
- Go语言中channel，slice，map这三种类型的实现机制类似指针，所以可以直接传递，而不用取地址后传递指针。（注：若函数需改变slice的长度，则仍需要取地址传递指针）

defer 

1.在defer后指定的函数会在函数退出前调用
<pre>
package main
//2.后进先出
import "fmt"

func main() {

	for i := 0; i < 5; i++ {
		defer fmt.Println(i)
	}
}
output==>
4
3
2
1
0
</pre>
函数作为值、类型
<pre>
package main

import "fmt"

type testInt func(int) bool

func isOdd(integer int) bool {
	if integer%2 == 0 {
		return false
	}
	return true
}

func isEven(integer int) bool {
	if integer%2 == 0 {
		return true
	}
	return false
}
func filter(slice []int, f testInt) []int {
	var result []int
	for _, value := range slice {
		if f(value) {
			result = append(result, value)
		}
	}
	return result
}

func main() {
	slice := []int{1, 2, 3, 4, 5, 6, 7}
	fmt.Println("slice = ", slice)
	odd := filter(slice, isOdd)
	fmt.Println("Odd elements of slice are :", odd)
	even := filter(slice, isEven)
	fmt.Println("Even elements of slice are :", even)
}
output==>
slice =  [1 2 3 4 5 6 7]
Odd elements of slice are : [1 3 5 7]
Even elements of slice are : [2 4 6]
</pre>
###常量 const iota 
const可以放到func外面，其他变量的声明不可以放到外面。
<pre>
package main

import "fmt"
import "os"

const z string = "这是string"

//常量组合声明时，iota每次引用会逐步自增，初始值为0，步进值为1
const (
	a uint8  = iota
	b uint8  = iota
	c uint16 = iota
)
//即使iota不是在常量组内第一个开始引用，也会按组内常量数量递增
const (
	a1        = 4
	a2 string = "d"
	a3 bool   = true
	a4 int    = iota
)
//枚举的常量都为同一类型时，可以使用简单序列格式(组内复用表达式).
const (
	x = iota
	x1
	x2
)

//定制iota序列初始值与步进值
const (
	z1 = (iota + 2) * 3 //初始值 6 ，步进值 3
	z2
	z3
)

func main() {
	var i int
	i = 4
	j := "hello"
	t := "你好"
	const x int = 4
	fmt.Println(i, j, t, "ok")
	fmt.Println(x, z, "c iota:", c, "a4 iota:", a4) //c等于2 a4等于3
	fmt.Println("x2 iota:", x2)                     //x2等于2
	fmt.Println("z1 value :", z1, "z2 value:", z2, "z3 value:", z3)
	os.Exit(0)
}
output==>
4 hello 你好 ok
4 这是string c iota: 2 a4 iota: 3
x2 iota: 2
z1 value : 6 z2 value: 9 z3 value: 12
</pre>
###数组 Array
<pre>
package main

import "fmt"

func main() {
	var a [3]int = [3]int{3, 4, 5}
	var b [2]int = [2]int{} //[0 0]
	//使用...自动计算数组的长度
	var c = [...]int{5, 6, 7, 8, 9} //[5 6 7 8 9]
	d := [6]int{}                   //[0 0 0 0 0 0]
	//多维数组.多维数组只能自动计算最外围数组长度
	e := [...][3]int{{1, 2, 5}, {4, 6, 4}} //[[1 2 5] [4 6 4]]
	//初始化指定索引的数组元素，未指定初始化的元素保持默认零值
	var f = [...]string{2: "first", 4: "second"}
	fmt.Println(a[2])
	fmt.Println(b)
	fmt.Println(c)
	fmt.Println(d)
	//通过下标访问多维数组元素
	fmt.Println(e, "++++++++", e[1][2]) //[[1 2 5] [4 6 4]] ++++++++ 4
	fmt.Println("f数组所有：", f, "f数组元素：", f[2])
}
output==>
5
[0 0]
[5 6 7 8 9]
[0 0 0 0 0 0]
[[1 2 5] [4 6 4]] ++++++++ 4
f数组所有： [  first  second] f数组元素： first
</pre>
###Slice
<pre>
package main

import "fmt"

func main() {
	//slice
	var a []int
	fmt.Println(a, "len:", len(a), "cap:", cap(a)) //	[] len: 0 cap: 0
	var b []int = []int{5, 78, 5, 56, 45, 3}
	fmt.Println(b)         //[5 78 5 56 45 3]
	for _, pp := range b { //5,78,5,56,45,3,
		fmt.Printf("%d,", pp)
		if len(b) == 6 {
			fmt.Println()
		}
	}
	//使用内置函数make初始化slice，第一参数是slice类型，第二参数是长度，第三参数是容量(省略时与长度相同)
	var c = make([]int, 3, 10)
	fmt.Println(c, "len:", len(c), "cap:", cap(c))
	var d = new([]int)                               //var d= new([]int ,2,3)写法错误
	fmt.Println(d, "len:", len(*d), "cap:", cap(*d)) //这里用地址的形式访问
	e := []int{4, 5, 7, 8, 54}
	fmt.Println("e:", e, "len:", len(e), "cap:", cap(e))
	e1 := e[0:2] //值是e[0] e[1]，不包括e[2].  [4 5]
	fmt.Println("e1:", e1, "len:", len(e1), "cap:", cap(e1))
	e2 := e[:3] //值是e[0] e[1] e[2] ,不包括e[3]
	fmt.Println(e2)
	e3 := e[:] //相当于复制了一个e切片
	fmt.Println("e3:", e3, "len:", len(e3), "cap:", cap(e3))
	//向slice中增加/修改元素
	f := []string{} //空的slice
	f = append(f, "Jason")
	f = append(f, "Miao")
	fmt.Println(f, "len:", len(f), "cap:", cap(f))
	/*删除slice中指定的元素,因为slice引用指向底层数组，数组的长度不变元素是不能删除的，
	所以删除的原理就是排除待删除元素后用其他元素重新构造一个数组*/
	index := 2 //删除第三个元素
	var ee []int
	ee = append(e[:index], e[index+1:]...)
	fmt.Println(ee) //[4 5 8 54]
	//向slice中间插入元素 注意：保存后部剩余元素，必须新建一个临时切片
	rear := append([]int{}, ee[index:]...)
	ee = append(ee[0:index], 100)
	ee = append(ee, rear...)
	fmt.Println("after insert:", ee)

}
output==>
[] len: 0 cap: 0
[5 78 5 56 45 3]
5,
78,
5,
56,
45,
3,
[0 0 0] len: 3 cap: 10
&[] len: 0 cap: 0
e: [4 5 7 8 54] len: 5 cap: 5
e1: [4 5] len: 2 cap: 5
[4 5 7]
e3: [4 5 7 8 54] len: 5 cap: 5
[Jason Miao] len: 2 cap: 2
[4 5 8 54]
after insert: [4 5 100 8 54]
</pre>
###字典/映射 Map
<pre>
package main

import "fmt"

func main() {
	/*map是引用类型，使用内置函数 make进行初始化，
	未初始化的map零值为 nil长度为0，并且不能赋值元素
	var m map[int]int
	fmt.Println(len(m))   -->false can`t enter value to a nil map
	*/
	//使用内置函数make初始化map
	var m map[int]int = make(map[int]int)
	m[0] = 4
	m[3] = 2
	m[4] = 1
	m[7] = 2
	fmt.Println(m)
	fmt.Println("is nil:", nil == m) //false
	//直接赋值初始化map
	n1 := map[string]int{
		"jason": 3,
		"miao":  2, //最后的逗号一定要加上
	}
	type S struct {
		age  int
		name int
	}
	n2 := map[string]S{
		"a": S{3, 5},
		"b": {23, 7}, //最后的逗号一定要加上;类型名称可忽略
	}
	fmt.Println(n1, n2)
	//map的使用:修改、删除元素
	fmt.Println(n2["b"]) //{23 7}
	n2["b"] = S{77, 77}  //修改
	fmt.Println(n2["b"])
	delete(n2, "b")      //删除
	fmt.Println(n2["b"]) //空的map是{0,0}   {0 0}
}
output==>
map[4:1 7:2 0:4 3:2]
is nil: false
map[jason:3 miao:2] map[b:{23 7} a:{3 5}]
{23 7}
{77 77}
{0 0}
</pre>
###结构体Struct
<pre>
package main

import "fmt"

func main() {
	type S struct {
		a int
		b string
	}
	//结构体初始化通过结构体字段的值作为列表来新分配一个结构体
	var s S = S{4, "jason"}
	fmt.Println(s)   //{4 jason}
	fmt.Println(s.a) //4
	//结构体是值类型，传递时会复制值，其默认零值不是nil
	var a S
	var b = S{}
	fmt.Println(a)      //{0 }
	fmt.Println(b)      //{0 }
	fmt.Println(a == b) //true
	type People struct {
		name  string
		age   int
		phone int
	}
	var jason People = People{"jason", 12, 123434545}
	fmt.Println("jason`s phone :", jason.phone)
	fmt.Println("jason`s name :", jason.name)
	//匿名结构体
	//匿名结构体声明时省略了type关键字，并且没有名称
	var x struct{}
	var y struct{ x int }
	fmt.Println(x, y) //{} {0}
	y.x = 3
	fmt.Println(y.x) //3
	
}
output==>
{4 jason}
4
{0 }
{0 }
true
jason`s phone : 123434545
jason`s name : jason
{} {0}
3
</pre>
###指针pointer
<pre>
package main

import "fmt"

func main() {
	var i int = 1
	pi := &i
	fmt.Println(pi) //0xc082006288
	a := []int{4, 5, 6}
	pa := &a
	fmt.Println(pa) //&[4 5 6]
	//使用*读取/修改指针指向的值
	i1 := new(int)
	*i1 = 3
	fmt.Println(i1, *i1) //0xc082048098 3
}
output==>
0xc082048038
&[4 5 6]
0xc082048098 3
</pre>
###通道 Channel
channel用于两个goroutine之间传递指定类型的值来同步运行和通讯。操作符<-用于指定channel的方向，发送或接收。如果未指定方向，则为双向channel。

channel是引用类型，使用make函数来初始化。未初始化的channel零值是nil，且不能用于发送和接收值。
<pre>
package main

import "fmt"

func main() {
	/*
		关闭channel，只能用于双向或只发送类型的channel
		只能由 发送方调用close函数来关闭channel
		接收方取出已关闭的channel中发送的值后，后续再从channel中取值时会以非阻塞的方式立即返回channel传递类型的零值。
	*/
	ch := make(chan string, 1)
	ch <- "hello"
	close(ch)
	s, ok := <-ch
	if ok {
		fmt.Println("receive value from sender:", s)
	} else {
		fmt.Println("get zero value from closed channel")
	}
	/*//向已关闭的通道发送值会产生运行时恐慌panic
	ch <- "hi"
	fmt.Println(<-ch)
	// 再次关闭已经关闭的通道也会产生运行时恐慌panic
	close(ch)
	*/

	//使用for range语句依次读取发送到channel的值，直到channel关闭。
	var chh = make(chan int)
	go func() {
		for i := 0; i < 5; i++ {
			chh <- i
		}
		close(chh)
	}()
	for x := range chh {
		fmt.Printf("%d-", x)
	}
}
output==>
receive value from sender: hello
0-1-2-3-4-
</pre>
在实际的项目中，我们的程序一般都是很多个goroutine同时工作，知道所有goroutine是否都完成不是一件容易的事情。以前的经验是通过轮询的方式，但是在golang中这种方式比较浪费性能。
<pre>
package main

import (
	"fmt"
)

var (
	flag bool
	str  string
)

var ch chan string = make(chan string)
/*
不要用无限轮询的方式来检查goroutine是否完成,而是要通过使用channel，
让foo()和main()实现通信，让foo()执行完毕后通过channel发送一个消息给main()，
告诉它自己的事儿完成了，然后main()收到消息后继续执行其他操作
*/
func foo() {
	flag = true
	str = "setup complete"
	ch <- "I am complete"
}
func main() {
	go foo()
	<-ch
	for !flag {
	}
	fmt.Println(str)
}

</pre>
使用channel精确控制goroutine的数量
<pre>
package main

import "fmt"

var ch chan int = make(chan int)

func afunction(ch chan int, t int) {
	fmt.Println(t+1, ":finished")
	<-ch
}

func main() {
	for i := 0; i < 7; i++ {
		go afunction(ch, i)
		ch <- 1
	}
}
output==>
1 :finished
2 :finished
3 :finished
4 :finished
5 :finished
6 :finished
7 :finished
</pre>
###switch case goto break continue
<pre>
package main

import "fmt"

func main() {
	x := 2
	//分支选择 switch
	switch x {
	case 0:
		fmt.Println("x=0")
	case 1:
		fmt.Println("x=1")
	case 2:
		fmt.Println("x=2")
	default:
		fmt.Println("default value")
	}
	switch {
	case x == 1:
		fmt.Println("1")
	case x == 2:
		fmt.Println("bingo 2")
	default:
		fmt.Println("default value")
	}
	//循环
	sl := []int{2, 4, 5, 6, 7}
	for i := 0; i < len(sl); i++ {
		fmt.Printf("%d ", sl[i])
	}
	fmt.Println()
	for k, v := range sl {
		fmt.Println("key:", k, "value:", v) //下标,值
	}
	//循环的继续、中断、跳转
	for k, v := range sl {
		if v == 2 {
			fmt.Println(k)
			continue
		} else if v == 5 {
			break
		} else {
			goto JASON
		}
	JASON:
		fmt.Println("goto action done")
	}

}
output==>
x=2
bingo 2
2 4 5 6 7 
key: 0 value: 2
key: 1 value: 4
key: 2 value: 5
key: 3 value: 6
key: 4 value: 7
0
goto action done
</pre>
###有缓冲 无缓冲channel
无缓冲
<pre>
package main

import "fmt"

func writeRoutine(test_chan chan int, value int) {
	test_chan <- value
}
func readRoutine(test_chan chan int) {
	<-test_chan
	return
}
func main() {
	c := make(chan int)
	x := 100
	go writeRoutine(c, x)
	readRoutine(c)
	fmt.Println(x)
}
outout==>
100
</pre>
有缓冲
<pre>
package main 
import "fmt"
var c = make(chan int, 1)

func f() {
     c <- 'c'
 
     fmt.Println("在goroutine内")
 }
 
 func main() {
     go f()
 
     c <- 'c'  //要是没有这个，则"在main中"没有打印
     <-c
     <-c
 
     fmt.Println("在main中")  
 }
output==>
在goroutine内
在main中
</pre>
###方法
<pre>
package main

import "fmt"

type A struct {
	x, y int
}

/*// 定义结构体的方法，'_'表示方法内忽略使用结构体、字段及其他方法
func (_ A) echo_A() {
	fmt.Println("(_ A)")
} */
// 同上
func (A) echoA(s string) {
	fmt.Println("(_A)", s)
}

/*
func (_ *A) echo_PA() {
	fmt.Println("(_ *A)")
}
*/
// 同上
func (*A) echo_PA(s string) {
	fmt.Println("(*A)", s)
}

//定义结构体的方法，方法内可以引用结构，字段及其他方法
func (a A) setX(x int) {
	a.x = x
}

// 定义结构体指针的方法，方法内可以引用结构体、结构体指针、字段及其他方法
func (a *A) setY(y int) {
	a.y = y
}
func main() {
	var a A
	a.setX(3)
	a.setY(9)
	fmt.Println(a.x, a.y)
	a.echoA("a")       //(_A) a
	a.echo_PA("jason") //(*A) jason
}
output==>
0 9
(_A) a
(*A) jason
</pre>
###并发 Concurrency
使用关键字go调用一个函数/方法，启动一个新的协程goroutine
<pre>
package main

import "time"

//主协程goroutine输出0，其他由go启动的几个子协程分别输出1～5
func say(i int) {
	println("goroutine:", i)
}
func main() {
	for i := 1; i <= 5; i++ {
		go say(i)
	}
	say(0)
	time.Sleep(5 * time.Second)
}
output==>
goroutine: 0
goroutine: 1
goroutine: 2
goroutine: 3
goroutine: 4
goroutine: 5
</pre>
goroutine 在相同的地址空间中运行，因此访问共享内存必须进行同步
<pre>
package main

import "time"
import "sync"

var mu sync.Mutex
var i int

func add() {
	/*
		使用互斥锁防止多个协程goroutine同时修改共享变量
		只能限制同时访问此方法修改变量，在方法外修改则限制是无效的
	*/
	mu.Lock()
	defer mu.Unlock()
	i++
}
func main() {
	for range [100]byte{} {
		go add()
	}
	time.Sleep(1 * time.Second)
	println(i)
}
output==>
100
</pre>
使用通道channel进行同步
<pre>
package main

import "time"

var i int
var ch = make(chan byte, 1)

//将channel用作同步开关
func main() {
	for range [100]byte{} {
		go add()
	}
	time.Sleep(1 * time.Second)
	println(i)
}
func add() {
	ch <- 0
	i++
	<-ch
}
output==>
100
</pre>
使用channel在不同的goroutine之间通信
<pre>
package main

import "time"

var i int
var ch = make(chan int, 1)

func add() {
	x := <-ch
	x++
	ch <- x
}
func main() {
	for range [100]byte{} {
		go add()
	}
	ch <- i
	time.Sleep(1 * time.Second)
	i = <-ch
	println(i)
}
output==>
100
</pre>
###测试 Testing
Go中自带轻量级的测试框架testing和自带的go test命令来实现单元测试和基准测试
####单元测试 Unit
- 测试源文件名必须是_test.go结尾的，go test的时候才会执行到相应的代码
- 必须import testing包
- 所有的测试用例函数必须以Test开头
- 测试用例按照源码中编写的顺序依次执行
- 测试函数TestXxx()的参数是*testing.T，可以使用该类型来记录错误或者是测试状态
- 测试格式：func TestXxx (t *testing.T)，Xxx部分可以为任意的字母数字的组合，首字母不能是小写字母[a-z]，例如Testsum是错误的函数名。
- 函数中通过调用*testing.T的Error，Errorf，FailNow，Fatal，FatalIf方法标注测试不通过，调用Log方法用来记录测试的信息。

测试分两个文件，分别是:

- test.go
<pre>
package testgo
import "math"
func Sum(min, max int) (sum int) {
	if min < 0 || max < 0 || max > math.MaxInt32 || min > max {
		return 0
	}
	for ; min <= max; min++ {
		sum += min
	}
	return
}
</pre>

- test_test.go

<pre>
package testgo
import "testing"
func TestSum(t *testing.T) {
	s := Sum(1, 0)
	t.Log("Sum 1 to 0:", s)
	if 0 != s {
		t.Error("not equal.")
	}
	s = Sum(1, 10)
	t.Log("Sum 1 to 10:", s)
	if 55 != s {
		t.Error("not equal.")
	}
}
</pre>
在当前位置执行测试：
<pre>
go test -v

输出：
=== RUN   TestSum
--- PASS: TestSum (0.00s)
	test_test.go:7: Sum 1 to 0: 0
	test_test.go:12: Sum 1 to 10: 55
PASS
ok  	test	0.237s
</pre>
####基准测试 Benchmark
基准测试 Benchmark用来检测函数/方法的性能。

- 基准测试用例函数必须以Benchmark开头
- go test默认不会执行基准测试的函数，需要加上参数-test.bench，语法:-test.bench="test_name_regex"，例如go test -test.bench=".*"表示测试全部的基准测试函数
- 在基准测试用例中，在循环体内使用testing.B.N，使测试可以正常的运行

测试分两个文件，分别是:

- test.go
<pre>
//test.go
package testgo
import "math"
func Sum(min, max int) (sum int) {
	if min < 0 || max < 0 || max > math.MaxInt32 || min > max {
		return 0
	}

	for ; min <= max; min++ {
		sum += min
	}
	return
}
</pre>
- test_test.go
<pre>
package testgo
import "testing"
func BenchmarkSum(b *testing.B) {
    b.Logf("Sum 1 to %d: %d\n", b.N, Sum(1, b.N))
}
</pre>
在当前位置执行测试： (注意bench后面还有一个 . )
<pre>
go test -v -bench .   

	输出：
	PASS
	BenchmarkSum-2	2000000000	         0.69 ns/op
	--- BENCH: BenchmarkSum-2
		test_test.go:6: Sum 1 to 1: 1
		test_test.go:6: Sum 1 to 100: 5050
		test_test.go:6: Sum 1 to 10000: 50005000
		test_test.go:6: Sum 1 to 1000000: 500000500000
		test_test.go:6: Sum 1 to 100000000: 5000000050000000
		test_test.go:6: Sum 1 to 2000000000: 2000000001000000000
	ok  	test	1.697s
	
	testing: warning: no tests to run
</pre>

###简单web服务器

<pre>
package main
import "fmt"
import "net/http"
import "log"
func sayhi(w http.ResponseWriter, r *http.Request) {
	fmt.Fprintf(w, "Hello Jason")
}
func jason(w http.ResponseWriter, r *http.Request) {
	fmt.Fprintf(w, "I am jason")
}
func main() {
	http.HandleFunc("/", sayhi)
	http.HandleFunc("/jason", jason)
	err := http.ListenAndServe(":8089", nil)
	if err != nil {
		log.Fatal("listenandserve:", err)
	}
}
</pre>
###模板 template
<pre>
func temp(w http.ResponseWriter, r *http.Request) {
	t := template.New("some template")      //创建一个模板
	t, _ = t.ParseFiles("welcome.tpl", nil) //解析模板文件
	user := Getuser()                       //获取用户信息
	t.Execute(w, user)                      //执行模板的merge操作
}
/*
Parse与ParseFiles：Parse可以直接测试一个字符串，而不需要额外的文件；ParseFiles只能解析文件;
os.Stdout与http.ResponseWriter:os.Stdout实现了io.Writer接口，而http.ResponseWriter没有.
实现了io.Writer接口就可以用于t.Execute(),比如 temp.Execute(os.Stdout,element) [temp:待渲染的模板或则字符串,element:写入模板的参数] 。没有实现io.Writer接口的则会报如下的错误：
.\test.go:15: cannot use http.ResponseWriter (type func(...interface {}) (int, error)) as type io.Writer in argument to t.Execute:

*/
</pre>
####模板中如何插入数据？
上面我们演示了如何解析并渲染模板，接下来让我们来更加详细的了解如何把数据渲染出来。一个模板都是应用在一个Go的对象之上，Go对象的字段如何插入到模板中呢？
#####字段操作
Go语言的模板通过{{}}来包含需要在渲染时被替换的字段，{{.}}表示当前的对象，这和Java或者C++中的this类似，如果要访问当前对象的字段通过{{.FieldName}},但是需要注意一点：这个字段必须是导出的(字段首字母必须是大写的),否则在渲染的时候就会报错，请看下面的这个例子：
<pre>
package main

import "html/template"
import "os"

type Person struct {
	Username string // 字段首字母必须大写，否则失败
}

func main() {
	t := template.New("fieldname example")
	t, _ = t.Parse("hello {{.Username }}")
	p := Person{Username: "Jason"}
	t.Execute(os.Stdout, p)
}
output==>
hello Jason
</pre>
#####输出嵌套字段内容
上面我们例子展示了如何针对一个对象的字段输出，那么如果字段里面还有对象，如何来循环的输出这些内容呢？我们可以使用{{with …}}…{{end}}和{{range …}}{{end}}来进行数据的输出。

- {{range}} 这个和Go语法里面的range类似，循环操作数据
- {{with}}操作是指当前对象的值，类似上下文的概念

<pre>
package main
import "html/template"
import "os"
type Friend struct {
	Fname string
}
type Person struct {
	Username string
	Emails   []string
	Friends  []*Friend
}
func main() {
	f1 := Friend{Fname: "Jack"}
	f2 := Friend{Fname: "Jason"}
	t := template.New("fieldname template")
	/*{{with}}操作是指当前对象的值，类似上下文的概念*/
	t, _ = t.Parse(`
		hello {{.Username}}!
		{{range .Emails}}
			an email {{.}}
		{{end}}
		{{with .Friends}}  
		{{range .}}
			my friend name is {{.Fname}}
		{{end}}
		{{end}}
	`)
	p := Person{Username: "Jason",
		Emails:  []string{"jason@qq.com", "jason@163.com"},
		Friends: []*Friend{&f1, &f2}}
	t.Execute(os.Stdout, p)
}
output==>
hello Jason!
			
	an email jason@qq.com

	an email jason@163.com

	my friend name is Jack

	my friend name is Jason
</pre>

#####条件处理
在Go模板里面如果需要进行条件判断，那么我们可以使用和Go语言的if-else语法类似的方式来处理，如果pipeline为空，那么if就认为是false，下面的例子展示了如何使用if-else语法：
<pre>
package main

import (
	"os"
	"text/template"
)

func main() {
	tEmpty := template.New("template test")
	tEmpty = template.Must(tEmpty.Parse("空的pipeline if demo:{{if ``}}不会输出.{{end}}\n"))
	tEmpty.Execute(os.Stdout, nil)

	tWithValue := template.New("template test")
	tWithValue = template.Must(tWithValue.Parse("不为空的pipeline if demo:{{if `anything`}} 我有内容，我会输出.{{end}}\n"))
	tWithValue.Execute(os.Stdout, nil)

	tIfElse := template.New("template test")
	tIfElse = template.Must(tIfElse.Parse("if-else demo: {{if `anything`}} if部分 {{else}} else部分.{{end}}\n"))
	tIfElse.Execute(os.Stdout, nil)
}
output==>
空的pipeline if demo:
不为空的pipeline if demo: 我有内容，我会输出.
if-else demo:  if部分
</pre>
#####pipelines
Unix用户已经很熟悉什么是pipe了，ls | grep "beego"类似这样的语法你是不是经常使用，过滤当前目录下面的文件，显示含有"beego"的数据，表达的意思就是前面的输出可以当做后面的输入，最后显示我们想要的数据，而Go语言模板最强大的一点就是支持pipe数据，<font color=red>在Go语言里面任何{{}}里面的都是pipelines数据</font>，例如我们上面输出的email里面如果还有一些可能引起XSS注入的，那么我们如何来进行转化呢？
<pre>
{{. | html}}
</pre>
在email输出的地方我们可以采用如上方式可以把输出全部转化html的实体，上面的这种方式和我们平常写Unix的方式是不是一模一样，操作起来相当的简便，调用其他的函数也是类似的方式。
#####模板变量
有时候，我们在模板使用过程中需要定义一些局部变量，我们可以在一些操作中申明局部变量，例如withrangeif过程中申明局部变量，这个变量的作用域是{{end}}之前，Go语言通过申明的局部变量格式如下所示：
<pre>
$variable := pipeline
</pre>
详细的例子看下面的：
<pre>
{{with $x := "output" | printf "%q"}}{{$x}}{{end}}
{{with $x := "output"}}{{printf "%q" $x}}{{end}}
</pre>
#####模板函数
模板在输出对象的字段值时，采用了fmt包把对象转化成了字符串。但是有时候我们的需求可能不是这样的，例如有时候我们为了防止垃圾邮件发送者通过采集网页的方式来发送给我们的邮箱信息，我们希望把@替换成at例如：astaxie at beego.me，如果要实现这样的功能，我们就需要自定义函数来做这个功能。

每一个模板函数都有一个唯一值的名字，然后与一个Go函数关联，通过如下的方式来关联
<pre>
type FuncMap map[string]interface{}
</pre>
例如，如果我们想要的email函数的模板函数名是emailDeal，它关联的Go函数名称是EmailDealWith,那么我们可以通过下面的方式来注册这个函数
<pre>
t = t.Funcs(template.FuncMap{"emailDeal": EmailDealWith})
</pre>
EmailDealWith这个函数的参数和返回值定义如下：
<pre>
func EmailDealWith(args …interface{}) string
</pre>
例子如下：
<pre>
package main

import (
	"fmt"
	"html/template"
	"os"
	"strings"
)

type Friend struct {
	Fname string
}

type Person struct {
	Username string
	Emails   []string
}

func EmailDealWith(args ...interface{}) string {
	ok := false
	var s string
	if len(args) == 1 {
		s, ok = args[0].(string)
	}
	if !ok {
		s = fmt.Sprint(args...)
	}
	substrs := strings.Split(s, "@")
	if len(substrs) != 2 {
		return s
	}
	return (substrs[0] + " AT " + substrs[1])
}
func main() {
	t := template.New("filedname test")
	t = t.Funcs(template.FuncMap{"emailDeal": EmailDealWith})
	t, _ = t.Parse(`
		hello {{.Username}}!
	        {{range .Emails}}
	            an emails {{.|emailDeal}}
	        {{end}}
	`)
	p := Person{Username: "Jason", Emails: []string{"jason@qq.com", "jason@163.com", "jack@sina.com"}}
	t.Execute(os.Stdout, p)
}
output==>
hello Jason!
		        
    an emails jason AT qq.com

    an emails jason AT 163.com

    an emails jack AT sina.com
</pre>
类似的还有一个是重写gt eq lt等常见筛选条件：
<pre>
package main

import (
	"os"
	"text/template"
)

type Person struct {
	Name string
	Age  int
}

func main() {

	t := template.Must(
		template.New("test").Funcs(
			template.FuncMap{
				"lt": func(a, b int) bool { return a < b },
				"eq": func(a, b int) bool { return a == b },
				"gt": func(a, b int) bool { return a > b },
			},
		).Parse(
			"{{.Name}}:{{ if .Age | lt 5 }} 5 < age.{{else}} 5 > age.{{end}}\n",
		),
	)

	t.Execute(os.Stdout, &Person{
		Name: "lulu",
		Age:  4,
	})
	t.Execute(os.Stdout, &Person{
		Name: "lili",
		Age:  6,
	})
}
output==>
lulu: 5 > age.
lili: 5 < age.
</pre>
上面演示了如何自定义函数，其实，在模板包内部已经有内置的实现函数，下面代码截取自模板包里面
<pre>
var builtins = FuncMap{
    "and":      and,
    "call":     call,
    "html":     HTMLEscaper,
    "index":    index,
    "js":       JSEscaper,
    "len":      length,
    "not":      not,
    "or":       or,
    "print":    fmt.Sprint,
    "printf":   fmt.Sprintf,
    "println":  fmt.Sprintln,
    "urlquery": URLQueryEscaper,
}
</pre>
#####Must操作
模板包里面有一个函数Must，它的作用是检测模板是否正确，例如大括号是否匹配，注释是否正确的关闭，变量是否正确的书写。接下来我们演示一个例子，用Must来判断模板是否正确：
<pre>
package main

import "fmt"
import "text/template"

func main() {
	tok := template.New("first")
	template.Must(tok.Parse("some static text /*and a comment*/"))
	fmt.Println("The first one parsed ok")

	template.Must(template.New("second").Parse("some static text {{ .Name }}"))
	fmt.Println("the second one parsed ok")

	fmt.Println("the next one ought to fail")
	tErr := template.New("check parse error with Must")
	template.Must(tErr.Parse("some static text {{ .Name }"))
}
output==>
The first one parsed ok
the second one parsed ok
the next one ought to fail

panic: template: check parse error with Must:1: unexpected "}" in operand

goroutine 1 [running]:
panic(0x556d40, 0xc08202a3e0)
	D:/go/src/runtime/panic.go:481 +0x3f4
text/template.Must(0x0, 0x760000, 0xc08202a3e0, 0x0)
	D:/go/src/text/template/helper.go:23 +0x52
main.main()
	D:/gopath/src/test/test.go:16 +0x928
exit status 2

exit status 1
</pre>
#####嵌套模板
我们平常开发Web应用的时候，经常会遇到一些模板有些部分是固定不变的，然后可以抽取出来作为一个独立的部分，例如一个博客的头部和尾部是不变的，而唯一改变的是中间的内容部分。所以我们可以定义成header、content、footer三个部分。Go语言中通过如下的语法来申明
<pre>
{{define "子模板名称"}}内容{{end}}
</pre>
通过如下方式来调用：
<pre>
{{template "子模板名称"}}
</pre>
接下来我们演示如何使用嵌套模板，我们定义三个文件，header.tmpl、content.tmpl、footer.tmpl文件，里面的内容如下

header.tmpl
<pre>
{{define "header"}}
<html>
<head>
    <title>演示信息</title>
</head>
<body>
{{end}}
</pre>
content.tmpl
<pre>
//content.tmpl
{{define "content"}}
{{template "header"}}
<h1>演示嵌套</h1>
<ul>
    <li>嵌套使用define定义子模板</li>
    <li>调用使用template</li>
</ul>
{{template "footer"}}
{{end}}
</pre>
footer.tmpl
<pre>
//footer.tmpl
{{define "footer"}}
</body>
</html>
{{end}}
</pre>
演示如下：
<pre>
package main

import (
    "fmt"
    "os"
    "text/template"
)

func main() {
    s1, _ := template.ParseFiles("header.tmpl", "content.tmpl", "footer.tmpl")
    s1.ExecuteTemplate(os.Stdout, "header", nil)
    fmt.Println()
    s1.ExecuteTemplate(os.Stdout, "content", nil)
    fmt.Println()
    s1.ExecuteTemplate(os.Stdout, "footer", nil)
    fmt.Println()
    s1.Execute(os.Stdout, nil)
}
</pre>
通过上面的例子我们可以看到通过template.ParseFiles把所有的嵌套模板全部解析到模板里面，其实每一个定义的{{define}}都是一个独立的模板，他们相互独立，是并行存在的关系，内部其实存储的是类似map的一种关系(key是模板的名称，value是模板的内容)，然后我们通过ExecuteTemplate来执行相应的子模板内容，我们可以看到header、footer都是相对独立的，都能输出内容，content 中因为嵌套了header和footer的内容，就会同时输出三个的内容。但是当我们执行s1.Execute，没有任何的输出，因为在默认的情况下没有默认的子模板，所以不会输出任何的东西。
#####将struct传入模板
<pre>
package main

import (
	"os"
	"text/template"
)

type Inventory struct {
	Material string
	Count    uint
}

func main() {
	sweaters := Inventory{"wool", 34}
	muban := "{{.Count}} items are made of {{.Material}}"
	tmpl, err := template.New("test").Parse(muban)
	if err != nil {
		panic(err)
	}
	err = tmpl.Execute(os.Stdout, sweaters) //将struct与模板合成，合成结果放到os.Stdout里
	if err != nil {
		panic(err)
	}
}
outout==>
34 items are made of wool
</pre>

##深入理解 net/http 
###第一版
<pre>
package main

import (
	"io"
	"net/http"
)

/*
Hander是啥呢，它是一个接口。这个接口很简单，只要某个struct
有ServeHTTP(http.ResponseWriter, *http.Request)这个方法，
那这个struct就自动实现了Hander接口
*/

func sayhi(w http.ResponseWriter, r *http.Request) {
	io.WriteString(w, "hello jason")
}

func main() {
	http.HandleFunc("/", sayhi)       //注册一个sayhello函数给“/”，当浏览器浏览“/”的时候，会调用sayhello函数
	http.ListenAndServe(":8089", nil) //开始监听和服务
}
</pre>
###第二版
认识http.ResponseWriter

当http.ListenAndServe(":8080", &a{})后，开始等待有访问请求
一旦有访问请求过来，http包帮我们处理了一系列动作后，最后他会去调用a的ServeHTTP这个方法，并把自己已经处理好的http.ResponseWriter, *http.Request传进去
而a的ServeHTTP这个方法，拿到*http.ResponseWriter后，并往里面写东西，客户端的网页就显示出来了
<pre>
package main

//重写ServeHTTP方法
import (
	"io"
	"net/http"
)

type a struct{}

func (*a) ServeHTTP(w http.ResponseWriter, r *http.Request) {
	io.WriteString(w, "version 1")
}
func main() {
	http.ListenAndServe(":8089", &a{})
}
</pre>
认识*http.Request
<pre>
package main

import (
	"io"
	"net/http"
)

type a struct{}

func (*a) ServeHTTP(w http.ResponseWriter, r *http.Request) {
	path := r.URL.String() //获取访问的路径
	io.WriteString(w, path)
}

func main() {
	http.ListenAndServe(":8089", &a{})
}
output==>
地址栏输入：http://localhost:8089/ffffffffffffffffffffff
/ffffffffffffffffffffff
</pre>
一个非常简单的网站
<pre>
package main

import (
	"io"
	"net/http"
)

type a struct{}

func (*a) ServeHTTP(w http.ResponseWriter, r *http.Request) {
	path := r.URL.String()
	switch path {
	case "/":
		io.WriteString(w, "<h1>ROOT</h1><a href=\"abc\">abc</a> | <a href=\"hello\">hello</a>")
	case "/abc":
		io.WriteString(w, "<h1>ABC</h1><a href=\"/\">root</a>")
	case "/hello":
		io.WriteString(w, "<h1>HELLO</h1><a href=\"/\">root</a>")
	}
}

func main() {
	http.ListenAndServe(":8089", &a{})
}
/*
运行后，可以看出，一个case就是一个页面
如果一个网站有上百个页面，那是否要上百个case？
很不幸，是的
那管理起来岂不是要累死？
要累死，不过，还好有ServeMux
*/
</pre>
###第三版-用ServeMux拯救上面的问题
ServeMux大致作用是，他有一张map表，map里的key记录的是r.URL.String()，而value记录的是一个方法，这个方法ServeHTTP是一样的，这个方法有一个别名，叫HandlerFunc.ServeMux还有一个方法名字是Handle，他是用来注册HandlerFunc 的.
ServeMux还有另一个方法名字是ServeHTTP，这样ServeMux是实现Handler接口的，否者无法当http.ListenAndServe的第二个参数传输.
<pre>
package main

import (
	"io"
	"net/http"
)

type b struct{}

func (*b) ServeHTTP(w http.ResponseWriter, r *http.Request) {
	io.WriteString(w, "version 2")
}

func main() {
	mux := http.NewServeMux()
	mux.Handle("/hi", &b{})
	http.ListenAndServe(":8089", mux)
}
/*
mux := http.NewServeMux():新建一个ServeMux。
mux.Handle("/", &b{}):注册路由，把"/"注册给b这个实现Handler接口的struct，注册到map表中。
http.ListenAndServe(":8080", mux)第二个参数是mux。
运行时，因为第二个参数是mux，所以http会调用mux的ServeHTTP方法。
ServeHTTP方法执行时，会检查map表（表里有一条数据，key是“/h”，value是&b{}的ServeHTTP方法）
如果用户访问/h的话，mux因为匹配上了，mux的ServeHTTP方法会去调用&b{}的 ServeHTTP方法，从而打印hello
如果用户访问/abc的话，mux因为没有匹配上，从而打印404 page not found

ServeMux就是个二传手！
*/
</pre>
ServeMux的HandleFunc方法
<pre>
package main

/*
发现了没有，b这个struct仅仅是为了装一个ServeHTTP而存在，所以能否跳过b呢，
ServeMux说：可以 mux.HandleFunc是用来注册func到map表中的
*/
import (
	"io"
	"net/http"
)

func main() {
	mux := http.NewServeMux()
	mux.HandleFunc("/hi", func(w http.ResponseWriter, r *http.Request) {
		io.WriteString(w, "hi")
	})
	mux.HandleFunc("/hello", func(w http.ResponseWriter, r *http.Request) {
		io.WriteString(w, "hello")
	})
	mux.HandleFunc("/", func(w http.ResponseWriter, r *http.Request) {
		io.WriteString(w, "ROOT")
	})

	http.ListenAndServe(":8089", mux)
}
</pre>
###time 
time.Sleep
<pre>
fmt.Println("start sleeping...")
time.Sleep(time.Second)
fmt.Println("end sleep.")
//【结果】打印start sleeping后，等了正好1秒后，打印了end sleep
//会阻塞，Sleep时，什么事情都不会做
</pre>
time.After
<pre>
package main

import "fmt"
import "time"

func main() {
	fmt.Println("the 1")
	//返回一个time.C这个管道，1秒(time.Second)后会在此管道中放入一个时间点
	tc := time.After(time.Second)
	fmt.Println("the 2")
	fmt.Println("the 3")
	<-tc
	fmt.Println("the 4")
}
//【结果】立即打印123，等了1秒不到一点点的时间，打印了4，结束
//打印the 1后，获得了一个空管道，这个管道1秒后会有数据进来
//打印the 2，（这里可以做更多事情）
//打印the 3
//等待，直到可以取出管道的数据（取出数据的时间与获得tc管道的时间正好差1秒钟）
//打印the 4
</pre>
time.AfterFunc

time.AfterFunc(time.Duration,func());
和After差不多，意思是多少时间之后在goroutine line执行函数.
<pre>
package main

import "time"
import "fmt"

func main() {
	f := func() {
		fmt.Println("time out")
	}
	time.AfterFunc(time.Second, f)
	time.Sleep(2 * time.Second)
}

//【结果】运行了1秒后，打印出timeout，又过了1秒，程序退出
//将一个间隔和一个函数给AfterFunc后
//间隔时间过后，执行传入的函数
</pre>
time.Tick

每隔多少时间后
<pre>
package main

import "time"
import "fmt"

func main() {
	fmt.Println("the 1")
	tc := time.Tick(time.Second)

	for i := 1; i <= 5; i++ {
		<-tc
		fmt.Println("hello")
	}
}
/*
首先打印一个 the 1
然后每隔1秒，打印一个hello
*/
</pre>
Before & After方法、

判断一个时间点是否在另一个时间点的前面（后面），返回true或false
<pre>
t1:=time.Now()
time.Sleep(time.Second)
t2:=time.Now()
a:=t2.After(t1)     //t2的记录时间是否在t1记录时间的**后面**呢，是的话，a就是true
fmt.Println(a)       //true
b:=t2.Before(t1)     //t2的记录时间是否在t1记录时间的**前面**呢，是的话，b就是true
fmt.Println(b)       //false
</pre>
Sub方法

两个时间点相减，获得时间差（Duration）
<pre>
t1:=time.Now()
time.Sleep(time.Second)
t2:=time.Now()
d:=t2.Sub(t1)     //时间2减去时间1
fmt.Println(d)       //打印结果差不多为1.000123几秒，因为Sleep无法做到精确的睡1秒
后发生的时间  减去   先发生时间，是正数
</pre>
Add方法

拿一个时间点，add一个时长，获得另一个时间点
<pre>
t1:=time.Now()              //现在是12点整（假设）,那t1记录的就是12点整
t2:=t1.Add(time.Hour)          //那t1的时间点 **加上(Add)** 1个小时，是几点呢？
fmt.Println(t2)       //13点（呵呵）
</pre>
###Golang处理表单输入
<pre>
package main

import (
	"fmt"
	"html/template"
	"log"
	"net/http"
	_ "strings"
)

func sayhelloName(w http.ResponseWriter, r *http.Request) {
	r.ParseForm() //解析url传递的参数,对于POST则解析响应包的主体(request body)
	//注意:如果没有调用ParseForm方法,下面无法获取表单的数据
	/*fmt.Println(r.Form) //这些信息是输出到服务器端的打印信息
	fmt.Println("path", r.URL.Path)
	fmt.Println("scheme", r.URL.Scheme)
	fmt.Println(r.Form["url_long"])
	for k, v := range r.Form {
		fmt.Println("key:", k)
		fmt.Println("val:", strings.Join(v, ""))
	} */
	fmt.Fprintf(w, "Hello Jason!") //这个写入到w的是输出到客户端的
}

func login(w http.ResponseWriter, r *http.Request) {
	fmt.Println("method:", r.Method) //获取请求的方法
	if r.Method == "GET" {
		t, _ := template.ParseFiles("login.gtpl")
		t.Execute(w, nil)
	} else {
		//请求的是登陆数据，那么执行登陆的逻辑判断
		/*
			默认情况下，Handler里面是不会自动解析form的，必须显式的调用r.ParseForm()后，
			你才能对这个表单数据进行操作。我们修改一下代码，
			在fmt.Println("username:", r.Form["username"])之前加一行r.ParseForm(),重新编译
		*/
		r.ParseForm()
		fmt.Println("username:", r.Form["username"])
		fmt.Println("password:", r.Form["password"])
	}
}

func main() {
	http.HandleFunc("/", sayhelloName)       //设置访问的路由
	http.HandleFunc("/login", login)         //设置访问的路由
	err := http.ListenAndServe(":9090", nil) //设置监听的端口
	if err != nil {
		log.Fatal("ListenAndServe: ", err)
	}
}
//其中一种情况
method: GET
method: POST
username: [rrtrt]
password: [rtt]
</pre>
###验证表单的输入
验证项目包括

- 必填字段
- 是否数字
- 是否中文
- 英文
- 电子邮件地址
- 手机号码
- 下拉菜单
- 单选按钮
- 复选框
- 日期和时间
- 身份证号码

见 https://github.com/astaxie/build-web-application-with-golang/blob/master/zh/04.2.md
###客户端上传文件
<pre>
package main

import (
    "bytes"
    "fmt"
    "io"
    "io/ioutil"
    "mime/multipart"
    "net/http"
    "os"
)

func postFile(filename string, targetUrl string) error {
    bodyBuf := &bytes.Buffer{}
    bodyWriter := multipart.NewWriter(bodyBuf)

    //关键的一步操作
    fileWriter, err := bodyWriter.CreateFormFile("uploadfile", filename)
    if err != nil {
        fmt.Println("error writing to buffer")
        return err
    }

    //打开文件句柄操作
    fh, err := os.Open(filename)
    if err != nil {
        fmt.Println("error opening file")
        return err
    }
    defer fh.Close()

    //iocopy
    _, err = io.Copy(fileWriter, fh)
    if err != nil {
        return err
    }

    contentType := bodyWriter.FormDataContentType()
    bodyWriter.Close()

    resp, err := http.Post(targetUrl, contentType, bodyBuf)
    if err != nil {
        return err
    }
    defer resp.Body.Close()
    resp_body, err := ioutil.ReadAll(resp.Body)
    if err != nil {
        return err
    }
    fmt.Println(resp.Status)
    fmt.Println(string(resp_body))
    return nil
}
//sample usage
func main() {
    target_url := "http://localhost:9090/upload"
    filename := "./astaxie.pdf"
    postFile(filename, target_url)
}
</pre>
###数据库接口
在我们使用database/sql接口和第三方库的时候经常看到如下:
<pre>
  import (
      "database/sql"
      _ "github.com/mattn/go-sqlite3"
  )
</pre>
新手都会被这个 _ 所迷惑，其实这个就是Go设计的巧妙之处，我们在变量赋值的时候经常看到这个符号，它是用来忽略变量赋值的占位符，那么包引入用到这个符号也是相似的作用，这儿使用_的意思是引入后面的包名而不直接使用这个包中定义的函数，变量等资源。
###Golang Session Cookie 
- Cookie

Golang中通过net/http包中的SetCookie来设置：
<pre>
http.SetCookie(w ResponseWriter, cookie *Cookie)
</pre>
w表示需要写入的response，cookie是一个struct，让我们来看一下cookie对象是怎么样的
<pre>
type Cookie struct {
    Name       string
    Value      string
    Path       string
    Domain     string
    Expires    time.Time
    RawExpires string

// MaxAge=0 means no 'Max-Age' attribute specified.
// MaxAge<0 means delete cookie now, equivalently 'Max-Age: 0'
// MaxAge>0 means Max-Age attribute present and given in seconds
    MaxAge   int
    Secure   bool
    HttpOnly bool
    Raw      string
    Unparsed []string // Raw text of unparsed attribute-value pairs
}
</pre>
我们来看一个例子，如何设置cookie
<pre>
expiration := time.Now()
expiration = expiration.AddDate(1, 0, 0)
cookie := http.Cookie{Name: "username", Value: "jason", Expires: expiration}
http.SetCookie(w, &cookie)
</pre>
Golang读取cookie
上面的例子演示了如何设置cookie数据，我们这里来演示一下如何读取cookie
<pre>
cookie, _ := r.Cookie("username")
fmt.Fprint(w, cookie)
</pre>
还有另外一种读取方式
<pre>
for _, cookie := range r.Cookies() {
    fmt.Fprint(w, cookie.Name)
}
</pre>
可以看到通过request获取cookie非常方便。

- Session 

session是在服务器端实现的一种用户和服务器之间认证的解决方案，目前Go标准包没有为session提供任何支持，这小节我们将会自己动手来实现go版本的session管理和创建。
<pre>
//created by astaxie
package session

import (
	"crypto/rand"
	"encoding/base64"
	"fmt"
	"io"
	"net/http"
	"net/url"
	"sync"
	"time"
)

type Session interface {
	Set(key, value interface{}) error //set session value
	Get(key interface{}) interface{}  //get session value
	Delete(key interface{}) error     //delete session value
	SessionID() string                //back current sessionID
}

type Provider interface {
	SessionInit(sid string) (Session, error)
	SessionRead(sid string) (Session, error)
	SessionDestroy(sid string) error
	SessionGC(maxlifetime int64)
}

var provides = make(map[string]Provider)

// Register makes a session provide available by the provided name.
// If Register is called twice with the same name or if driver is nil,
// it panics.
func Register(name string, provide Provider) {
	if provide == nil {
		panic("session: Register provide is nil")
	}
	if _, dup := provides[name]; dup {
		panic("session: Register called twice for provide " + name)
	}
	provides[name] = provide
}

type Manager struct {
	cookieName  string     //private cookiename
	lock        sync.Mutex // protects session
	provider    Provider
	maxlifetime int64
}

func NewManager(provideName, cookieName string, maxlifetime int64) (*Manager, error) {
	provider, ok := provides[provideName]
	if !ok {
		return nil, fmt.Errorf("session: unknown provide %q (forgotten import?)", provideName)
	}
	return &Manager{provider: provider, cookieName: cookieName, maxlifetime: maxlifetime}, nil
}

//get Session
func (manager *Manager) SessionStart(w http.ResponseWriter, r *http.Request) (session Session) {
	manager.lock.Lock()
	defer manager.lock.Unlock()
	cookie, err := r.Cookie(manager.cookieName)
	if err != nil || cookie.Value == "" {
		sid := manager.sessionId()
		session, _ = manager.provider.SessionInit(sid)
		cookie := http.Cookie{Name: manager.cookieName, Value: url.QueryEscape(sid), Path: "/", HttpOnly: true, MaxAge: int(manager.maxlifetime)}
		http.SetCookie(w, &cookie)
	} else {
		sid, _ := url.QueryUnescape(cookie.Value)
		session, _ = manager.provider.SessionRead(sid)
	}
	return
}

//Destroy sessionid
func (manager *Manager) SessionDestroy(w http.ResponseWriter, r *http.Request) {
	cookie, err := r.Cookie(manager.cookieName)
	if err != nil || cookie.Value == "" {
		return
	} else {
		manager.lock.Lock()
		defer manager.lock.Unlock()
		manager.provider.SessionDestroy(cookie.Value)
		expiration := time.Now()
		cookie := http.Cookie{Name: manager.cookieName, Path: "/", HttpOnly: true, Expires: expiration, MaxAge: -1}
		http.SetCookie(w, &cookie)
	}
}

func (manager *Manager) GC() {
	manager.lock.Lock()
	defer manager.lock.Unlock()
	manager.provider.SessionGC(manager.maxlifetime)
	time.AfterFunc(time.Duration(manager.maxlifetime)*time.Second, func() { manager.GC() })
}

func (manager *Manager) sessionId() string {
	b := make([]byte, 32)
	if _, err := io.ReadFull(rand.Reader, b); err != nil {
		return ""
	}
	return base64.URLEncoding.EncodeToString(b)
}
</pre>
###xml文件处理
servers.xml
<pre>
<?xml version="1.0" encoding="utf-8"?>
<servers version="1">
    <server>
        <serverName>Shanghai_VPN</serverName>
        <serverIP>127.0.0.1</serverIP>
    </server>
    <server>
        <serverName>Beijing_VPN</serverName>
        <serverIP>127.0.0.2</serverIP>
    </server>
</servers>
</pre>
处理代码是：
<pre>
package main

import (
	"encoding/xml"
	"fmt"
	"io/ioutil"
	"os"
)

type Recurlyservers struct {
	XMLName     xml.Name `xml:"servers"`
	Version     string   `xml:"version,attr"`
	Svs         []server `xml:"server"`
	Description string   `xml:",innerxml"`
}

type server struct {
	XMLName    xml.Name `xml:"server"`
	ServerName string   `xml:"serverName"`
	ServerIP   string   `xml:"serverIP"`
}

func main() {
	file, err := os.Open("servers.xml") // For read access.
	if err != nil {
		fmt.Printf("error: %v", err)
		return
	}
	defer file.Close()
	data, err := ioutil.ReadAll(file)
	if err != nil {
		fmt.Printf("error: %v", err)
		return
	}
	v := Recurlyservers{}
	err = xml.Unmarshal(data, &v)
	if err != nil {
		fmt.Printf("error: %v", err)
		return
	}

	fmt.Println(v)
}
</pre>
###Json处理

- 解析到结构体
<pre>
package main
import "encoding/json"
import "fmt"
type Server struct {
	Servername string
	Serverip   string
}
type Serverslice struct {
	Servers []Server
}
func main() {
	var s Serverslice
	str := `{"Servers":[{"Servername":"shanghai","ServerIP":"127.0.0.1"},{"Servername":"beijing","Serverip":"127.0.0.3"}]}`
	json.Unmarshal([]byte(str), &s)
	fmt.Println(s)
}
output==>
{[{shanghai 127.0.0.1} {beijing 127.0.0.3}]}
</pre>

- 解析到interface

我们知道interface{}可以用来存储任意数据类型的对象，这种数据结构正好用于存储解析的未知结构的json数据的结果。JSON包中采用map[string]interface{}和[]interface{}结构来存储任意的JSON对象和数组。Go类型和JSON类型的对应关系如下：

- bool 代表 JSON booleans
- float64 代表 JSON numbers
- string 代表 JSON strings
- nil 代表 JSON null

对于未知结构的json，建议使用https://github.com/bitly/go-simplejson。

- 生成Json

我们开发很多应用的时候，最后都是要输出JSON数据串，那么如何来处理呢？JSON包里面通过Marshal函数来处理，函数定义如下：
<pre>
func Marshal(v interface{}) ([]byte, error)
</pre>
例子 ：
<pre>
package main

import "encoding/json"
import "fmt"

type Server struct {
	Servername string
	Serverip   string
}

type Serverslice struct {
	Servers []Server
}

func main() {
	var s Serverslice
	s.Servers = append(s.Servers, Server{Servername: "shanghai", Serverip: "1234.56.45.56"})
	s.Servers = append(s.Servers, Server{Servername: "beijing", Serverip: "55.87.67.8"})
	b, err := json.Marshal(s)
	if err != nil {
		fmt.Println("json err :", err)
	}
	fmt.Println(string(b))
}
output==>
{"Servers":[{"Servername":"shanghai","Serverip":"1234.56.45.56"},{"Servername":"beijing","Serverip":"55.87.67.8"}]}
</pre>
###regexp 正则
使用正则来过滤或截取抓取到的百度搜索首页内容
<pre>
package main

import (
	"fmt"
	"io/ioutil"
	"net/http"
	"regexp"
	"strings"
)

func main() {
	resp, err := http.Get("http://www.baidu.com")
	if err != nil {
		fmt.Println("http get error.")
	}
	defer resp.Body.Close()
	body, err := ioutil.ReadAll(resp.Body)
	if err != nil {
		fmt.Println("http read error")
		return
	}

	src := string(body)

	//将HTML标签全转换成小写
	re, _ := regexp.Compile("\\<[\\S\\s]+?\\>")
	src = re.ReplaceAllStringFunc(src, strings.ToLower)

	//去除STYLE
	re, _ = regexp.Compile("\\<style[\\S\\s]+?\\</style\\>")
	src = re.ReplaceAllString(src, "")

	//去除SCRIPT
	re, _ = regexp.Compile("\\<script[\\S\\s]+?\\</script\\>")
	src = re.ReplaceAllString(src, "")

	//去除所有尖括号内的HTML代码，并换成换行符
	re, _ = regexp.Compile("\\<[\\S\\s]+?\\>")
	src = re.ReplaceAllString(src, "\n")

	//去除连续的换行符
	re, _ = regexp.Compile("\\s{2,}")
	src = re.ReplaceAllString(src, "\n")

	fmt.Println(strings.TrimSpace(src))
}
output==>
百度一下，你就知道
输入法
手写
拼音
关闭
百度首页
设置
登录
糯米
新闻
hao123
地图
视频
贴吧
登录
设置
更多产品
网页
新闻
贴吧
知道
音乐
图片
视频
地图
文库
更多»
手机百度
快人一步
百度糯米
一元大餐
把百度设为主页
把百度设为主页
关于百度
About&nbsp;&nbsp;Baidu
&copy;2016&nbsp;Baidu&nbsp;
使用百度前必读
&nbsp;
意见反馈
&nbsp;京ICP证030173号&nbsp;
京公网安备11000002000001号
</pre>
<pre>
package main

import (
    "fmt"
    "regexp"
)

func main() {
    a := "I am learning Go language"

    re, _ := regexp.Compile("[a-z]{2,4}")

    //查找符合正则的第一个
    one := re.Find([]byte(a))
    fmt.Println("Find:", string(one))

    //查找符合正则的所有slice,n小于0表示返回全部符合的字符串，不然就是返回指定的长度
    all := re.FindAll([]byte(a), -1)
    fmt.Println("FindAll", all)

    //查找符合条件的index位置,开始位置和结束位置
    index := re.FindIndex([]byte(a))
    fmt.Println("FindIndex", index)

    //查找符合条件的所有的index位置，n同上
    allindex := re.FindAllIndex([]byte(a), -1)
    fmt.Println("FindAllIndex", allindex)

    re2, _ := regexp.Compile("am(.*)lang(.*)")

    //查找Submatch,返回数组，第一个元素是匹配的全部元素，第二个元素是第一个()里面的，第三个是第二个()里面的
    //下面的输出第一个元素是"am learning Go language"
    //第二个元素是" learning Go "，注意包含空格的输出
    //第三个元素是"uage"
    submatch := re2.FindSubmatch([]byte(a))
    fmt.Println("FindSubmatch", submatch)
    for _, v := range submatch {
        fmt.Println(string(v))
    }

    //定义和上面的FindIndex一样
    submatchindex := re2.FindSubmatchIndex([]byte(a))
    fmt.Println(submatchindex)

    //FindAllSubmatch,查找所有符合条件的子匹配
    submatchall := re2.FindAllSubmatch([]byte(a), -1)
    fmt.Println(submatchall)

    //FindAllSubmatchIndex,查找所有字匹配的index
    submatchallindex := re2.FindAllSubmatchIndex([]byte(a), -1)
    fmt.Println(submatchallindex)
}
output==>
Find: am
FindAll [[97 109] [108 101 97 114] [110 105 110 103] [108 97 110 103] [117 97 103 101]]
FindIndex [2 4]
FindAllIndex [[2 4] [5 9] [9 13] [17 21] [21 25]]
FindSubmatch [[97 109 32 108 101 97 114 110 105 110 103 32 71 111 32 108 97 110 103 117 97 103 101] [32 108 101 97 114 110 105 110 103 32 71 111 32] [117 97 103 101]]
am learning Go language
 learning Go 
uage
[2 25 4 17 21 25]
[[[97 109 32 108 101 97 114 110 105 110 103 32 71 111 32 108 97 110 103 117 97 103 101] [32 108 101 97 114 110 105 110 103 32 71 111 32] [117 97 103 101]]]
[[2 25 4 17 21 25]]
</pre>
###sync.WaitGroup
<pre>
package main

import (
	"fmt"
	"sync"
)

/*
sync.WaitGroup
sync包中的WaitGroup实现了一个类似任务队列的结构，
你可以向队列中加入任务，任务完成后就把任务从队列中移除，
如果队列中的任务没有全部完成，队列就会触发阻塞以阻止程序继续运行，
具体用法参考如下代码：
*/
var waitgroup sync.WaitGroup

func Afunction(shownum int) {
	fmt.Println(shownum)
	waitgroup.Done() //任务完成，将任务队列中的任务数量-1，其实.Done就是.Add(-1)
}

func main() {
	for i := 0; i < 10; i++ {
		waitgroup.Add(1) //每创建一个goroutine，就把任务队列中任务的数量+1
		go Afunction(i)
	}
	waitgroup.Wait() //.Wait()这里会发生阻塞，直到队列中所有的任务结束就会解除阻塞
}
output==>
9
0
1
2
3
4
5
6
7
8
</pre>
### Socket编程
https://github.com/astaxie/build-web-application-with-golang/blob/master/zh/08.1.md
