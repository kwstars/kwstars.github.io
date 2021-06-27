---
title: "Golang的环境变量"
date: 2020-10-10T18:01:22+08:00
draft: false
tags: ["Go"]
categories: ["Go"]
---

## 什么是环境变量
环境变量是系统级的键-值对，正在运行的进程可以访问它。这些通常用于使同一程序在不同的部署环境（例如PROD，DEV或TEST）中表现不同。在环境中存储配置是`a twelve-factor app`的原理之一。它使应用程序具有可移植性。

## 为什么要使用环境变量
主要作用是保护敏感信息和环境切换

- 如果您在代码中使用敏感信息，则所有有权访问该代码的未授权用户都将拥有敏感数据，您可能不希望这样做。
- 如果您使用的代码版本控制工具`git`，则可以将DB凭据与代码一起推送，它将公开。
- 如果要在一个地方管理变量，则可以进行任何更改，而不必在应用程序代码中的所有位置都进行更改。
- 您可以管理多个部署环境，例如PROD，DEV或TEST。在部署之间可以轻松更改环境变量，而无需更改任何应用程序代码。

> 永远不要忘记在.gitignore中包含环境变量文件

## 内置OS包
我们可以不需要任何第三方包即可访问Golang中的环境变量，并且可以使用标准os程序包来实现。以下是与环境变量有关的功能及其用途的列表。
-   `os.Setenv()`：设置环境变量。
-   `os.Getenv()`：获取环境变量。
-   `os.Unsetenv()`：删除环境变量，如果我们尝试使用该环境值来获取该环境值`os.Getenv()`，则会返回一个空值。
-   `os.ExpandEnv`：根据环境变量的值替换字符串中的$ {var}或$ var。如果不存在任何环境变量，则将使用空字符串替换它。
-   `os.LookupEnv()`：判断环境变量是否存在。如果系统中不存在该变量，则返回值将为空，并且布尔值将为false。否则，它将返回值（可以为空），并且布尔值为true。

> 如果不存在环境变量，则os.Getenv（）将返回一个空字符串，使用LookupEnv来区分空值和未设置值。

```go
package main

import (
	"fmt"
	"os"
)

func main() {
	// 设置环境变量
	os.Setenv("SITE_TITLE", "Test Site")
	os.Setenv("DB_HOST", "localhost")
	os.Setenv("DB_PORT", "27017")
	os.Setenv("DB_USERNAME", "admin")
	os.Setenv("DB_PASSWORD", "password")
	os.Setenv("DB_NAME", "testdb")

	// 获取环境变量
	host := os.Getenv("SITE_TITLE")
	port := os.Getenv("DB_HOST")
	fmt.Printf("Site Title: %s, Host: %s\n", host, port) // Site Title: Test Site, Host: localhost

	// 删除环境变量
	os.Unsetenv("SITE_TITLE")
	fmt.Printf("After unset, Site Title: %s\n", os.Getenv("SITE_TITLE")) // After unset, Site Title:

	// 检查环境变量是否存在
	redisHost, ok := os.LookupEnv("REDIS_HOST")
	if !ok {
		fmt.Println("REDIS_HOST is not present") // REDIS_HOST is not present
	} else {
		fmt.Printf("Redis Host: %s\n", redisHost)
	}

	// Expand a string containing environment variables in the form of $var or ${var}
	dbURL := os.ExpandEnv("mongodb://${DB_USERNAME}:${DB_PASSWORD}@$DB_HOST:$DB_PORT/$DB_NAME")
	fmt.Println("DB URL: ", dbURL) // DB URL: mongodb://admin:password@localhost:27017/testdb
}
```

还有两个功能os.Clearenv，os.Environ()让我们在单独的程序中使用它们。
- `os.Clearenv`：删除所有环境变量，清理测试环境可能很有用。
- `os.Environ()`：以key = value的形式返回包含所有环境变量的字符串的一部分。

```go
package main

import (
	"fmt"
	"os"
	"strings"
)

func main() {
	// Environ returns a slice of string containing all the environment variables in the form of key=value.
	for _, env := range os.Environ() {
		// env is
		envPair := strings.SplitN(env, "=", 2)
		key := envPair[0]
		value := envPair[1]

		fmt.Printf("%s : %s\n", key, value)
	}

	// Delete all environment variables
	os.Clearenv()

	fmt.Println("Number of environment variables: ", len(os.Environ()))
}
```

上面的函数将列出系统中所有可用的环境变量，包括NAME和DB_HOST。一旦运行os.Clearenv()，它将清除正在运行的进程的所有环境变量。

## godotenv包
Ruby dotenv项目启发了[GoDotEnv](https://github.com/joho/godotenv)包，它从.env文件加载环境变量

让我们创建一个.env文件，其中将具有所有配置。
```bash
# .env file
# This is a sample config file

SITE_TITLE=Test Site

DB_HOST=localhost
DB_PORT=27017
DB_USERNAME=admin
DB_PASSWORD=password
DB_NAME=testdb
```

然后在main.go文件中，我们将使用godotenv加载环境变量。
```go
// main.go
package main

import (
	"fmt"
	"log"
	"os"

	"github.com/joho/godotenv"
)

func main() {

	// load .env file from given path
	// we keep it empty it will load .env from current directory
	err := godotenv.Load(".env")

	if err != nil {
		log.Fatalf("Error loading .env file")
	}

	// getting env variables SITE_TITLE and DB_HOST
	siteTitle := os.Getenv("SITE_TITLE")
	dbHost := os.Getenv("DB_HOST")

	fmt.Printf("godotenv : %s = %s \n", "Site Title", siteTitle) // godotenv : Site Title = Test Site
	fmt.Printf("godotenv : %s = %s \n", "DB Host", dbHost)       // godotenv : DB Host = localhost
}
```

> 我们也可以一次加载多个env文件。godotenv还支持YAML。


## Viper 包
> Viper是包括12要素应用程序在内的Go应用程序的完整配置解决方案。它旨在在应用程序中工作，并且可以处理所有类型的配置需求和格式。

[Viper](https://github.com/spf13/viper)支持多种文件格式来加载环境变量，例如，从JSON，TOML，YAML，HCL，envfile和Java属性配置文件中读取。因此，在此示例中，我们将研究如何从YAML文件中加载环境变量。

> YAML是一种人类可读的数据序列化语言。它通常用于配置文件和用于存储或传输数据的应用程序。

让我们在一个空文件夹中创建config.yaml和main.go。
```yaml
# config.yaml
# config.yaml
SITE:
    TITLE: Test Site

DB:
  PORT: "27017"
  USERNAME: "admin"
  PASWORD: "password"
  NAME: "testdb"
```

在下面的代码中，我们使用Viper从config.yaml中加载环境变量。我们可以从所需的任何路径加载配置文件。如果配置文件中没有任何环境变量，我们还可以为任何环境变量设置默认值。

```go
// main.go
package main

import (
	"fmt"
	"log"

	"github.com/spf13/viper"
)

func main() {
	// Set the file name of the configurations file
	viper.SetConfigName("config")

	// Set the path to look for the configurations file
	viper.AddConfigPath(".")

	// Enable VIPER to read Environment Variables
	viper.AutomaticEnv()

	viper.SetConfigType("yml")

	if err := viper.ReadInConfig(); err != nil {
		fmt.Printf("Error reading config file, %s", err)
	}

	// Set undefined variables
	viper.SetDefault("DB.HOST", "127.0.0.1")

	// getting env variables DB.PORT
	// viper.Get() returns an empty interface{}
	// so we have to do the type assertion, to get the value
	DBPort, ok := viper.Get("DB.PORT").(string)

	// if type assert is not valid it will throw an error
	if !ok {
		log.Fatalf("Invalid type assertion")
	}

	fmt.Printf("viper : %s = %s \n", "Database Address", DBPort) // viper : Database Port = 27017
	fmt.Println(viper.GetString("DB.HOST"))                      // 127.0.0.1
}
```

## 结论

使用环境变量是在我们的应用程序中处理配置的绝佳方法。总体而言，它为您提供了易于配置，更好的安全性，多个部署环境和更少的生产错误。

现在您可以在go应用程序中管理环境变量，并且可以在我们的[Github Repo](https://github.com/LoginRadius/engineering-blog-samples/tree/master/GoLang/EnvironmentVariables)上找到本教程中使用的完整代码

---
[Environment variables in Golang](https://www.loginradius.com/engineering/blog/environment-variables-in-golang/)

