


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
    




    

