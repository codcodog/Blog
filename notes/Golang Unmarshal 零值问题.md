Golang Unmarshal 零值问题
==========================

### 场景

`Golang` 在开发一个 `API` 的时候，常常会把用户的输入绑定到一个对应的结构体上.  

例如，用户在前端输入一个产品信息，数据结构如下:
```go
type ProductInfo struct {
	ID     int    `json:"id"`     // 产品 ID
	Name   string `json:"name"`   // 产品名称
	Status int    `json:"status"` // 产品状态，0：上架，1：下架
}
```

而 `Golang` 在绑定 `json` 到对应的结构体时，若缺少字段，则会自动初始化为「零值」
```go
func main() {
	data := []byte(`{"id": 1, "name": "产品一"}`)
	product := &ProductInfo{}

	err := json.Unmarshal(data, product)
	if err != nil {
		fmt.Println(err)
	}
	fmt.Printf("%+v \n", product)  // 输出：&{ID:1 Name:产品一 Status:0}
}
```

这时候，用户没有输入 `status`，但绑定结构体的时候自动初始化为「零值」了  

从而导致用户「没有输入 `status`」 和「输入 `status = 0`」的场景重合了.

### 如何解决

设置 `Status` 数据类型为指针类型 `*int`，则初始化「零值」的时候为 `nil`
```go
type ProductInfo struct {
	ID     int    `json:"id"`     // 产品 ID
	Name   string `json:"name"`   // 产品名称
	Status *int   `json:"status"` // 产品状态，0：上架，1：下架
}

func main() {
	data := []byte(`{"id": 1, "name": "产品一"}`)
	product := &ProductInfo{}

	err := json.Unmarshal(data, product)
	if err != nil {
		fmt.Println(err)
	}
	fmt.Printf("%+v \n", product) // 输出：&{ID:1 Name:产品一 Status:<nil>}
}

```

或者，在创建结构体时，提前给 `Status` 赋值一个默认值
```go
type ProductInfo struct {
	ID     int    `json:"id"`     // 产品 ID
	Name   string `json:"name"`   // 产品名称
	Status int    `json:"status"` // 产品状态，0：上架，1：下架
}

func main() {
	data := []byte(`{"id": 1, "name": "产品一"}`)
	product := &ProductInfo{Status: -1}

	err := json.Unmarshal(data, product)
	if err != nil {
		fmt.Println(err)
	}
	fmt.Printf("%+v \n", product)  // 输出：&{ID:1 Name:产品一 Status:-1}
}
```

判断 `status = -1` 来区分用户是否输入了 `status`.
