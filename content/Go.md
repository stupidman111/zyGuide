# CLI
```shell
# 直接运行
> go run main.go

# 先编译，再运行
> go build main.go
> ./main
```


# 基础语法
## 关键字

## 可见性
* 声明在函数内部，是函数的本地值，类似`private`；
* 声明在函数外部，是对当前包可见（包内所有.go文件都可见）的全局值，类似`protect`；
* 声明在函数外部且首字母大写是所有包可见的全局值，类似`public`；

## 变量声明
* 指定变量类型，声明后若不赋值，使用默认值零值
* 根据变量值自动判定变量类型
* 使用`:=`声明，自动判定变量类型（`:=`左侧的变量名必须是没有声明过的）
* 多变量声明
```go
package main  
  
import "fmt"  
  
/* 指定变量类型，声明后不赋值，使用默认零值 */var V1 int  
var V2 *int  
  
/* 根据值自动判断变量类型 */var V3 = 0  
  
/* 多变量声明 */var (  
    V5 = 5  
    V6 = 6  
)  
  
func main() {  
    fmt.Println(V1) //默认零值为：0  
    fmt.Println(V2) //默认零值为：nil  
  
    fmt.Printf("%T\n", V3) //变量类型自动推断得出为int  
        //使用 `:=` 声明   
V4 := 2.1  
    fmt.Printf("%T\n", V4) //变量类型自动推断得出为float64  
          
fmt.Println(V5);  
    fmt.Println(V6);  
}
```

## 常量声明
> 常量数据类型只能是 布尔型、数字型（整数型、浮点型和复数）和字符串型。

```go
const HOST = "127.0.0.1"  //自动推导类型
const PORT int = 8080   //标明类型

/* 多常量声明 */
const (  
    USER_NAME = "root"  
    PASS_WORD = "root"  
    MAX_CONNECTIONS = 1  
)
```

## iota
