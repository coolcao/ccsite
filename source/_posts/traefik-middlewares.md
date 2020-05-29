---
title: traefik自定义中间件
date: 2020-05-29 10:28:58
tags: [traefik]
categories:
- 技术博客
- 原创
---

# Træfɪk自定义中间件

Træfɪk 是一个为了让部署微服务更加便捷而诞生的现代HTTP反向代理、负载均衡工具。 它支持多种后台 (Docker, Swarm, Kubernetes, Marathon, Mesos, Consul, Etcd, Zookeeper, BoltDB, Rest API, file…) 来自动化、动态的应用它的配置文件设置。

Traefik是golang写的，与docker，k8s深度集成，支持服务的自动发现与热部署。

从Traefik2.0版本开始，其加入了中间件，功能更丰富，但是目前（v2.2）官方还不支持以插件的形式自定义中间件。因此，如果要自定义中间件的话，需要在源码上做改动。

<!-- more -->

比如，我们要添加一个简单的中间件，来把所有的请求都加上一个自定义的请求头：`Hello: World`。

## 前置条件
1. 安装golang 1.14及以上版本
2. 安装git

## clone Traefik源码
首先，克隆Traefik的源码，并切换到自己的开发分支进行开发。

```shell
# 克隆
git clone https://github.com/containous/traefik.git
# 切换到最新的2.2.1分支
git checkout v2.2.1
# 从v2.2.1切换自己的开发分支，比如叫dev
git checkout -b dev
```

## 自定义中间件

1. 首先，在`pkg/middlewares/`包下新建一个包，我们就命名为`hello`,然后新建`hello.go`。内容如下：
   ```go
    package hello

    import (
    	"context"
    	"net/http"

    	"github.com/containous/traefik/v2/pkg/config/dynamic"
    	"github.com/containous/traefik/v2/pkg/log"
    	"github.com/containous/traefik/v2/pkg/middlewares"
    	"github.com/containous/traefik/v2/pkg/tracing"
    	"github.com/opentracing/opentracing-go/ext"
    )

    const (
    	typeName = "Hello"
    )

    type hello struct {
    	next http.Handler
    	name string
    }

    // New creates a new handler.
    func New(ctx context.Context, next http.Handler, config dynamic.Hello, name string) (http.Handler, error) {
    	log.FromContext(middlewares.GetLoggerCtx(ctx, name, typeName)).Debug("Creating middleware")
    	var result *hello

    	result = &hello{
    		next: next,
    		name: name,
    	}
    	return result, nil

    }

    func (a *hello) GetTracingInformation() (string, ext.SpanKindEnum) {
    	return a.name, tracing.SpanKindNoneEnum
    }

    func (a *hello) ServeHTTP(rw http.ResponseWriter, req *http.Request) {
    	logger := log.FromContext(middlewares.GetLoggerCtx(req.Context(), a.name, typeName))
    	logger.Infoln("Hello World")
    	req.Header.Add("Hello", "World")
    	a.next.ServeHTTP(rw, req)
    }

   ```

   其中两个函数New和ServeHttp两个函数是必须的，要根据自己的业务重写。New函数是创建一个新的handle，ServeHttp函数就是具体的中间件的处理函数。
   我们要做的就是在头部添加一个Hello:World的header。
2. New函数的第三个参数,config是 dynamic.Hello类型，这个需要我们新定义，实际就是从配置文件读取的当前中间件的配置信息。
   在`pkg/config/dynamic/middlewares.go`文件中新建一个Hello的结构体：
   ```go
   type Hello struct {
    }
   ```
   我们这个中间件很简单，只是向请求新加一个自定义的请求头，所以这里就没定义属性。根据自己业务来。
   然后，再在Middleware这个结构体（pkg/config/dynamic/middlewares.go）中添加刚才新建的属性：
   ```go
   type Middleware struct {
    	AddPrefix         *AddPrefix         `json:"addPrefix,omitempty" toml:"addPrefix,omitempty" yaml:"addPrefix,omitempty"`
    	StripPrefix       *StripPrefix       `json:"stripPrefix,omitempty" toml:"stripPrefix,omitempty" yaml:"stripPrefix,omitempty"`
    	StripPrefixRegex  *StripPrefixRegex  `json:"stripPrefixRegex,omitempty" toml:"stripPrefixRegex,omitempty" yaml:"stripPrefixRegex,omitempty"`
    	ReplacePath       *ReplacePath       `json:"replacePath,omitempty" toml:"replacePath,omitempty" yaml:"replacePath,omitempty"`
    	ReplacePathRegex  *ReplacePathRegex  `json:"replacePathRegex,omitempty" toml:"replacePathRegex,omitempty" yaml:"replacePathRegex,omitempty"`
    	Chain             *Chain             `json:"chain,omitempty" toml:"chain,omitempty" yaml:"chain,omitempty"`
    	IPWhiteList       *IPWhiteList       `json:"ipWhiteList,omitempty" toml:"ipWhiteList,omitempty" yaml:"ipWhiteList,omitempty"`
    	Headers           *Headers           `json:"headers,omitempty" toml:"headers,omitempty" yaml:"headers,omitempty"`
    	Errors            *ErrorPage         `json:"errors,omitempty" toml:"errors,omitempty" yaml:"errors,omitempty"`
    	RateLimit         *RateLimit         `json:"rateLimit,omitempty" toml:"rateLimit,omitempty" yaml:"rateLimit,omitempty"`
    	RedirectRegex     *RedirectRegex     `json:"redirectRegex,omitempty" toml:"redirectRegex,omitempty" yaml:"redirectRegex,omitempty"`
    	RedirectScheme    *RedirectScheme    `json:"redirectScheme,omitempty" toml:"redirectScheme,omitempty" yaml:"redirectScheme,omitempty"`
    	BasicAuth         *BasicAuth         `json:"basicAuth,omitempty" toml:"basicAuth,omitempty" yaml:"basicAuth,omitempty"`
    	DigestAuth        *DigestAuth        `json:"digestAuth,omitempty" toml:"digestAuth,omitempty" yaml:"digestAuth,omitempty"`
    	ForwardAuth       *ForwardAuth       `json:"forwardAuth,omitempty" toml:"forwardAuth,omitempty" yaml:"forwardAuth,omitempty"`
    	InFlightReq       *InFlightReq       `json:"inFlightReq,omitempty" toml:"inFlightReq,omitempty" yaml:"inFlightReq,omitempty"`
    	Buffering         *Buffering         `json:"buffering,omitempty" toml:"buffering,omitempty" yaml:"buffering,omitempty"`
    	CircuitBreaker    *CircuitBreaker    `json:"circuitBreaker,omitempty" toml:"circuitBreaker,omitempty" yaml:"circuitBreaker,omitempty"`
    	Compress          *Compress          `json:"compress,omitempty" toml:"compress,omitempty" yaml:"compress,omitempty" label:"allowEmpty"`
    	PassTLSClientCert *PassTLSClientCert `json:"passTLSClientCert,omitempty" toml:"passTLSClientCert,omitempty" yaml:"passTLSClientCert,omitempty"`
    	Retry             *Retry             `json:"retry,omitempty" toml:"retry,omitempty" yaml:"retry,omitempty"`
    	ContentType       *ContentType       `json:"contentType,omitempty" toml:"contentType,omitempty" yaml:"contentType,omitempty"`
    	Hello             *Hello             `json:"hello,omitempty" toml:"hello,moitempty" yaml:"hello,omitempty"`
    }

   ```
   其中最后一行就是我们新定义的Hello配置的结构体。

3. 好了，现在我们要启用这个中间件。在`pkg/server/middleware/middlewares.go`中的`buildConstructor`函数中，添加初始化这个中间件的代码：
    ```go
    if config.Hello != nil {
		if middleware != nil {
			return nil, badConf
		}
		middleware = func(next http.Handler) (http.Handler, error) {
			return hello.New(ctx, next, *config.Hello, middlewareName)
		}
	}
    ```


好了，到现在，最简单的一个中间件就定义完了。下面我们来编译一下二进制文件。

## 编译
1. 先进入webui目录，编译前端dashboard。
    1. `cd webui`
    2. `npm install`
    3. `npm run build`
2. 下载依赖。`go mod download`
3. 安装 go-bindata: `go get github.com/containous/go-bindata/...`
4. 将一些非代码组件打到二进制：`go generate`
5. 编译二进制：`go build ./cmd/traefik`

编译得到二进制文件，就可以测试了。

## 启用自定义中间件
在traefik的配置文件中启用该中间件：

```toml
[http]
  # Add the router
  [http.routers]
    [http.routers.router0]
      entryPoints = ["web"]
      # 启用hello中间件
      middlewares = ["myhello"]
      service = "foob"
      rule = "PathPrefix(`/api/v2/`)"


  [http.middlewares]
    # 定义hello中间件
    [http.middlewares.myhello.hello]

  [http.services]
    [http.services.foob]
      [http.services.foob.loadBalancer]
        [[http.services.foob.loadBalancer.servers]]
          url = "http://localhost:8808/"

```

这样，所以经过web这个入口点的请求，都会在请求上加上一个Hello的请求头。

到这里，简单的中间件的自定义过程就结束了。

## 总结
Traefik的中间件自定义过程还是挺简单的，无非就下面几个步骤，整理一下：

1. 克隆源码到本地，并从最新分支切换一个自己的开发分支出来。
2. 下载依赖。`go mod download`
3. 安装go-bindata：`go get github.com/containous/go-bindata/...`
4. 修改代码，加中间件的处理代码
5. 编译前端
6. 编译后端


这样定义的中间件，耦合太高，改动了源码。还是希望官方尽快将中间件以插件的形式分离出来，这样我们只需要开发插件并启用插件即可。

当然，在这之前，如果是很通用的中间件，也可以提给官方，合并到官方。
