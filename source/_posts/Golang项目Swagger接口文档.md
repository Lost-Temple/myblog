---
title: Golang项目Swagger接口文档
tags:
  - swagger
  - swaggo
  - 接口文档
categories:
  - Golang
  - 库
  - swaggo
cover: 'https://s2.loli.net/2022/12/22/d42tkxbPGjYWLTQ.png'
abbrlink: 13550
date: 2022-12-21 20:22:46
---

# 安装swagger
[swagger github地址](https://github.com/swaggo/swag/blob/master/README.md)
>go1.17版本后，使用go install 安装swag
```shell
go install github.com/swaggo/swag/cmd/swag@latest
```

>验证是否安装成功（会在$GOPATH/bin)目录下生成swag可执行文件
```shell
swag -v
```

# 对gin框架的支持
```shell
go get -u github.com/swaggo/gin-swagger
go get -u github.com/swaggo/files
```

# 注解
## API接口的注解
| 注解 | 描述 |
| -- | -- |
| @Summary | 摘要 |
| @Produce | API可以产生的MIME类型列表。我们可以把MIME类型简单地理解为响应类型，如JSON/XML/HTML等 |
| @Param | 参数格式，从左到右分别为：参数名、入参类型、数据类型、是否必填和注释 |
| @Success | 响应成功，从左到右分别为：状态码、参数类型、数据类型和注释 |
| @Failure | 响应失败，从左到右分别为：状态码、参数类型、数据类型和注释 |
| @Router | 路由，从左到右分别为：路由地址和HTTP方法 |

## 例子
```go
// @Summary 获取多个标签
// @Produce json
// @Param name query string false "标签名称" maxlength(100)
// @Param state query int false "状态" Enums(0, 1) default(1)
// @Param page query int false "页码"
// @Param page_size query int false "每页数量"
// @Success 200 {object} model.TagSwagger "成功"
// @Failure 400 {object} errcode.Error "请求错误"
// @Failure 500 {object} errcode.Error "内部错误"
// @Router /api/v1/tags [get]
func (t Tag) List(c *gin.Context) {

}

// @Summary 新增标签
// @Produce json
// @Param name body string true "标签名称" minlength(3) maxlength(100)
// @Param state body int false "状态" Enums(0, 1) default(1)
// @Param created_by body string false "创建者" minlength(3) maxlength(100)
// @Success 200 {object} model.Tag "成功"
// @Failure 400 {object} errcode.Error "请求错误"
// @Failure 500 {object} errcode.Error "内部错误"
// @Router /api/v1/tags [post]
func (t Tag) Create(c *gin.Context) {

}

// @Summary 更新标签
// @Produce json
// @Param id path int true "标签ID"
// @Param name body string false "标签名称" minlength(3) maxlength(100)
// @Param state body int false "状态" Enums(0, 1) default(1)
// @Param modified_by body string true "修改者" minlength(3) maxlength(100)
// @Success 200 {array} model.Tag "成功"
// @Failure 400 {object} errcode.Error "请求错误"
// @Failure 500 {object} errcode.Error "内部错误"
// @Router /api/v1/tags/{id} [put]
func (t Tag) Update(c *gin.Context) {

}

// @Summary 删除标签
// @Produce json
// @Param id path int true "标签ID"
// @Success 200 {string} string "成功"
// @Failure 400 {object} errcode.Error "请求错误"
// @Failure 500 {object} errcode.Error "内部错误"
// @Router /api/v1/tags/{id} [delete]
func (t Tag) Delete(c *gin.Context) {

}
```

# main 方法
既然接口方法本身有了注解，那么针对这个项目，能不能写注解呢？如果有很多个项目，如何知道这个项目具体是哪个呢？实际上是可以识别出来的，只需针对main方法写入如下注解即可：
```go
// @title 博客系统
// @version 1.0
// @description Go 编程之旅：一起用Go做项目
// @termsOfService https://github.com/go-programming-tour-book
func main() {

}

```

# 生成
```shell
swag init
```