


### 一、类型系统
* 基本类型（basic type）
    * 内置字符串类型：string.  
    * 内置布尔类型：bool.
    * 内置数值类型：  
        * int8、uint8（byte）、int16、uint16、int32（rune）、uint32、int64、uint64、int、uint、uintptr。
        * float32、float64。
        * complex64、complex128。  
`注意，byte是uint8的一个内置别名，rune是int32的一个内置别名。 下面将要提到如何声明自定义的类型别名。`
* 组合类型（composite type）  
    * 指针类型 - 类C指针
    * 结构体类型 - 类C结构体
    * 函数类型 - 函数类型在Go中是一种一等公民类别
    * 容器类型，包括:
        * 数组类型 - 定长容器类型
        * 切片类型 - 动态长度和容量容器类型
        * 映射类型（map）- 也常称为字典类型。在标准编译器中映射是使用哈希表实现的。
    * 通道类型 - 通道用来同步并发的协程
    * 接口类型 - 接口在反射和多态中发挥着重要角色

参考:  
https://gfw.go101.org/article/type-system-overview.html

### 二、数据类型  
* 数组  
    * 概述  
        * 数组是值类型，所有的值类型变量在赋值和作为参数传递是都将产生一次复制动作。   
        * 数组的长度在定义之后无法再次修改

    * 定义  
        ```
        [32]byte // 长度为32的数组，每个元素为一个字节  
        [2*N] struct { x, y int32 } // 复杂类型数组  
        [1000]*float64 // 指针数组  
        [3][5]int // 二维数组  
        [2][2][2]float64 // 等同于`[2]([2]([2]float64))`
        ```
    * 获取长度  
        arrLength := len(arr)
    * 遍历  
        ```
        for i := 0; i < len(array); i++ { 
            fmt.Println("Element", i, "of array is", array[i]) 
        } 
        ```
        or
        ```
        for i, v := range array { 
            fmt.Println("Array element[", i, "]=", v) 
        } 
        ```
    * 数据切片  
        * myArray[first:last]
        ```
        // 先定义一个数组
        var myArray [10]int = [10]int{1, 2, 3, 4, 5, 6, 7, 8, 9, 10} 
        // 基于数组创建一个数组切片
        var mySlice []int = myArray[:5] 
        ```
        * make()直接创建
        ```
        创建一个初始元素个数为5的数组切片，元素初始值为0：
        mySlice1 := make([]int, 5)  
        创建一个初始元素个数为5的数组切片，元素初始值为0，并预留10个元素的存储空间：
        mySlice2 := make([]int, 5, 10) 
        直接创建并初始化包含5个元素的数组切片：
        mySlice3 := []int{1, 2, 3, 4, 5} 
        当然，事实上还会有一个匿名数组被创建出来，只是不需要我们来操心而已。
        ```
        * cap()返回数组切片分配的空间大小  
        * len()返回数组切片中当前所存储的元素个数
        * append()追加元素
        ```
        //从尾端给mySlice加上3个元素
        mySlice = append(mySlice, 1, 2, 3) 

        mySlice2 := []int{8, 9, 10} 
        // 给mySlice后面添加另一个数组切片
        mySlice = append(mySlice, mySlice2...)
        ```
        `注意: 在第二个参数mySlice2后面加了三个点，即一个省略号，如果没有这个省
        略号的话，会有编译错误，因为按append()的语义，从第二个参数起的所有参数都是待附加的
        元素。因为mySlice中的元素类型为int，所以直接传递mySlice2是行不通的。加上省略号相
        当于把mySlice2包含的所有元素打散后传入。`
        * copy()
        ```
        slice1 := []int{1, 2, 3, 4, 5} 
        slice2 := []int{5, 4, 3} 
        copy(slice2, slice1) // 只会复制slice1的前3个元素到slice2中
        copy(slice1, slice2) // 只会复制slice2的3个元素到slice1的前3个位置
        ```
* map
    ```
    package main 
    import "fmt" 
    // PersonInfo是一个包含个人详细信息的类型
    type PersonInfo struct { 
        ID string
        Name string
        Address string
    } 
    func main() { 
        var personDB map[string] PersonInfo 
        personDB = make(map[string] PersonInfo) 
        // 往这个map里插入几条数据
        personDB["12345"] = PersonInfo{"12345", "Tom", "Room 203,..."} 
        personDB["1"] = PersonInfo{"1", "Jack", "Room 101,..."} 
        // 从这个map查找键为"1234"的信息
        person, ok := personDB["1234"] 
        // ok是一个返回的bool型，返回true表示找到了对应的数据
        if ok { 
            fmt.Println("Found person", person.Name, "with ID 1234.") 
        } else { 
            fmt.Println("Did not find person with ID 1234.") 
        }
        // 元素删除  
        delete(personDB, "12345")
    }
    ```


* 结构体
    * 概述  

    * 定义  
    ```
    type struct_variable_type struct {  
        member definition;  
        member definition;  
        ...  
        member definition;  
    }
    ```
    ```
    type Books struct {
        title string
        author string
        subject string
        book_id int
    }
    // 创建一个新的结构体
    b1 := Books{"Go 语言", "www.runoob.com", "Go 语言教程", 6495407}

    // 也可以使用 key => value 格式
    b2 := Books{title: "Go 语言", author: "www.runoob.com", subject: "Go 语言教程", book_id: 6495407}
    ```

    * 可见性  
        Go语言对关键字的增加非常吝啬，其中没有private、protected、public这样的关键字。
        要使某个符号对其他包（package）可见（即可以访问），需要将该符号定义为以大写字母
        开头
        ```
        type Rect struct { 
            X, Y float64
            Width, Height float64
        } 
        ```
        成员方法的可访问性遵循同样的规则，例如：
        ```
        func (r *Rect) area() float64 { 
            return r.Width * r.Height 
        } 
        ```

        `注意:Rect的area()方法只能在该类型所在的包内使用。`     
        `Go语言中符号的可访问性是包一级的而不是类型一级的。在上面的例子中，尽管area()是Rect的内部方法，但同一个包中的其他类型也都可以访问到它。这样的可访问性控制很粗旷，很特别，但是非常实用。如果Go语言符号的可访问性是类型一级的，少不了还要加上friend这样的关键字，以表示两个类是朋友关系，可以访问彼此的私有成员。`


    * 标签(tag)

* 方法   
    * 声明  
        `func (t T) functionName([params type]) [return type]`  
        `func (t *T) functionName([params type]) [return type]`
        * T必须是一个定义类型；
        * T必须和此方法声明定义在同一个代码包中；
        * T不能是一个指针类型；
        * T不能是一个接口类型。

        ```
        // Age和int是两个不同的类型。我们不能为int和*int
        // 类型声明方法，但是可以为Age和*Age类型声明方法。
        type Age int
        func (age Age) LargerThan(a Age) bool {
            return age > a
        }
        func (age *Age) Increase() {
            *age++
        }

        // 为自定义的函数类型FilterFunc声明方法。
        type FilterFunc func(in int) bool
        func (ff FilterFunc) Filte(in int) bool {
            return ff(in)
        }

        // 为自定义的映射类型StringSet声明方法。
        type StringSet map[string]struct{}
        func (ss StringSet) Has(key string) bool {
            _, present := ss[key]
            return present
        }
        func (ss StringSet) Add(key string) {
            ss[key] = struct{}{}
        }
        func (ss StringSet) Remove(key string) {
            delete(ss, key)
        }

        // 为自定义的结构体类型Book和它的指针类型*Book声明方法。
        type Book struct {
            pages int
        }
        func (b Book) Pages() int {
            return b.pages
        }
        func (b *Book) SetPages(pages int) {
            b.pages = pages
        }

        ```



    * 参考  
        https://gfw.go101.org/article/method.html



* 接口
    * 定义接口  
        ```
        type interface_name interface {
            method_name1 [return_type]
            method_name2 [return_type]
            method_name3 [return_type]
            ...
            method_namen [return_type]
        }
        ```

    * 例子
        ```
        package main
        import (
            "fmt"
        )
        type Phone interface {
            call()
        }
        type NokiaPhone struct {
        }

        func (nokiaPhone NokiaPhone) call() {
            fmt.Println("I am Nokia, I can call you!")
        }

        type IPhone struct {
        }

        func (iPhone IPhone) call() {
            fmt.Println("I am iPhone, I can call you!")
        }

        func main() {
            var phone Phone
            phone = new(NokiaPhone)
            phone.call()
            phone = new(IPhone)
            phone.call()
        }
        ```
        `在上面的例子中，我们定义了一个接口Phone，接口里面有一个方法call()。然后我们在main函数里面定义了一个Phone类型变量，并分别为之赋值为NokiaPhone和IPhone。然后调用call()方法。`

* 通道(channel)
    * 概述  
        通道（channel）是用来传递数据的一个数据结构。  
        通道可用于两个 goroutine 之间通过传递一个指定类型的值来同步运行和通讯。操作符 <- 用于指定通道的方向，发送或接收。如果未指定方向，则为双向通道。   

        ```
        ch := make(chan int) // 定义，chan为关键字
        ch <- value  // 写入发送
        value := <-ch // 读取接收
        close(ch) //关闭通道
        ```

    * 例子
        ```
        package main
        import "fmt"

        func sum(s []int, c chan int) {
            sum := 0
            for _, v := range s {
                    sum += v
            }
            c <- sum // 把 sum 发送到通道 c
        }

        func main() {
            s := []int{7, 2, 8, -9, 4, 0}

            c := make(chan int)
            go sum(s[:len(s)/2], c)
            go sum(s[len(s)/2:], c)
            x, y := <-c, <-c // 从通道 c 中接收

            fmt.Println(x, y, x+y)
        }
        ```
        结果:   
        ```
        -5 17 12
        ```

        `注意：默认情况下，通道是不带缓冲区的。发送端发送数据，同时必须又接收端相应的接收数据。`

    * 通道缓冲区  
        ```
        ch := make(chan int, 100) // make 的第二个参数指定缓冲区大小
        ```
        `注意：如果通道不带缓冲，发送方会阻塞直到接收方从通道中接收了值。如果通道带缓冲，发送方则会阻塞直到发送的值被拷贝到缓冲区内；如果缓冲区已满，则意味着需要等待直到某个接收方获取到一个值。接收方在有值可以接收之前会一直阻塞。`

    * 例子  
        ```
        package main
        import "fmt"

        func main() {
            // 这里我们定义了一个可以存储整数类型的带缓冲通道
            // 缓冲区大小为2
            ch := make(chan int, 2)

            // 因为 ch 是带缓冲的通道，我们可以同时发送两个数据
            // 而不用立刻需要去同步读取数据
            ch <- 1
            ch <- 2

            // 获取这两个数据
            fmt.Println(<-ch)
            fmt.Println(<-ch)
        }
        ```
    * 遍历通道与关闭通道  
    ```
        package main
        import (
            "fmt"
        )

        func fibonacci(n int, c chan int) {
            x, y := 0, 1
            for i := 0; i < n; i++ {
                    c <- x
                    x, y = y, x+y
            }
            close(c)
        }

        func main() {
            c := make(chan int, 10)
            go fibonacci(cap(c), c)
            // range 函数遍历每个从通道接收到的数据，因为 c 在发送完 10 个
            // 数据之后就关闭了通道，所以这里我们 range 函数在接收到 10 个数据
            // 之后就结束了。如果上面的 c 通道不关闭，那么 range 函数就不
            // 会结束，从而在接收第 11 个数据的时候就阻塞了。
            for i := range c {
                    fmt.Println(i)
            }
        }
    ```



    

参考:  
[Go语言101](https://gfw.go101.org/article/101.html)