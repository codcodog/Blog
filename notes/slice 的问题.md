slice 的问题
=============

### 场景

```golang
package main

import "fmt"

type cities []string

func main() {
	Nepal := cities{"Kathmandu", "Pokhara", "Lumbini"}
	Nepal.add()
	Nepal.print()
}

func (c cities) print() {
	for i, city := range c {
		fmt.Println(i, city)
	}
}

func (c cities) add() {
	c = append(c, "hello")
}
```

执行输出如下：
```
0 Kathmandu
1 Pokhara
2 Lumbini
```
结果没有 `3 hello`

### 原因

`slice` 跟 `array` 很类似，但是它更灵活，可以自动扩容.  

由源码可以看到它的本质：
```
// runtime/slice.go
type slice struct {
	array unsafe.Pointer // 元素指针
	len   int // 长度
	cap   int // 容量
}
```

`slice` 有三个属性，分别是指向底层数组的指针，切片可用的元素个数，底层数组的元素个数.
> 底层的数组可能被多个 `slice` 指向，所以修改其中一个 `slice` 的时候，会因此影响到其他的 `slice`.

上面的代码 `c = append(c, "hello")` 之后，遍历 `c` 却没有 `hello`，这主要涉及到 `slice` 的扩容问题.

在添加元素的时候，如果 `slice` 的容量不够的时候，是会先读取原有元素，然后 cap 扩大一倍，再重新复制到新的数组中去，`slice` 的数组指针
也更新为新的数组指针.

把上面代码 `c` 的指针地址打印下，就清晰很多了.
```golang
package main

import "fmt"

type cities []string

func main() {
	Nepal := cities{"Kathmandu", "Pokhara", "Lumbini"}
	Nepal.add()
	Nepal.print()
}

func (c cities) print() {
	fmt.Printf("print(): %p \n", c)
	for i, city := range c {
		fmt.Println(i, city)
	}
}

func (c cities) add() {
	fmt.Printf("add() append before: %p \n", c)
	c = append(c, "hello")
	fmt.Printf("add() append after: %p \n", c)
}
```

输出如下：
```
add() append before: 0xc000070150 # append 前后地址不一样
add() append after: 0xc00007c120
print(): 0xc000070150
0 Kathmandu
1 Pokhara
2 Lumbini
```

### 解决

在 `add()` 方法的时候，使用指针赋值，直接修改原来的 `slice`.
```golang
func (c *cities) add() {
	*c = append(*c, "hello")
}
```

输出如下：
```
0 Kathmandu
1 Pokhara
2 Lumbini
3 hello
```

### 参考
[how-to-append-to-a-slice-pointer-receiver](https://stackoverflow.com/questions/36854408/how-to-append-to-a-slice-pointer-receiver/36854446)  
[深度解密Go语言之Slice](https://www.cnblogs.com/qcrao-2018/p/10631989.html)
