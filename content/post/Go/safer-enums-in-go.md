---
title: "Safer Enums in Go"
date: 2021-12-11T19:50:31+08:00
draft: false
tags: [""]
categories: ["Go"]
---



此文为 [Safer Enums in Go](https://threedots.tech/post/safer-enums-in-go/) 译文

枚举是 Web 应用程序的重要组成部分。Go 并没有开箱即用地支持它们，但有一些方法可以模拟它们。

许多显而易见的解决方案远非理想。以下是我们使用的一些想法，它们通过设计使枚举更安全。

## iota

Go 中可以使用 `iota` 枚举。

```go
const (
	Guest = iota
	Member
	Moderator
	Admin
)
```



`iota` 适用于由 2 的幂表示的标志。

```go
const (
	Guest = 1 << iota // 1
	Member            // 2
	Moderator         // 4
	Admin             // 8
)

// ...

user.Roles = Member | Moderator // 6
```

位掩码很有效，有时也很有帮助。但是，它与大多数 Web 应用程序中的枚举不同。通常，您可以将所有角色存储在列表中。它也将更具可读性。



iota的主要问题是它在整数上工作，没有防止传递无效的值。

```go
func CreateUser(role int) error {
	fmt.Println("Creating user with role", role)
	return nil
}

func main() {
	err := CreateUser(-1)
	if err != nil {
		fmt.Println(err)
	}
	
	err = CreateUser(42)
	if err != nil {
		fmt.Println(err)
	}
}
```

即使没有相应的角色，CreateUser 也会很乐意接受 -1 或 42。



当然，我们可以在函数中验证这一点。但是我们使用具有强类型的语言，所以让我们利用它。在我们的应用程序上下文中，用户角色不仅仅是一个模糊的数字。

> 反模式：整数枚举 
>
> 不要使用基于 iota 的整数来表示不是连续数字或标志的枚举。

我们可以引入一种类型来改进解决方案。

```go
type Role uint

const (
	Guest Role = iota
	Member
	Moderator
	Admin
)
```

它看起来更好，但仍然可以传递任意整数来代替 Role。 Go 编译器在这里没有帮助我们。

```go
func CreateUser(role Role) error {
	fmt.Println("Creating user with role", role)
	return nil
}

func main () {
	err := CreateUser(0)
	if err != nil {
		fmt.Println(err)
	}
	
    err = CreateUser(role.Role(42))
    if err != nil {
        fmt.Println(err)
    }
}
```

该类型是对裸整数的改进，但它仍然是一种错觉。它不给我们任何保证该角色是有效的。



## Sentinel values

因为 iota 从 0 开始，Guest 也是 Role 的零值。这使得很难检测角色是否为空或有人传递了 Guest 值。

您可以通过从 1 开始计数来避免这种情况。更好的是，保留一个可以比较且不会误认为实际角色的明确标记值。

```go
const (
	Unknown Role = iota
	Guest
	Member
	Moderator
	Admin
)

func CreateUser(r role.Role) error {
	if r == role.Unknown {
		return errors.New("no role provided")
	}
	
	fmt.Println("Creating user with role", r)
	
	return nil
}
```



> 最佳实践： 
>
> 为枚举的零值保留一个显式变量。



## Slugs

枚举似乎是关于连续整数，但它很少是有效的表示。在 Web 应用程序中，我们使用枚举来对某种类型的可能变体进行分组。它们不能很好地映射到数字。

当您在 API 响应、数据库表或日志中看到 3 时，很难理解上下文。您必须检查来源或过时的文档以了解其内容。

在大多数情况下，字符串 slug 比整数更有意义。无论你在哪里看到它，主持人都很明显。由于 iota 无论如何都无济于事，我们也可以使用人类可读的字符串。

```go
type Role string

const (
	Unknown   Role = ""
	Guest     Role = "guest"
	Member    Role = "member"
	Moderator Role = "moderator"
	Admin     Role = "admin"
)
```



> 最佳实践：Slugs 
>
> 使用字符串值而不是整数。 避免使用空格以便于解析和记录。使用camelCase、snake_case 或kebab-case。

Slug 对错误代码特别有用。与 {"error": 4102} 相比，像 {"error": "user-not-found"} 这样的错误响应是显而易见的。

但是，该类型仍然可以包含任意字符串。

```go
err = CreateUser("super-admin")
if err != nil {
	fmt.Println(err)
}
```



## Struct-based Enums

最后的迭代使用结构。它让我们可以使用设计安全的代码。我们不需要检查传递的值是否正确。

```go
type Role struct {
	slug string
}

func (r Role) String() string {
	return r.slug
}

var (
	Unknown   = Role{""}
	Guest     = Role{"guest"}
	Member    = Role{"member"}
	Moderator = Role{"moderator"}
	Admin     = Role{"admin"}
)
```

由于 slug 字段未导出，因此无法从包外部填充它。您可以构建的唯一无效角色是空角色：Role{}。

我们可以添加一个构造函数来创建一个基于 slug 的有效角色：

```go
func FromString(s string) (Role, error) {
	switch s {
	case Guest.slug:
		return Guest, nil
	case Member.slug:
		return Member, nil
	case Moderator.slug:
		return Moderator, nil
	case Admin.slug:
		return Admin, nil
	}

	return Unknown, errors.New("unknown role: " + s)
}
```



> 最佳实践：
>
> 基于结构的枚举 将枚举封装在结构中以获得额外的编译时安全性

这种方法在你处理业务逻辑时是完美的。保持结构在内存中始终处于有效状态，使你的代码更容易操作和理解。检查枚举类型是否为空就足够了，而且你可以确定它是一个正确的值。

这种方法存在一个潜在问题。 Go 中的结构不能是常量，因此可以像这样覆盖全局变量：

```go
roles.Guest = role.Admin
```

但是，没有合理的理由这样做。你更有可能意外地传递一个无效的整数。

另一个缺点是您必须在两个地方进行更新：枚举列表和构造函数。然而，它很容易被发现，即使你一开始错过了它。
