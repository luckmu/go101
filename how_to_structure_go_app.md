# how to structure Go app

[How Do You Structure Your Go Apps - GopherCon Denver 2018](https://www.youtube.com/watch?v=oL6JBUk6tj0)

+ flat structure
+ group by function: [MVC](#mvc-model-view-controller)
+ ~~group by modules:~~
+ group by context: `DDD (Domain-driven design)`

## MVC (Model-view-controller)

~~DAO(Data Access Object)~~

[MVC Components](https://en.wikipedia.org/wiki/Model%E2%80%93view%E2%80%93controller#Components)

+ Model: 管理 app 的数据, 从 Controller 接收 input
+ View: 将 Model 数据以特定形式表示
+ Controller: controller 处理用户输入, 以及 model 数据的交互, 修改。
> The controller responds to the user input and performs interactions on the data model objects.

Model 提供 data 的抽象（object）, 不局限于存储形式（纯文本, 数据库等）, 框架提供与数仓的转换而不用手动造轮子（即实现 object 与数仓结构的映射）。

换数仓只需要变 driver, 上层无感知。

在 mvc 业务级别上, 提供 `CreateUser()`, `DeleteUser()` 这种接口供 Controller 调用。

如 `model/user.go`, 
```go
type User struct {
    ID   int64
    Name string
}

func CreateUser() {}
func DeleteUser(id int64) {}
```
