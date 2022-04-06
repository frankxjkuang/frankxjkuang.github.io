---
layout: post
title:  在 Kubernetes 中部署 go 应用程序
date:   2022-03-07 10:30:00 +0800
categories: k8s
tag: note
---

* content
{:toc}

目标：
- 创建 go deployment
- 创建 go service
- 调用 go 应用

#### 创建一个名称为my-gin的go项目

代码如下
```go
package main

import (
	"fmt"
	"github.com/gin-gonic/gin"
)

func main()  {
	fmt.Println("hello world")

	r := gin.Default()

	r.GET("/", HomePage)
	r.POST("/", PostHomePage)
	r.Run()
}

func HomePage(c *gin.Context)  {
	reqHeader := c.Request.Header.Get("reqHeader")
	c.JSON(200, gin.H{
		"msg": "Hello World",
		"reqHeader": reqHeader,
	})
}

func PostHomePage(c *gin.Context)  {
	c.JSON(200, gin.H{
		"msg": "Post Hello World",
	})
}
```

#### 创建 Dockerfile 文件
```dockerfile
FROM golang:latest as builder
ENV GOPROXY https://goproxy.cn,direct
WORKDIR /workSpace/workphone/src/my-gin
COPY . .
RUN go get
RUN CGO_ENABLED=0 go build -a -ldflags="-w -s"

FROM alpine as final
WORKDIR /my-gin
COPY --from=builder /workSpace/workphone/src/my-gin /my-gin
EXPOSE 8080
ENTRYPOINT ["/my-gin/my-gin"]
```

#### 打镜像
```
docker build -t my-gin .
```

完成之后查看镜像
```
docker images | grep my-gin

# my-gin  latest   05eb3df8f614   About a minute ago   12.6MB
```

#### 推送镜像到 dockerhub 仓库
```
docker login
docker tag my-gin:latest frankxjkuang/my-gin:latest
docker push frankxjkuang/my-gin:latest
```

#### 创建 Deployment

在项目根目录下创建 `deployment.yaml` 文件
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-gin
  labels:
    app: my-gin
spec:
  replicas: 2
  selector:
    matchLabels:
      app: my-gin
  template:
    metadata:
      labels:
        app: my-gin
    spec:
      containers:
        - name: my-gin
          # 注意镜像  
          image: frankxjkuang/my-gin:latest
          ports:
            - containerPort: 8080
```

在项目根目录下创建 `service.yaml` 文件：
```
apiVersion: v1
kind: Service
metadata:
  name: my-gin-svc
  labels:
    app: my-gin
spec:
  selector:
    app: my-gin
  ports:
    - port: 8080
      protocol: TCP
      name: http-port
      targetPort: 8080
      nodePort: 30001
  type: NodePort
```

启动程序
```
kubectl apply -f deployment.yaml 
kubectl apply -f service.yaml
```

查看 service 的运行情况
```
# kubectl get services -o wide <service-name>
kubectl get services -o wide 

# my-gin-svc  NodePort    10.107.177.243   <none>   8080:30001/TCP   68s   app=my-gin
```

#### 调用 go 应用接口
```
curl 127.0.0.1:30001
# {"msg":"Hello World"}

curl -XPOST 127.0.0.1:30001
# {"msg":"Post Hello World"}
```