---
description: 配置文件解析库/管理工具
---

# viper的使用

## 一、viper的介绍

[viper](https://github.com/spf13/viper) 配置管理解析库，是由大神 [Steve Francia](https://github.com/spf13) 开发，他在google领导着 [golang](https://github.com/golang) 的产品开发，他也是 [gohugo.io](https://github.com/gohugoio) 的创始人之一，命令行解析库 [cobra](https://github.com/spf13/cobra) 开发者。总之，他在golang领域是专家，很牛的一个人。

他的github地址：[https://github.com/spf13](https://github.com/spf13)

viper是一个配置管理的解决方案，它能够从 json，toml，ini，yaml，hcl，env 等多种格式文件中，读取配置内容，它还能从一些远程配置中心读取配置文件，如consul，etcd等；它还能够监听文件的内容变化。

## 二、viper功能介绍

* 读取 json，toml，ini，yaml，hcl，env 等格式的文件内容
* 读取远程配置文件，如 consul，etcd 等和监控配置文件变化
* 读取命令行 flag 的值
* 从 buffer 中读取值

配置文件又可以分为不同的环境，比如dev，test，prod等。

[viper](https://github.com/spf13/viper) 可以帮助你专注配置文件管理。

[viper](https://github.com/spf13/viper) 读取配置文件的优先顺序，从高到低，如下：

* 显式设置的Set函数
* 命令行参数
* 环境变量
* 配置文件
* 远程k-v 存储系统，如consul，etcd等
* 默认值

> Viper 配置key是不区分大小写的。

其实，上面的每一种文件格式，都有一些比较有名的解析库，如：

* toml ：[https://github.com/BurntSushi/toml](https://github.com/BurntSushi/toml)
* json ：json的解析库比较多，下面列出几个常用的
  * [https://github.com/json-iterator/go](https://www.cnblogs.com/jiujuan/p/github.com/json-iterator/go)
  * [https://github.com/mailru/easyjson](https://github.com/mailru/easyjson)
  * [https://github.com/bitly/go-simplejson](https://github.com/bitly/go-simplejson)
  * [https://github.com/tidwall/gjson](https://github.com/tidwall/gjson)
* ini : [https://github.com/go-ini/ini](https://github.com/go-ini/ini)\
  等等单独文件格式解析库。

但是为啥子要用viper，因为它是一个综合文件解析库，包含了上面所有的文件格式解析，是一个集合体，少了配置多个库的烦恼。

## 三、viper使用

**安装viper命令：**\
`go get github.com/spf13/viper`

> 文档: [https://github.com/spf13/viper/blob/master/README.md#putting-values-into-viper](https://github.com/spf13/viper/blob/master/README.md#putting-values-into-viper)

### 3.1通过viper.Set设置值

如果某个键通过viper.Set设置了值，那么这个值读取的优先级最高

```go
viper.Set("mysql.info", "this is mysql info")
```

### 3.2  设置默认值

> [https://github.com/spf13/viper/blob/master/README.md#establishing-defaults](https://github.com/spf13/viper/blob/master/README.md#establishing-defaults)

viper 支持默认值的设置。如果配置文件、环境变量、远程配置中没有设置键值，就可以通过viper设置一些默认值。

Examples：

```go
viper.SetDefault("ContentDir", "content")
viper.SetDefault("LayoutDir", "layouts")
viper.SetDefault("Taxonomies", map[string]string{"tag": "tags", "category": "categories"})
```

### 3.3 读取配置文件

**读取配置文件要求**：最少要知道从哪个位置查找配置文件。用户一定要设置这个路径。

viper可以从多个路径搜索配置文件，单个viper实例只支持单个配置文件。\
viper本身没有设置默认的搜索路径，需要用户自己设置默认路径。

**viper搜索和读取配置文件例子片段**：

```go
Copyviper.SetConfigName("config") // 配置文件的文件名，没有扩展名，如 .yaml, .toml 这样的扩展名
viper.SetConfigType("yaml")  // 设置扩展名。在这里设置文件的扩展名。另外，如果配置文件的名称没有扩展名，则需要配置这个选项
viper.AddConfigPath("/etc/appname/") // 查找配置文件所在路径
viper.AddConfigPath("$HOME/.appname") // 多次调用AddConfigPath，可以添加多个搜索路径
viper.AddConfigPath(".")             // 还可以在工作目录中搜索配置文件
err := viper.ReadInConfig()       // 搜索并读取配置文件
if err != nil { // 处理错误
  panic(fmt.Errorf("Fatal error config file: %s \n", err))
}
```

> 说明：\
> 这里执行viper.ReadInConfig()之后，viper才能确定到底用哪个文件，viper按照上面的AddConfigPath() 进行搜索，找到第一个名为 config.**ext** (**这里的ext代表扩展名**： 如 json,toml,yaml,yml,ini,prop 等扩展名) 的文件后即停止搜索。

> 如果有多个名称为config的配置文件，viper怎么搜索呢？它会按照如下顺序搜索
>
> * config.json
> * config.toml
> * config.yaml
> * config.yml
> * config.properties (这种一般是java中的配置文件名)
> * config.props (这种一般是java中的配置文件名)

你还可以处理一些特殊情况：

```go
if err := viper.ReadInConfig(); err != nil {
    if _, ok := err.(viper.ConfigFileNotFoundError); ok {        
        // 配置文件没有找到; 如果需要可以忽略
    } else {        
        // 查找到了配置文件但是产生了其它的错误
    }
}

// 查找到配置文件并解析成功
```

### 3.4 例子 1.读取配置文件

**config.yaml**

```yaml
mysql:
    host: 192.168.1.0
    password: root123
    port: 3306
    username: root123
redis:
    host: 127.0.0.1
    port: 4405
```

**viper\_yaml.go**

```go
type Config struct {
	Redis RedisConfig
	Mysql MysqlConfig
}

type RedisConfig struct {
	Host string
	Port int
}
type MysqlConfig struct {
	Host     string
	Password string
	Port     int
	Username string
}

// ViperReadInConfig
func ViperReadInConfig() {
	viper.SetConfigName("config")
	viper.SetConfigType("yaml")
	viper.AddConfigPath(".")

	err := viper.ReadInConfig()
	if err != nil {
		panic(err)
	}
	config := &Config{}

	err = viper.Unmarshal(&config)
	if err != nil {
		return
	}
	fmt.Println("config", config)
}

```

### 3.5 例子 2. 读取多个配置文件

在例子1基础上多增加一个json的配置文件，config3.yaml 配置文件：

**config3.yaml**

```yaml
mysql:
  host: 192.168.1.0
  port: 3306
  username: root
  password: 123456
redis:
  host: 127.0.0.1
  port: 33000
```

**viper\_multi.go**

```go
type Config struct {
	Redis RedisConfig
	Mysql MysqlConfig
}

type RedisConfig struct {
	Host string
	Port int
}
type MysqlConfig struct {
	Host     string
	Password string
	Port     int
	Username string
}

// ViperReadMultiConfig  读取多个配置文件
func ViperReadMultiConfig() {
	var config1 Config
	vyaml1 := viper.New()
	vyaml1.SetConfigName("config")
	vyaml1.SetConfigType("yaml")
	vyaml1.AddConfigPath(".")

	err := vyaml1.ReadInConfig()
	if err != nil {
		return
	}
	err = vyaml1.Unmarshal(&config1)
	if err != nil {
		return
	}
	fmt.Println("config1", config1)

	var config3 Config
	vyaml3 := viper.New()
	vyaml3.SetConfigName("config3")
	vyaml3.SetConfigType("yaml")
	vyaml3.AddConfigPath(".")

	err = vyaml3.ReadInConfig()
	if err != nil {
		fmt.Println(err)
	}
	err = vyaml3.Unmarshal(&config3)
	if err != nil {
		fmt.Println(err)
	}
	fmt.Println("config3", config3)
}
```



### 3.6 例子 3.读取配置项的值

**config.yaml**

```yaml
mysql:
    host: 192.168.1.0
    port: 3306
    username: root123
    password: root123
redis:
    host: 127.0.0.1
    port: 4405

```

**viper\_get.go**

```go
// ViperGet viper.get()的测试
func ViperGet() {
	viper.SetConfigName("config")
	viper.SetConfigType("yaml")
	viper.AddConfigPath(".")
	err := viper.ReadInConfig()
	if err != nil {
		return
	}
	mysql_host := viper.Get("mysql.host")
	fmt.Println("mysql.host", mysql_host)
	mysql_port := viper.GetInt("mysql.port")
	fmt.Println("mysql.port", mysql_port)

	redisMap := viper.GetStringMapString("redis")
	fmt.Println("redis", redisMap)
}
```

### 3.7 例子 4.读取命令行的值

新建文件夹 cmd，然后cmd文件夹里新建config.json文件：

```json
{
  "redis":{
    "port": 3301,
    "host": "127.0.0.1"
  },
  "mysql": {
    "port": 3306,
    "host": "127.0.0.1",
    "username": "root",
    "password": "123456"
  }
}
```

go解析文件，cmd/viper\_pflag.go：

```go
package main

import (
	"fmt"

	"github.com/spf13/pflag"
	"github.com/spf13/viper"
)

func main() {
	pflag.Int("redis.port", 3302, "redis port")

	viper.BindPFlags(pflag.CommandLine)
	pflag.Parse()

	viper.SetConfigName("config")
	viper.SetConfigType("json")
	viper.AddConfigPath(".")
	err := viper.ReadInConfig() //根据上面配置加载文件
	if err != nil {
		fmt.Println(err)
		return
	}

	host := viper.Get("mysql.host")
	username := viper.GetString("mysql.username")
	port := viper.GetInt("mysql.port")
	redisHost := viper.GetString("redis.host")
	redisPort := viper.GetInt("redis.port")

	fmt.Println("mysql - host: ", host, ", username: ", username, ", port: ", port)
	fmt.Println("redis - host: ", redisHost, ", port: ", redisPort)
}
```

**1.不加命令行参数运行：**

> $ go run viper\_pflag.go
>
> mysql - host:  127.0.0.1 , username:  root , port:  3306\
> redis - host:  127.0.0.1 , port:  3301

说明：redis.port 的值是 3301，是 config.json 配置文件里的值。

**2.加命令行参数运行**

> $ go run viper\_pflag.go --redis.port 6666
>
> mysql - host:  127.0.0.1 , username:  root , port:  3306\
> redis - host:  127.0.0.1 , port:  6666

说明：加了命令行参数 `--redis.port 6666`，这时候redis.port输出的值为 `6666`，读取的是cmd命令行的值

### 3.8 例子 5. io.Reader中读取值

[https://github.com/spf13/viper#reading-config-from-ioreader](https://github.com/spf13/viper#reading-config-from-ioreader)\


**viper\_ioreader.go**

```go
package main

import (
	"bytes"
	"fmt"

	"github.com/spf13/viper"
)

func main() {
	viper.SetConfigType("yaml")

	var yaml = []byte(`
Hacker: true
name: steve
hobbies:
- skateboarding
- snowboarding
- go
clothing:
  jacket: leather
  trousers: denim
age: 35
eyes : brown
beard: true
    `)

	err := viper.ReadConfig(bytes.NewBuffer(yaml))
	if err != nil {
		fmt.Println(err)
		return
	}
	hacker := viper.GetBool("Hacker")
	hobbies := viper.GetStringSlice("hobbies")
	jacket := viper.Get("clothing.jacket")
	age := viper.GetInt("age")
	fmt.Println("Hacker: ", hacker, ",hobbies: ", hobbies, ",jacket: ", jacket, ",age: ", age)

}
```



### 3.9 例子 6. 写配置文件

> [https://github.com/spf13/viper#writing-config-files](https://github.com/spf13/viper#writing-config-files)

新建文件 writer/viper\_write\_config.go:

```go
package main

import (
	"fmt"

	"github.com/spf13/viper"
)

func main() {
	viper.SetConfigName("config")
	viper.SetConfigType("yaml")
	viper.AddConfigPath(".")

	viper.Set("yaml", "this is a example of yaml")

	viper.Set("redis.port", 4405)
	viper.Set("redis.host", "127.0.0.1")

	viper.Set("mysql.port", 3306)
	viper.Set("mysql.host", "192.168.1.0")
	viper.Set("mysql.username", "root123")
	viper.Set("mysql.password", "root123")

	if err := viper.WriteConfig(); err != nil {
		fmt.Println(err)
	}
}
```

运行：

> $ go run viper\_write\_config.go

没有任何输出表示生成配置文件成功

```yaml
mysql:
  host: 192.168.1.0
  password: root123
  port: 3306
  username: root123
redis:
  host: 127.0.0.1
  port: 4405
yaml: this is a example of yaml
```
