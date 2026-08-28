# 00

## 一个最简单的 Web 服务

创建一个 `main.go` 入口文件，内容如下：

```go
package main

import (
	"net/http"

	"github.com/gin-gonic/gin"
)

func main() {
	// 创建一个包含默认中间件（日志记录和恢复）的 Gin 路由器实例
	router := gin.Default()
	// 定义一个简单的 GET 接口（也叫注册一个路由）
	router.GET("/ping", func(c *gin.Context) {
		// 返回 JSON 响应
		c.JSON(http.StatusOK, gin.H{
			"message": "pong",
		})
	})
	// 在端口 8080（默认）上启动 HTTP 服务器
	router.Run()
}
```

执行 `go run .` 命令运行这段代码。不过在此之前请先执行 `go mod init [module name]`（初始化 Go 模块） 和 `go get`（添加依赖） 命令进行初始化 `go.mod` 和 `go.sum` 文件。

最后浏览器访问 `http://localhost:8080/ping` 就会看到 `{ "message": "pong" }` 的 JSON 响应。

在上面的示例中，我们还用到了 `net/http` 包中的 `StatusOK` 常量。其实也可以直接使用 `200` 来代替 `http.StatusOK`，但是推荐使用常量，因为常量更易读且可维护性更好。

## HTTP 方法

Gin 框架为 GET、POST、PUT、PATCH、DELETE、HEAD 和 OPTIONS 这些 HTTP 请求类型提供了相应的处理方法。

```go
func getting(c *gin.Context) {
	// 返回 JSON 响应
	c.JSON(http.StatusOK, gin.H{"method": "GET"})
}

func posting(c *gin.Context) {
	// 返回 JSON 响应
	c.JSON(http.StatusOK, gin.H{"method": "POST"})
}

func putting(c *gin.Context) {
	// 返回 JSON 响应
	c.JSON(http.StatusOK, gin.H{"method": "PUT"})
}

func patching(c *gin.Context) {
	// 返回 JSON 响应
	c.JSON(http.StatusOK, gin.H{"method": "PATCH"})
}

func deleting(c *gin.Context) {
	// 返回 JSON 响应
	c.JSON(http.StatusOK, gin.H{"method": "DELETE"})
}

func head(c *gin.Context) {
	c.Status(http.StatusOK)
}

func options(c *gin.Context) {
	c.Status(http.StatusOK)
}

func main() {
	// 创建一个包含默认中间件（日志记录和恢复）的 Gin 路由器实例
	router := gin.Default()

	// 定义一个 GET 接口
	router.GET("/someGet", getting)
	// 定义一个 POST 接口
	router.POST("/somePost", posting)
	// 定义一个 PUT 接口
	router.PUT("/somePut", putting)
    // 定义一个 PATCH 接口
	router.PATCH("/somePatch", patching)
	// 定义一个 DELETE 接口
	router.DELETE("/someDelete", deleting)
	// 定义一个 HEAD 接口
	router.HEAD("/someHead", head)
	// 定义一个 OPTIONS 接口
	router.OPTIONS("/someOptions", options)

	// 在端口 8080（默认）上启动 HTTP 服务器
	router.Run()
}
```

在实际的业务开发中，大家默认编写 RESTful 风格的 API。简单地说就是对资源进行增删改查时选择合适的类型：

- 获取资源：HTTP GET
- 创建资源：HTTP POST
- 更新资源：HTTP PUT / PATCH
- 删除资源：HTTP DELETE

回到上面的示例，因为接口已经不只是 GET 类型了，所以不能简单的通过浏览器访问 `http://localhost:8080/somePost` 来进行测试（因为通过浏览器直接访问是发送的 GET 请求，如果访问的接口是其他类型则会返回 404），我们换成在终端执行 `curl` 命令来测试：

```sh
curl -X GET http://localhost:8080/someGet
curl -X POST http://localhost:8080/somePost
curl -X PUT http://localhost:8080/somePut
curl -X PATCH http://localhost:8080/somePatch
curl -X DELETE http://localhost:8080/someDelete
curl -I http://localhost:8080/someHead
curl -X OPTIONS http://localhost:8080/someOptions
```

# 01

## Path 参数

`:` 可以匹配单个路径段：例如 `/user/:name` 匹配 `/user/pany` 但不匹配 `/user/` 或 `/user`。

`*` 可以匹配前缀之后的所有内容：例如 `/user/:name/*action` 匹配 `/user/pany/send` 和 `/user/pany/`。

```go
func main() {
	router := gin.Default()

	router.GET("/user/:name", func(c *gin.Context) {
        // 获取参数 name
		name := c.Param("name")
		c.String(http.StatusOK, "name: %s", name)
	})

	router.GET("/user/:name/*action", func(c *gin.Context) {
        // 获取参数 action
		action := c.Param("action")
		c.String(http.StatusOK, "action: %s", action)
	})

	router.GET("/redirect1", func(c *gin.Context) {
		c.String(http.StatusOK, "访问 /redirect1/ 会重定向到 /redirect1")
	})

	router.GET("/redirect2/", func(c *gin.Context) {
		c.String(http.StatusOK, "访问 /redirect2 会重定向到 /redirect2/")
	})

	router.Run()
}
```

现在继续用 `curl` 命令来测试：

```sh
curl http://localhost:8080/user # 404 page not found
curl http://localhost:8080/user/ # 404 page not found
curl http://localhost:8080/user/pany # name: pany
curl http://localhost:8080/user/pany/ # action: /
curl http://localhost:8080/user/pany/send # action: /send
```

针对上面的示例，还有一个小细节需要我们记住：用浏览器访问 `http://localhost:8080/redirect1/` 会被重定向到 `http://localhost:8080/redirect1`（自动去掉了末尾的 `/` ）；访问 `http://localhost:8080/redirect2` 会被重定向到 `http://localhost:8080/redirect2/`（自动加上了末尾的 `/` ）。

总结一下上面测试结果：

- 如果没有匹配到路径，Gin 会默认返回 `404 page not found`
- 在匹配到路径后可以通过 `c.Param()` 方法取到对应的参数值
- 通配符 `*` 的值始终是从 `/` 开始的剩余路径
- Gin 有时会自动进行重定向（末尾 `/` 会自动增减）

## Query 参数

```go
func main() {
	router := gin.Default()

	router.GET("/user", func(c *gin.Context) {
		id := c.Query("id")                    // Query 获取参数值时，如果参数不存在则返回空字符串
		name := c.DefaultQuery("name", "pany") // DefaultQuery 获取参数值时，如果参数不存在则返回指定的默认值
		c.String(http.StatusOK, "id: %s, name: %s", id, name)
	})

	router.Run()
}
```

这个小节非常简单。用浏览器访问 `http://localhost:8080/user?id=1` 后可以看见接口返回的是 `id: 1, name: pany`。

## Form 参数

> 这一小节默认你了解最最基础的前后端通信知识：[HTTP 协议入门](https://juejin.cn/post/6991345251604660232)。

浏览器原始提交表单数据的两种标准方式：`application/x-www-form-urlencoded` 和 `multipart/form-data`（上传文件）。

注意这里说的是原始提交表单，不是将平时常用的 JSON 数据作为请求体的方式！JSON 数据作为请求体在后续的小节中会讲到。

对这这种提交方式，均可使用下面的方法来获取对应的字段：

- `c.PostForm("key")` 如果字段不存在则返回空字符串
- `c.DefaultPostForm("key", "默认值")` 如果字段不存在则返回指定的默认值

```go
func main() {
	router := gin.Default()

	router.POST("/form_post", func(c *gin.Context) {
		id := c.PostForm("id")
		name := c.DefaultPostForm("name", "pany")
        // 返回 JSON 响应
		c.JSON(http.StatusOK, gin.H{
			"id":   id,
			"name": name,
		})
	})

	router.Run()
}
```

测试：

```sh
# application/x-www-form-urlencoded
curl -X POST http://localhost:8080/form_post -d "id=1"
curl -X POST http://localhost:8080/form_post -d "id=1&name=pany"
# multipart/form-data
curl -X POST http://localhost:8080/form_post -F "id=1"
curl -X POST http://localhost:8080/form_post -F "id=1" -F "name=pany"

# 返回结果都是: {"id":"1","name":"pany"}
```

到这里，我们需要记住：`c.Query() / c.DefaultQuery()` 是从 `URL Query` 读取，`c.PostForm() / c.DefaultPostForm()` 是从请求体读取。如果你想要 Gin 自动检查两个数据源，请改用 `c.ShouldBind()` 配合结构体（后续章节会介绍）。

## QS 参数

在一个遥远的年代...如果前端不使用 `JSON` 传递数据给后端，又想要传递复杂的结构（比如对象），那么就会选择像 `qs` 这类的序列化库，将数据结构拍平后再通过 `Query` 或 `Form` 传递。

在这种情况下，需要将 `c.Query()` 和 `c.PostForm()` 方法换成 `c.QueryMap()` 和 `c.PostFormMap()` 方法：

```go
func main() {
	router := gin.Default()

	router.POST("/post", func(c *gin.Context) {
		user := c.QueryMap("user")
		page := c.PostFormMap("page")

		c.JSON(http.StatusOK, gin.H{
			"user": user,
			"page": page,
		})
	})

	router.Run()
}
```

`c.QueryMap()` 和 `c.PostFormMap()` 这两种方法会将方括号表示法的参数（如 `user[id]=1`）解析为 `map[string]string`。

我们测试一下：

```sh
curl -g -X POST "http://localhost:8080/post?user[id]=1&user[name]=pany" -d "page[offset]=0&page[limit]=10"

# 返回结果: {"page":{"limit":"10","offset":"0"},"user":{"id":"1","name":"pany"}}
```

> 注意，嵌套方括号如 `user[name][nickname]=value` 不会被解析为嵌套 `map`。

但在绝大多数时候，我们都默认传递 `JSON` 数据，所以这一小节的内容并不是一个主流的用法，你只需留一个印象即可。

# 02

## 接收文件

Gin 接收文件需要用到前面 **Form 参数**章节提到的 `multipart/form-data` 方式。

有几个常用的方法来处理前端上传的文件：

- `c.FormFile(key)`：通过字段名从请求中获取单个文件
- `c.MultipartForm()`：解析整个表单，可以访问所有上传的文件和字段值
- `c.SaveUploadedFile(file, dst)`：将接收到的文件保存到磁盘上的目标路径
- `router.MaxMultipartMemory`：设置解析表单数据时可使用的最大内存（默认 32 MiB），超过此限制的文件将存储在磁盘上的临时文件中而不是内存中
- `http.MaxBytesReader`：严格限制上传文件的大小，超出限制时读取器会返回错误，你可以使用 413 状态码进行响应

> 注意，不要信任前端上传的文件名 `file.Filename`。在文件系统操作中使用之前，请始终对其进行清理或替换。使用 `filepath.Base` 去除目录组件以防止路径遍历攻击。

## 接收单文件

先看一个处理单文件的示例：

```go
import (
	"log"
	"net/http"
	"path/filepath"

	"github.com/gin-gonic/gin"
)

func main() {
	router := gin.Default()

	// 设置解析表单数据时可使用的最大内存为 8 MiB
	router.MaxMultipartMemory = 8 << 20

	router.POST("/upload", func(c *gin.Context) {
		// 获取单个文件
		file, err := c.FormFile("file")
		// 错误处理
		if err != nil {
			c.JSON(http.StatusBadRequest, gin.H{
				"error": err.Error(),
			})
			return
		}
		// 后端服务一般不使用 fmt.Println 打印日志，使用 log.Println 更合理
		log.Println(file.Filename)
		// 使用 `filepath.Base` 防止路径遍历攻击
		dst := filepath.Join("./files/", filepath.Base(file.Filename))
		// 将接收到的文件保存到磁盘上的目标路径
		saveErr := c.SaveUploadedFile(file, dst)
		// 错误处理
		if saveErr != nil {
			c.JSON(http.StatusInternalServerError, gin.H{
				"error": saveErr.Error(),
			})
			return
		}

		c.String(http.StatusOK, "%s 上传成功", file.Filename)
	})

	router.Run()
}
```

将下面的 `filepath` 换成真实的文件路径进行测试：

```sh
curl -X POST http://localhost:8080/upload -F "file=@filepath" -H "Content-Type: multipart/form-data"
```

## 接收多文件

再看一个处理多文件的示例：

```go
func main() {
	router := gin.Default()

	router.MaxMultipartMemory = 8 << 20

	router.POST("/upload", func(c *gin.Context) {
		// 解析整个表单
		form, err := c.MultipartForm()
		if err != nil {
			c.JSON(http.StatusBadRequest, gin.H{
				"error": err.Error(),
			})
			return
		}
		// 获取多个文件
		files := form.File["files"]

		for _, file := range files {
			log.Println(file.Filename)
			dst := filepath.Join("./files/", filepath.Base(file.Filename))
			c.SaveUploadedFile(file, dst)
		}
		c.String(http.StatusOK, "%d 个文件上传成功", len(files))
	})

	router.Run()
}
```

将下面的 `filepath1 / filepath2` 换成真实的文件路径进行测试：

```sh
curl -X POST http://localhost:8080/upload \
  -F "files=@filepath1" \
  -F "files=@filepath2" \
  -H "Content-Type: multipart/form-data"
```

## 限制文件大小

会稍微复杂一点，但是阅读完前面章节的内容之后，这里的示例应该不是问题：

```go
const (
	MaxUploadSize = 1 << 20 // 1 MiB
)

func uploadHandler(c *gin.Context) {
	// 使用 http.MaxBytesReader 封装 c.Request.Body，使其仅允许读取 MaxUploadSize 字节的数据
	c.Request.Body = http.MaxBytesReader(c.Writer, c.Request.Body, MaxUploadSize)
    // c.Request.ParseMultipartForm 触发读取
	if err := c.Request.ParseMultipartForm(MaxUploadSize); err != nil {
        // 检查 *http.MaxBytesError
		if _, ok := err.(*http.MaxBytesError); ok {
            // 返回 413 状态码
			c.JSON(http.StatusRequestEntityTooLarge, gin.H{
				"error": fmt.Sprintf("文件超过最大 %d bytes 限制", MaxUploadSize),
			})
			return
		}
		c.JSON(http.StatusBadRequest, gin.H{
			"error": err.Error(),
		})
		return
	}

	file, _, err := c.Request.FormFile("file")
	if err != nil {
		c.JSON(http.StatusBadRequest, gin.H{
			"error": err.Error(),
		})
		return
	}
	defer file.Close()

	c.JSON(http.StatusOK, gin.H{
		"message": "上传成功",
	})
}

func main() {
	router := gin.Default()
	router.POST("/upload", uploadHandler)
	router.Run()
}
```

# 03

## 路由分组

使用 `router.Group()` 可以设置公共路径前缀、进行 API 版本管理，将相关的处理函数放到一起：

```go
func loginEndpoint(c *gin.Context) {
	c.JSON(http.StatusOK, gin.H{"action": "login"})
}

func captchaEndpoint(c *gin.Context) {
	c.JSON(http.StatusOK, gin.H{"action": "captcha"})
}

func main() {
	router := gin.Default()

	api := router.Group("/api") // 公共路径前缀
	{
		v1 := api.Group("/v1") // 版本管理: v1 版本
		{
			auth := v1.Group("/auth") // 认证相关接口
			{
				auth.POST("/login", loginEndpoint)    // /api/v1/auth/login
				auth.GET("/captcha", captchaEndpoint) // /api/v1/auth/captcha
			}
		}
		v2 := api.Group("/v2") // 版本管理: v2 版本
		{
			auth := v2.Group("/auth") // 认证相关接口
			{
				auth.POST("/login", loginEndpoint)    // /api/v2/auth/login
				auth.GET("/captcha", captchaEndpoint) // /api/v2/auth/captcha
			}
		}
	}

	router.Run()
}
```

还可以让整个组**共享中间件**：

```go
// 写一个认证中间件
func AuthRequired() gin.HandlerFunc {
	return func(c *gin.Context) {
		// TODO 认证逻辑
		c.Next()
	}
}

func main() {
	router := gin.Default()

	// 不需要认证的组
	public := router.Group("/api")
	{
		public.POST("/captcha", func(c *gin.Context) {
			// TODO
		})
	}

	// 需要认证的组
	private := router.Group("/api", AuthRequired()) // 也可以调用 private.Use(AuthRequired()) 来添加中间件
	{
		private.GET("/profile", func(c *gin.Context) {
			// TODO
		})
	}

	router.Run()
}
```

这样，添加了中间件的 `Group` 里面的所有接口都会先执行中间件，再执行处理函数。

## 重定向

还是得稍微回忆一下 HTTP 状态码中与重定向相关的：

- `301`：永久重定向，资源已永久移动，浏览器和搜索引擎会更新它们的缓存。对应常量为 `http.StatusMovedPermanently`
- `302`：临时重定向，资源被临时改变了位置，浏览器会跟随但不会缓存新 URL。对应常量为 `http.StatusFound`
- `307`：临时重定向，与 302 区别是浏览器必须保留原始 HTTP 方法。对应常量为 `http.StatusTemporaryRedirect`
- `308`：永久重定向，与 301 区别是浏览器必须保留原始 HTTP 方法。对应常量为 `http.StatusPermanentRedirect`

其中 301 和 302 不可靠，遇见非 GET 请求时可能被浏览器改写成 GET 请求。所以当请求是 POST 时优先考虑 307 和 308 状态码。

### HTTP 重定向

服务端返回 3xx 状态码 + location，浏览器自动再发一次请求（有网络往返）。使用 `c.Redirect(code, location)` 方法来完成。

```go
import (
	"io"
	"net/http"

	"github.com/gin-gonic/gin"
)

func main() {
	router := gin.Default()

	// 跨站重定向
	router.GET("/baidu", func(c *gin.Context) {
		c.Redirect(http.StatusMovedPermanently, "https://www.baidu.com/")
	})

	// 接口版本迁移
	router.POST("/api/v1/captcha", func(c *gin.Context) {
		c.Redirect(http.StatusTemporaryRedirect, "/api/v2/captcha")
	})

	router.POST("/api/v2/captcha", func(c *gin.Context) {
        body, _ := io.ReadAll(c.Request.Body)
		c.JSON(http.StatusOK, gin.H{"version": "v2", "body": string(body)})
	})

	router.Run()
}
```

### 路由重定向

也叫服务器内部转发，同一次请求内改写 Path 后重新走一遍路由树（无网络往返，浏览器感知不到）。使用 `router.HandleContext(c)` 方法来完成。

```go
func main() {
	router := gin.Default()

	router.GET("/api/v1/captcha", func(c *gin.Context) {
		c.Request.URL.Path = "/api/v2/captcha"
		router.HandleContext(c)
	})

	router.GET("/api/v2/captcha", func(c *gin.Context) {
		c.JSON(http.StatusOK, gin.H{"version": "v2"})
	})

	router.Run()
}
```

## 响应格式

让每个接口的响应格式保持一致能让前端处理接口返回的数据时更加方便。

```go
// 统一的响应风格
type Response struct {
	Code    int    `json:"code"`
	Data    any    `json:"data,omitempty"`
	Message string `json:"message"`
}

// 业务状态码
const (
	CodeOK           = 0    // 成功
	CodeUserNotFound = 1001 // 用户不存在
)

// 返回成功响应
func OK(c *gin.Context, data any) {
	c.JSON(http.StatusOK, Response{
		Code:    CodeOK,
		Data:    data,
		Message: "success",
	})
}

// 返回错误响应，HTTP 状态码和业务状态码分别指定
func Fail(c *gin.Context, status int, code int, message string) {
	c.JSON(status, Response{
		Code:    code,
		Data:    nil,
		Message: message,
	})
}

func main() {
	r := gin.Default()

	r.GET("/api/users/:id", func(c *gin.Context) {
		id := c.Param("id")
		// 模拟查找
		if id == "0" {
			Fail(c, http.StatusNotFound, CodeUserNotFound, "用户不存在")
			return
		}
		OK(c, gin.H{"id": id, "name": "pany"})
	})

	r.Run()
}
```

请求成功很好理解，HTTP 状态码为 `200`，业务状态码也为对应的成功码。请求错误则分为两种情况，一种是 HTTP 状态码不为 `200`，另一种是业务状态码为错误码。

所以接口在处理业务状态码为错误码的时候，一般有两种流派：

- 统一返回 HTTP 状态码 `200`，业务状态码为错误码
- 返回合适的 HTTP 错误码，业务状态码为具体的错误码

上面给出的示例就对应着第二种流派，也是 Gin 官方文档中推荐的流派。但是开源社区里国内的项目大多采用第一种流派。属于是各有优劣，大家知晓即可。

还需要留意的是上面出现了**结构体标签**语法，是我们之前没有讲到的：

```go
type Response struct {
	Code    int    `json:"code"`
	Data    any    `json:"data,omitempty"`
	Message string `json:"message"`
}
```

`json:"code"` 是 Go 的结构体标签，写在字段后面的反引号字符串里，用来告诉 `encoding/json` 这个字段在 JSON 里叫什么名字。而 `omitempty` 表示字段可以不存在。

**作者正在努力更新...**
