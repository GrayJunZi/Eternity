# Eternity

Backend Master Class [Golang + Postgres + Kubernetes + gRPC]
---
了解有关后端Web开发的一切:Golang,Postgres,Redis,GRPC,Docker,Kubernetes,AWS,CI/CD

> - 课程学习地址: [B站](https://www.bilibili.com/video/BV1dy4y1u7Sq/)
> - 课程代码仓库：[Github](https://github.com/techschool/simplebank)

## 一、介绍

创建一个简单银行项目，包含以下功能。

- 创建并管理帐号
- 记录余额变更
- 金额交易事务

数据库设计：
- 使用 [dbdiagram.io](https://dbdiagram.io) 设计 SQL 数据库架构。
- 将架构另存为 PDF 或 PNG 图标。
- 生成SQL脚本。

## 二、安装 Docker 和 Postgres

拉取 `postgres` 镜像
```bash
docker pull postgres:12-alpine
```

创建容器
```bash
docker run --name postgres12 -p 5432:5432 -e POSTGRES_USER=root -e POSTGRES_PASSWORD=root -d postgres:12-alpine
```

进入容器
```bash
docker exec -it postgres12 psql -U root
```

退出容器
```bash
\q
```

## 三、Golang迁移数据库

### 安装 go

安装 `go`
```bash
rm -rf /usr/local/go && tar -C /usr/local -xzf go1.20.3.linux-amd64.tar.gz
```

修改 $HOME/.profile 文件追加如下命令
```bash
export PATH=$PATH:/usr/local/go
export GO111MODULE=on
export GOPROXY=https://goproxy.cn
```

### 安装 golang-migrate

安装 `golang-migrate`
```bash
mkdir /usr/local/golang-migrate
tar -C /usr/local/golang-migrate -xzf migrate.linux-arm64.tar.gz
```

修改 $HOME/.profile 文件追加如下命令
```bash
export PATH=$PATH:/usr/local/golang-migrate
```

查看 `golang-migrate` 版本
```bash
migrate version
```

### 迁移数据库

生成迁移文件
```bash
migrate create -ext sql -dir db/migration -seq init_schema
```

`migrate up` - 执行脚本更新数据库。
`migrate down` - 从数据库中撤回更改。

查看正在运行中的容器
```bash
docker ps
```

停止正在运行的容器
```bash
docker stop postgres12
```

查看所有容器
```bash
docker ps -a
```

进入容器
```bash
docker exec -it postgres12 /bin/sh
```

创建数据库
```bash
createdb --username=root --owner=root simple_bank
psql simple_bank
\q
```

迁移数据库
```bash
migrate -path db/migration -database "postgresql://root:root@localhost:5432/simple_bank?sslmode=disable" -verbose up
```

安装 `make`
```bash
sudo apt-get install make
```

查看 `make` 版本
```bash
make --version
```

执行 `Makefile` 文件
```bash
# 向上迁移
make migrateup

# 向下迁移
make migratedown
```

## 四、根据SOL生成CRUD代码

- Create - 在数据库中插入新记录
- Read - 在数据库中选择或搜索记录
- Update - 更改数据库中记录的某些字段
- Delete - 从数据库中删除记录

### database/sql

- 优点是 速度快并且代码简单(Straightforward)
- 缺点是 必须手动将SQL字段映射到变量上，容易出错。

### gorm

- 已经封装好CRUD的代码，只需声明模型。
- 必须学习如何使用gorm提供的函数编写查询。
- 当流量很高(high load)时运行很慢。

### sqlx

- 查询几乎和标准库一样快以及容易使用
- 字段映射是通过查询文本或结构标签完成的

### sqlc

- 速度快且容易使用
- 自动生成代码
- 缺点是只完全支持 PostgreSQL, MySQL处于实验阶段

安装 `sqlc`
```bash
sudo snap install sqlc
```

初始化 `sqlc`
```bash
sqlc init
```

生成代码
```bash
sqlc generate
```

初始化项目
```bash
go mod init github.com/grayjunzi/backend-master-class-golang
go mod tidy
```

## 五、数据库增删改查单元测试

安装依赖包 
```bash
go get github.com/lib/pq
go get github.com/stretchr/testify
```

## 六、实现数据库事务

### 什么是数据库事务？

- 单一的工作单元。
- 通常由多个数据库操作组成。

### 为什么需要事务呢？

1. 提供可靠和一致的工作单元，即使在系统出现故障的情况下。
2. 在并发访问数据库的程序之间提供隔离

### ACID 属性(Property)

数据库事务必须满足ACID属性

- Atomicity(原子性) - 要么所有操作都成功完成，要么事务失败，数据库保持不变。
- Consitency(一致性) - 数据库状态在事务执行之后必须有效。必须满足所有约束。
- Isolation(隔离性) - 并发事务不能相互影响。
- Durability(持久性) - 成功事务写入的数据必须记录在持久性存储中。

## 七、数据库事务锁

### 处理死锁

## 八、避免数据库死锁

## 九、事务隔离级别

### 读现象(Read Phenomena)

- `脏读(Dirty Read)` - 事务读取其他并发未提交事务写入的数据。
- `不可重复读(Non Repeatable Read)` - 一个事务两次读取同一行并看到不同的值，因为它已被其他已提交的事务修改
- `幻读(Phantom Read)` - 事务重新执行用于查找满足条件的行的查询，并且由于其他已提交事务的更改而看到一组不同的行。
- `序列化异常(Serialization Anomaly)` - 一组并发提交的事务的结果是不可能实现的，如果我们试图以任何顺序依次运行它们而不重叠

### 标准隔离级别

American National Standards Institute - ANSI

1. 读取未提交 - 可以看到未提交事务写入的数据。
2. 读取已提交 - 只看到已提交事务写入的数据。
3. 可重复读取 - 相同的读取查询总是返回相同的结果。
4. 可序列化的 - 如果按某种顺序而不是并发地执行事务则可以实现相同的结果。

### MySQL

查看事务隔离级别
```sql
select @@transaction_isolation;
```

查看全局事务隔离界别
```sql
select @@global.transaction_isolation;
```

修改事务隔离级别
```sql
set session transaction isolation level read uncommitted;
```

开启事务
```sql
start transaction;
```

提交事务
```sql
commit;
```

回滚事务
```sq;
rollback;
```

### Postgres

查看事务隔离级别
```bash
show transaction isolation level;
```

修改事务隔离级别
```bash
set transaction isolation level read uncommitted;
```


### 重试机制

处理可能存在错误、超时或死锁。

## 十、Github Actions

### Workflow

- 是一个自动过程。
- 由一个以上的Job组成。
- 由事件触发、计划或手动。
- 将 `.yml` 添加到存储库。

```yml
name: build-and-test

on:
  push:
    branches: [ master ]
  schedule:
    - cron: '*/15 * * * *'

jobs:
  build:
    runs-on: ubuntu-latest
```

### Runner

- 是运行Job的服务
- 一次运行一个Job。
- Github托管或自托管。
- 向Github报告进度、日志和结果。

```yml
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - name: Check out code
        uses: actions/checkout@v2
      - name: Build server
        run: ./build_server.sh
    test:
      needs: build
      runs-on: ubuntu-latest
      steps:
        - run: ./test_server.sh
```

### Job

- 是在同一个Runner上执行的一组步骤。
- 正常作业并行运行。
- 连续运行相关作业。

```yml
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - name: Check out code
        uses: actions/checkout@v2
      - name: Build server
        run: ./build_server.sh
    test:
      needs: build
      runs-on: ubuntu-latest
      steps:
        - run: ./test_server.sh
```

### Step

- 是一个单独的任务。
- 在作业中连续运行。
- 包含多个 Actions

### Action

- 是一个独立的命令。
- 在一个步骤内连续运行。
- 可重复使用。

```yml
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - name: Check out code
        uses: actions/checkout@v2
      - name: Build server
        run: ./build_server.sh
```

## 十一、使用 Gin 实现 HTTP API

### 流行的Web框架

- Gin
- Beego
- Echo
- Revel
- Martini
- Fiber
- Buffalo

### 流行 HTTP Router 库

- FastHttp
- Gorilla Mux
- HttpRouter
- Chi


### 安装 `gin`

```bash
go get -u github.com/gin-gonic/gin
```

## 十二、使用 Viper 从文件或环境变量中加载配置

### 为什么使用文件加载配置？

轻松指定本地开发和测试的默认配置。

### 为什么使用环境变量加载配置？

使用docker容器部署时轻松覆盖默认配置。

### 为什么使用Viper？

`Viper` 是一个非常受环境的包，它可以从配置文件中查找、浇在和解析配置文件。

- 支持多种类型的文件例如 `JSON`、`TOML`、`YAML`、`ENV`、`INI`。
- 支持从远程系统中读取配置信息例如 `ETCD`、`Consul`。
- 支持监视配置文件中的更改，并通知应用程序。
- 使用`Viper`保存我们对文件所作的任何配置修改。

### 安装viper

```bash
go get github.com/spf13/viper
```

## 十三、用于测试Go中的 HTTP API 并实现100%覆盖的MockDB

### 为什么使用 mock 数据库？

- 独立测试 - 隔离测试数据以避免冲突。
- 测试更快 - 减少与数据库对话的大量时间。
- 100%覆盖率 - 轻松设置边缘案例:意外错误

首先，它可以帮助我们更轻松的编写独立的测试。
其次，我们的测试将运行的更快。不必花时间与数据库进行交互并等待返回结果。所有的操作都将再内存中执行，并在同一进程中执行。
第三个也是非常重要的原因是：它允许我们编写实现100%覆盖率的测试。

### 用模拟数据库测试我们的API是否足够好?

是的!我们真正的DB存储已经过测试。

模拟数据库和真实数据库应该实现相同的接口。

### 如何模拟数据？

- 使用假数据库 - 内存(Memory), 实现一个假版本的数据库:在内存中存储数据。 
- 使用数据库存根 - GoMock, 生成和构建返回硬编码值的存根。

### 使用 gomock

安装 `mockgen`
```bash
go get github.com/golang/mock/mockgen@v1.6.0
```

查看 `mockgen` 路径
```bash
which mockgen
```

查看 `mockgen` 帮助
```bash
mockgen -help
```

```bash
mockgen --package mockdb -destination db/mock/store.go github.com/grayjunzi/backend-master-class-golang/db/sqlc Store
```

## 十四、自定义参数验证

## 十五、添加用户表并添加外键约束

### 创建一个新的迁移

```bash
migrate create -ext sql -dir db/migration -seq add_users
```

## 十六、处理数据库错误

## 十七、使用哈希对密码进行加密

### 如何安全存储密码？

密码将使用 `brcypt` 散列函数进行散列以产生散列值。
除了输入密码 `brcypt` 还需要一个 (COST, SALT) 参数，这将决定算法的密钥扩展轮数或迭代次数。
`Brcypt` 还会产生一个随即盐，用于这些迭代，将有助于防止彩虹 彩虹攻击(rainbow)。

在这个哈希字符串中，有四部分组成，第一部分是哈希算法标识符，第二部分是COST(2^*轮密钥扩展)，第三部分是长度16字节或128位的盐，它使用base64格式编码，这将生成22个字符的字符串，最后一部分是24字节的哈希值，编码位31个字符。四个部分连接在一起形成一个哈希字符串。

## 十八、编写更强的单元测试

## 十九、基于Token的身份认证

### 为什么 `PASETO` 比 `JWT` 更好？

JWT(Json web Token)是使用最广泛的一种，但 `JWT` 一些安全问题，主要是因为它设计的标准很差，使用`PASETO` 有望为应用程序带来更好的安全性。

### Json Web Token (JWT)

它是一个base64编码的字符串，由三部分组成，用 `.` 分隔，第一部分是 `token` 的 `header` 用于签署令牌的算法和类型，第二部分是存储有关登录用户的信息 例如 id 用户名 以及令牌过期的时间戳，第三部分用于验证签名。

### JWT 签名算法

- 对称加密算法(Symmetric digital signature algorithm) 
  - 使用相同的密钥(secret key)来签署和验证令牌。
  - 该算法适用与本地适用，对于内部服务可以共享密钥。
  - HS256, HS384, HS512
    - HS256 = HMAC + SHA256 的组合
    - HMAC 代表基于哈希的消息认证代码(Hash-based Message Authentication Code)
    - SHA 是安全散列算法(Secure Hash Algorithm)
    - 256/384/512 是输出的位数。

- 非对称加密算法(Asymmetric digital signature algorithm)
  - 该算法由一对密钥而不是一个单独的密钥，私钥(private key)用于对令牌进行签名，公钥(public key)用于验证签名。
  - 该算法可轻松地与任何外部第三方服务共享我们的公钥，不用担心泄漏私钥。
  - RS256, RS384, RS512 | PS256, PS384, PS512 | ES256, ES384, ES512
    - RS256 = RSA PKCSv1.5 + SHA256 的组合算法，PKCS(Public-Key Cryptography Standards)
    - PS256 = RSA PSS + SHA256 的组合，PSS(Probabilistic Signature Scheme)
    - ES256 = ECDSA + SHA256 的组合，ECDSA(Elliptic Curve Digital Signature Algorithm)
  
### JWT 的问题

- 弱签名算法(Weak algorithms)
  - JWT给了开发者太多的算法选择。
  - 包括易受攻击的算法，例如 RSA PKCSv1.5 容易受到 padding oracle 攻击，ECDSA 容易受到无效曲线攻击(invalid-curve attach)

- 伪造(Trivial Forgery)
  - 将 token 的 header 中 `alg` 设置为 none。
  - 将 token 中 header 的 `alg` 设置为 `HS256` 进行验证。

### Platform-Agnostic SEcurity TOkens (PASETO)

PASETo 是最成功的设计之一，被社区广泛接受为JWT的最佳安全替代方案。

- 提供开箱即用的强大签名算法。
- 开发人员不必再选择算法。
- 只需选择PASETO版本。
- 每个版本都已经实现了一个强大的密码套件(cipher suite)。
- 最多只有两个最新版本的PASETO处于活动状态。

PASETO 它只对payload进行base64编码并对令牌进行签名，实际上使用密钥对令牌中的所有数据进行加密和验证。使用具有关联数据(AEAD)算法的强身份验证加密。PASETO版本1中使用的AEAD算法是带有 ASE256 CTR + HMAC SHA384。
对于公共情况，有外部服务需要验证token，我们必须使用非对称加密算法，只对它进行base64编码，并使用私钥数字签名对内容进行签名。PASETO版本1中选择的非对称加密算法是带有RSA PSS + SHA384 算法。

PASETO v2 对于本地对称密钥场景它使用 `XChaCha20-Poly1305` 算法，对于公共非对称密钥方案，使用带有曲线的 `Ed25519(EdDSA + Curve25519)` 算法。

### PASETO 结构

适用于本地的令牌有四个主要部分用 `.` 分隔，第一部分是 PASETO的版本，第二部分是token的用途，表示是用于本地场景还是公共场景，第三部分是主要内容，即令牌的有效载荷(payload)数据，它是加密的，解密之后有三个较小的部分，
第一个小部分中具有消息和过期时间，其次是在加密和消息认证过程中使用的 nonce 的值，最后是消息认证标签 验证加密消息及其关联的未加密数据。第四部分是footer，可以在footer中存储任何公共信息，因为它不会像有效负载正文那样被加密，而是只进行base64编码。其中footer部分是可选的，可以拥有没有footer的PASETO令牌。

Paragon Initiative Enterprises 是PASETO的发明者

- Version: v2
- Purpose: local
- Payload
  - Body
  - Nonce
  - Authentication tag

适用于公共的令牌有三个部分，用 `.` 分隔，第一个部分是PASETO的版本，第二部分是token的用途，第三部分是有效载荷数据不会被加密而是被base64编码，其中一部分是令牌的签名由数字签名算法使用私钥创建，服务器将使用其配对的公钥来验证签名的真实性。

- Version: v2
- Purpose: public
- Payload

## 二十、创建和验证JWT与PASETO

安装 `uuid`
```bash
go get github.com/google/uuid
```

安装 `jwt`
```bash
go get github.com/dgrijalva/jwt-go
```

安装 `paseto`
```bash
go get github.com/o1egl/paseto
```

## 二十一、使用 PASETO 和 JWT 实现 登录API

## 二十二、实现认证中间件和鉴权规则

## 二十三、构建最小的Docker映像

### 分支

创建新分支
```bash
git checkout -b ft/docker
```

提交代码
```bash
git push origin ft/docker
```

### Docker

构建镜像
```bash
docker build -t simplebank:latest .
```

## 二十四、连接同一个docker网络中的容器

运行容器
```bash
docker run --name simplebank -p 8080:8080 -e GIN_MODE=release simplebank:latest
```

移除容器
```bash
docker rm simplebank
```

移除镜像
```bash
docker rmi simplebank
```

查看容器的网络设置
```bash
docker container inspect postgres12
```

查看网络
```bash
docker network ls
docker network inspect container
```

创建网络
```bash
docker network create simplbank-network
```

连接网络
```bash
docket network connect simplebank-network postgres12
```

运行容器
```bash
docker run --name simplebank --network simplebank-network -p 8080:8080 -e GIN_MODE=release simplebank:latest
```

## 二十五、使用 Docker Compose

## 二十六、创建免费AWS帐号

略过

## 二十七、使用Github Actions自动构建推送镜像到AWS ECR中

略过

## 二十八、在AWS RDS中创建生产数据库

略过

## 二十九、使用AWS Secrets Manager管理密钥

略过

## 三十、Kubernetes架构以及在AWS中创建EKS集群

### 什么是 Kubernetes？

- 一个开源容器编排引擎。
- 用于自动部署、扩展和管理容器化应用程序。

### Kubernetes的组件

- Worker Node - 运行容器化应用程序的工作机器或节点。每个Worker节点中，都有一个Kubelet代理。
  - Kubelet agent:确保容器在pods内运行
  - 容器运行时: Kubernetes支持多种容器，Docker、Containerd、CRI-O。
  - Kube-proxy: 维护网络规则，允许与pods通信

- Master Node - 它的职责是管理集群的工作节点和Pod。

> 第二部分是 Control Plane，它在主节点上运行

## 三十一、使用Kubectl 与 K9s连接AWS中的Kubernetes集群

略过

## 三十二、在AWS EKS上部署web app到Kubeneters集群中

略过

## 三十三、注册一个域名并建立 A-record

略过

## 三十四、在Kubernetes集群中设置Ingress

略过

## 三十五、k8s中的自动和免费TLS证书

略过

## 三十六、使用Github Action自动部署到Kubentes中

略过

## 三十七、使用Refresh Token管理会话

### 如何管理用户会话？

## 三十八、根据DBML代码生成数据库文档


- 使用 [dbdocs.io](https://dbdocs.io) 生成数据库文档。

全局安装 `dbdocs`
```bash
npm install -g dbdocs
```

查看 `dbdocs` 版本
```bash
dbdocs
```

登录 `dbdocs`
```bash
dbdocs login
```

构建 `dbml`
```bash
dbdocs build doc/db.dbml
```

vscode安装 `vscode-dbml`

## 三十九、GRPC介绍

### 什么是gRPC？

- `gRPC` - 是远程过程调用框架(Remote Procedure Call Framework)
  - 客户端可以在服务器上执行远程过程
  - 远程交互代码由gRPC处理
  - API和数据结构代码自动生成
  - 支持多种编程语言

- 最初由谷歌开发，现在是云原生计算基础的一部分。

### gRPC是如何工作？

1. 定义API和数据结构
2. 生成gRPC存根(stubs)
3. 实现服务
4. 客户端使用

### 为什么使用gRPC？

- 高性能 - 归功于底层的 HTTP/2 协议，它提供了一些好处，例如以二进制格式传输数据、多路复用(multiplexing, 允许通过相同的TCP连接发送多个请求)、请求头压缩以及客户端与服务器之间的双向通信等。
- 更好的API契约 - 服务器与客户端使用强类型的 请求/响应 数据结构 共享API的相同协议缓冲区定义(protobuf)。
- 自动代码生成 - 是gRPC最重要的特性之一，它可以让我们更快地开发服务，因为所有在客户端和服务器之间进行序列化、反序列或传输数据的代码都已经由gRPC生成和处理，所以我们需要关注的是实现服务的核心逻辑。

### gRPC的四种类型是什么？

- 一元gPRC(Unary gRPC) - 客户端发送1个请求，服务器回复1个响应。类似于普通的HTTP API。
- 客户端流式gRPC - 客户端发送多条消息流，并且它希望服务器只返回1个单一响应。
- 服务器流式gRPC - 客户端发送1个请求，服务器回复多条消息流。
- 双向流式gPRC(Bidirectional streaming gRPC) - 客户端和服务器以任意顺序并行发送和接收多条消息。它非常灵活且没有阻塞，这意味着在发送下一条消息之前，任何一方都不需要等待对方的响应。

### gRPC网关(gateway)

只编写一次服务器代码，能够同时为gRPC和HTTP请求提供服务。

- gRPC网关是一个协议缓冲区(protobuf)插件。
- 根据protobuf生成代理代码。
- 将HTTP调用转换为gPRC
  - 进程内(in-process)翻译，意味着网关可以直接在代码中调用gRPC处理程序而无需通过网络上的任何额外跳跃点，但是它仅适用于一元gRPC。
  - 将HTTP网关作为单独的代理服务器运行，HTTP请求将被翻译并通过网络调用转发到gRPC服务器。
- 编写一次代码，同时满足gRPC和HTTP请求

## 四十、根据protobuf生成Go代码

### gRPC是如何工作的?

1. 定义API和数据结构
2. 生成gRPC存根
3. 实现服务
4. 客户端使用

### gRPC使用

安装 protobuf
```bash
apt install -y protobuf-compiler
```

查看版本
```bash
protoc --version
```

安装go插件
```bash
go install google.golang.org/protobuf/cmd/protoc-gen-go@v1.28
go install google.golang.org/grpc/cmd/protoc-gen-go-grpc@v1.2
```

查看版本
```bash
protoc-gen-go --version
protoc-gen-go-gprc --version
```

安装vscode插件
```bash
vscode-proto3
```

修复依赖
```
go mod tidy
```

## 四十一、运行gRPC服务以及调用API

1. 使用生成的代码运行gRPC服务器
2. 使用Evans客户端连接到gRPC服务器


### Evans 使用

安装 `evans`
```bash
go install github.com/ktr0731/evans@latest
```

使用 `evans` 连接 gRPC 服务
```bash
evans --host localhost --port 9090 -r repl
```

查看所有服务
```bash
show service
```

调用服务
```bash
call CreateUser
```

退出 `evans` 控制台
```bash
exit
```

## 四十二、实现创建和登录的gRPC接口

## 四十三、gRPC网关(Gateway)

编写一次代码，同时支持gRPC与HTTP服务。

安装依赖
```bash
go install \
    github.com/grpc-ecosystem/grpc-gateway/v2/protoc-gen-grpc-gateway \
    github.com/grpc-ecosystem/grpc-gateway/v2/protoc-gen-openapiv2 \
    google.golang.org/protobuf/cmd/protoc-gen-go \
    google.golang.org/grpc/cmd/protoc-gen-go-grpc
```

下载 `googleapis`
```bash
git clone https://github.com/googleapis/googleapis.git
```

复制文件
```bash
cp google/api/annotations.proto ../backend-master-class-golang/proto/google/api/annotations.proto
cp google/api/field_behavior.proto ../backend-master-class-golang/proto/google/api
cp google/api/http.proto ../backend-master-class-golang/proto/google/api
cp google/api/httpbody.proto ../backend-master-class-golang/proto/google/api
```

## 四十四、使用gRPC元数据(metadata)

元数据只是有关特定RPC调用的一些信息，以键值对的形式出现，它允许客户端提供一些与服务器调用相关的额外信息。

## 四十五、自动生成和提供Swagger文档

下载 `gprc-gateway`
```bash
git clone https://github.com/grpc-ecosystem/grpc-gateway
```

复制文件
```bash
mkdir -p proto/protoc-gen-openapiv2/options
cp protoc-gen-openapiv2/options/*.proto ../backend-master-class-golang/proto/protoc-gen-openapiv2/options/
```


下载 `swagger-ui`
```bash
git clone https://github.com/swagger-api/swagger-ui
```

复制文件
```bash
cp -r dist/* ../backend-master-class-golang/doc/swagger
```

## 四十六、在Go二进制文件中嵌入前端代码

`Statik` 允许将静态文件目录嵌入到Go可执行二进制文件中。

```bash
go install github.com/rakyll/statik
```

## 四十七、GRPC参数验证

## 四十八、运行数据库迁移

安装 `migrate`
```bash
go get github.com/golang-migrate/migrate/v4
```

## 四十九、使用SQLC处理部分更新以及可空字段

## 五十、gRPC可选参数

## 五十一、gRPC添加身份认证

## 五十二、gRPC结构化日志

安装 `zero-logger`
```bash
go get -u github.com/rs/zerolog/log
```

## 五十三、HTTP日志中间件

## 五十四、后台处理异步任务

### 异步处理

#### 使用 go routines
  
1. 后台处理任务
2. 容易实现
3. 可能丢失的任务

#### 使用消息队列

1. 任务同时保存在内存和持久存储中
2. 高可用: Redis哨兵和Redis集群
3. 没有任务丢失

### 发送验证邮件

1. 在数据库中创建新的用户记录
2. 将发送验证邮件任务推送到Redis队列
3. 后台worker从队列中提取任务并处理它

### Asynq

它是一个简单、可靠且高效的分布式任务队列库。使用后台工作程序异步排队并处理任务。它由redis提供支持，并设计为易于扩展。

```bash
go get -u github.com/hibiken/asynq
```

## 五十五、集成异步工作程序

查看redis是否启动正常
```bash
docker exec -it redis redis-cli ping
```

## 五十六、使用数据库事务向REDIS发送任务

## 五十七、异步工作程序处理错误日志

## 五十八、异步任务中延迟的重要性

## 五十九、发送邮件

安装 `email`
```bash
go get github.com/jordan-wright/email
```

## 六十、跳过测试

## 六十一、邮箱验证数据库设计与邮件发送

迁移表
```bash
migrate create -ext sql -dir db/migration -seq add_verify_emails
```

## 六十二、实现邮箱验证
