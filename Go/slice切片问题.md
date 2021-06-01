参考：
	- https://www.zhihu.com/question/27161493

demo
```go
package main
import "fmt"
func tre(){
	a := []int{5}
    a = append(a, 7)
    a = append(a, 9)
    ax := append(a, 11)
    ay := append(a, 12)
   fmt.Println(a, ax, ay)
}
func main(){
	tre()//[5 7 9] [5 7 9 12] [5 7 9 12]
}
```

CTF例子：
	- https://github.com/mwarzynski/confidence2019_teaser_lottery

