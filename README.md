# 🐹 Go 面试 200 问（中文带答案）

> Go 面试一站通关：200 道高频面试题全中文带答案，基础/进阶/实战三级分级，从语法到并发到 Goroutine 到微服务，一册在手，Go 岗面试无忧。

![stars](https://img.shields.io/badge/⭐-给个Star吧-red)
![license](https://img.shields.io/badge/license-MIT-blue)
![status](https://img.shields.io/badge/status-持续更新-green)

## 为什么需要这份题库？

你是不是也有这样的经历：

- Go 语法会写，面试却被问 **Goroutine 调度/内存逃逸** 问懵了？
- 网上搜 Go 面试题，**要么是英文的、要么只有题目没答案**？
- 项目用 Go 开发，但一问 **channel/锁/GC 原理** 就答不上来？

**Go 面试，拼的不是会写多少代码，而是对并发模型和运行时的理解。**

Goroutine 调度、channel 通信、内存逃逸、垃圾回收、GMP 模型……这些才是面试官真正想考的。

这份题库把 Go 面试的**高频考点一次性整理全**：

- ✅ **200 道高频题**：覆盖面试 90% 以上的考点
- ✅ **全中文带答案**：每道题都有标准答案+解析
- ✅ **三级分级**：基础（必考）/进阶（拉差距）/实战（拿 offer）
- ✅ **真实代码示例**：不是空谈理论，直接看代码
- ✅ **持续更新**：紧跟 Go 1.2x 新特性

## 📚 题库结构

### 第一篇：基础篇（80 题，必考区）

#### 一、语法基础（30 题）

**1. Go 的特点是什么？**

**答案要点：**
- 静态类型 + 编译型，性能接近 C
- 原生并发：Goroutine + channel（CSP 模型）
- 垃圾回收自动管理内存
- 编译快、部署简单（单二进制）
- 标准库强大（net/http 等）

**2. `var` 和 `:=` 的区别？**

**答案要点：**
- `var`：显式声明，可用于包级变量
- `:=`：简短声明，只能在函数内使用，自动推断类型
```go
var name string = "张三"  // 显式声明
age := 25                  // 简短声明（函数内）
```

**3. 数组和切片的区别？**

**答案要点：**
- 数组：长度固定，值类型，赋值会拷贝整个数组
- 切片：长度可变，引用类型，底层是指向数组的指针+长度+容量
- 切片扩容：cap 不够时翻倍（小切片）或 1.25 倍（大切片）

**4. `make` 和 `new` 的区别？**

**答案要点：**
- `new`：分配内存，返回指针（零值），适用于任意类型
- `make`：只用于 slice/map/channel，返回初始化后的值（非指针）
```go
p := new(int)        // *int，值为 0
s := make([]int, 5)  // []int，len=5 cap=5
m := make(map[string]int)
```

**5. map 使用要注意什么？**

**答案要点：**
- 未初始化的 map 是 nil，写入会 panic
- map 不是并发安全的，并发写要加锁或用 sync.Map
- 遍历顺序随机，不能依赖
- 删除不存在的 key 不报错

**6. 什么是内存逃逸？**

**答案要点：**
- 变量本该在栈上分配，却逃逸到堆上
- 逃逸原因：返回局部变量指针、闭包引用、interface 装箱、大对象
- 影响：增加 GC 压力，性能下降
- 用 `go build -gcflags="-m"` 查看逃逸分析

**7. defer 的执行时机？**

**答案要点：**
- defer 在函数返回时执行，**后进先出**（栈）
- 参数在 defer 语句处求值（不是执行时）
- 常用来释放资源、recover 异常
```go
func f() {
    defer fmt.Println("1")  // 后执行
    defer fmt.Println("2")  // 先执行
    // 输出 2 1
}
```

**8. panic 和 recover？**

**答案要点：**
- panic：抛出异常，中断当前函数，逐层向上传递
- recover：捕获 panic，只能在 defer 中调用有效
- 没有 recover 的 panic 会导致程序崩溃
```go
func safe() {
    defer func() {
        if r := recover(); r != nil {
            fmt.Println("恢复:", r)
        }
    }()
    panic("出错了")
}
```

**9. 字符串是不可变的吗？怎么高效拼接？**

**答案要点：**
- 字符串不可变，修改会生成新字符串
- 高效拼接：strings.Builder（比 + 快很多，避免频繁分配）
- 大量拼接时用 Builder，少量用 + 或 fmt.Sprintf

**10. interface 是什么？空接口呢？**

**答案要点：**
- interface 定义一组方法，实现这些方法即实现该接口（隐式实现）
- 空接口 `interface{}`（Go 1.18+ 可用 `any`）：接受任何类型
- 类型断言：`v, ok := i.(string)`

（以下 20 题见仓库完整版）

#### 二、并发编程（30 题）

**31. Goroutine 是什么？**

**答案要点：**
- Goroutine 是 Go 的轻量级线程，栈初始 2KB，可动态增长
- 由 Go 运行时调度（不是操作系统线程），百万级并发轻松
- 创建：`go func() {}()`
- 特点：创建快（微秒级）、切换开销小

**32. GMP 模型是什么？**

**答案要点：**
- G：Goroutine（用户态协程）
- M：Machine（操作系统线程）
- P：Processor（本地队列，默认=CPU 核数）
- 调度流程：G 放入 P 的本地队列 → M 从 P 取 G 执行 → G 阻塞时 M 找其他 G
- M 不够时创建新 M；P 本地队列空时从全局队列或其他 P 偷（work stealing）

**33. channel 是什么？**

**答案要点：**
- channel 是 Goroutine 间通信的管道（CSP 模型）
- 类型：无缓冲（同步）和有缓冲（异步）
- 无缓冲：发送和接收必须同时准备好，否则阻塞
- 有缓冲：缓冲区满才阻塞发送，空才阻塞接收
- 关闭：`close(ch)`，关闭后读取返回零值；向已关闭的 channel 发送会 panic

```go
ch := make(chan int, 3)   // 有缓冲
ch <- 1                   // 发送
v := <-ch                 // 接收
```

**34. 有缓冲和无缓冲 channel 的区别？**

**答案要点：**
- 无缓冲：同步通信，收发必须配对，用于保证执行顺序
- 有缓冲：异步通信，发送方不阻塞直到缓冲区满，用于解耦

**35. 怎么避免死锁？**

**答案要点：**
- 不要在主 Goroutine 里等子 Goroutine 而没有接收方
- 用 select + default 做非阻塞操作
- 用 context 超时控制
- 用 sync.WaitGroup 等所有 Goroutine 完成
```go
var wg sync.WaitGroup
for i := 0; i < 3; i++ {
    wg.Add(1)
    go func(n int) {
        defer wg.Done()
        fmt.Println(n)
    }(i)
}
wg.Wait()
```

**36. 什么是 data race？怎么解决？**

**答案要点：**
- 多个 Goroutine 同时读写同一变量，结果不确定
- 检测：`go run -race` / `go build -race`
- 解决：加锁（sync.Mutex/RWMutex）、channel、原子操作（sync/atomic）

**37. Mutex 和 RWMutex 的区别？**

**答案要点：**
- Mutex：互斥锁，读读也互斥
- RWMutex：读写锁，读读共享、写写互斥、读写互斥
- 读多写少场景用 RWMutex 性能更好
- 注意：锁要成对使用，小心死锁

**38. sync.WaitGroup 的用法？**

**答案要点：**
- 等待一组 Goroutine 完成
- Add(delta) 增加计数、Done() 减一（defer 调用）、Wait() 阻塞到 0
- 注意：Add 要在 go 语句前调用，避免计数器为负 panic

**39. sync.Once 的用法？**

**答案要点：**
- 保证函数只执行一次（并发安全）
- 场景：单例初始化、懒加载
```go
var once sync.Once
var instance *Config

func GetConfig() *Config {
    once.Do(func() {
        instance = loadConfig()
    })
    return instance
}
```

**40. context 的作用？**

**答案要点：**
- 传递取消信号、超时控制、请求级元数据
- context.Background()：根 context
- WithCancel / WithTimeout / WithDeadline / WithValue
- 超时控制：
```go
ctx, cancel := context.WithTimeout(context.Background(), 3*time.Second)
defer cancel()
select {
case <-ctx.Done():
    fmt.Println("超时了")
case <-ch:
    fmt.Println("收到数据")
}
```

（以下 20 题见仓库完整版）

#### 三、语言特性（20 题）

**61. Go 的垃圾回收原理？**

**答案要点：**
- Go 使用三色标记-清除算法 + 并发 GC
- 三色：白（未标记）/灰（待处理）/黑（已处理）
- 写屏障：防止并发时漏标
- 通过 GOGC（默认 100）控制触发频率

**62. 什么是 struct 的方法集？**

**答案要点：**
- 值接收者：值类型和指针类型都可调用
- 指针接收者：只有指针类型可调用
- 要修改结构体字段，必须用指针接收者

（以下 18 题见仓库完整版）

### 第二篇：进阶篇（80 题，拉差距区）

#### 四、网络编程（20 题）

**81. net/http 标准库怎么用？**

**答案要点：**
```go
func main() {
    http.HandleFunc("/hello", func(w http.ResponseWriter, r *http.Request) {
        fmt.Fprintf(w, "Hello, Go!")
    })
    http.ListenAndServe(":8080", nil)
}
```

**82. HTTP 服务的性能优化？**

**答案要点：**
- 设置合理的超时（ReadTimeout/WriteTimeout）
- 连接复用、连接池
- 中间件拆分（日志/限流/鉴权）
- 用 pprof 定位性能瓶颈

（以下 18 题见仓库完整版）

#### 五、存储与数据库（20 题）

**101. Go 操作 MySQL？**

**答案要点：**
- 用 database/sql + 驱动（go-sql-driver/mysql）
- sql.Open 不建立连接，db.Ping() 才建立
- 查询：db.Query / db.QueryRow，注意 defer rows.Close()
- 防注入：用 ? 占位符，不用拼接 SQL

**102. Redis 在 Go 里怎么用？**

**答案要点：**
- go-redis / redigo 库
- 连接池配置：MaxActive、MaxIdle、IdleTimeout
- 常用：Get/Set、分布式锁（SETNX）、缓存穿透/击穿/雪崩处理

（以下 18 题见仓库完整版）

#### 六、微服务与框架（20 题）

**121. 什么是微服务？Go 微服务常用框架？**

**答案要点：**
- 微服务：把单体拆成独立部署的小服务，各自独立扩展
- Go 框架：Gin、Echo、Kratos、Go-zero、Dubbo-go
- 治理：注册发现（etcd/consul/nacos）、配置中心、熔断限流、链路追踪

**122. Gin 框架的核心？**

**答案要点：**
- 路由：Radix 树匹配，性能高
- 中间件：洋葱模型（请求进入→中间件→处理→返回）
- 参数绑定：ShouldBindJSON
- 路由组：分组管理

（以下 18 题见仓库完整版）

#### 七、Go 特性深入（20 题）

**141. 泛型是什么？怎么用？**

**答案要点：**
- Go 1.18+ 支持泛型
- 类型参数：[T any]、[T comparable]
- 类型约束：interface { ~int | ~string }
```go
func Max[T comparable](a, b T) T {
    if a > b { return a }
    return b
}
```

**142. 什么是嵌入（embedding）？**

**答案要点：**
- 结构体嵌入：类似继承，但 Go 用组合
- 嵌入的字段和方法会被提升
- 实现"继承"的效果，但更灵活

（以下 18 题见仓库完整版）

### 第三篇：实战篇（40 题，拿 offer 区）

#### 八、手写代码（15 题）

**161. 手写一个并发安全的计数器？**

**答案要点：**
```go
type Counter struct {
    mu    sync.Mutex
    count int
}

func (c *Counter) Inc() {
    c.mu.Lock()
    defer c.mu.Unlock()
    c.count++
}

func (c *Counter) Value() int {
    c.mu.Lock()
    defer c.mu.Unlock()
    return c.count
}
```

**162. 手写一个工作池（Worker Pool）？**

**答案要点：**
```go
func workerPool(jobs <-chan int, results chan<- int, workers int) {
    var wg sync.WaitGroup
    for i := 0; i < workers; i++ {
        wg.Add(1)
        go func() {
            defer wg.Done()
            for job := range jobs {
                results <- job * 2
            }
        }()
    }
    wg.Wait()
    close(results)
}
```

（以下 13 题见仓库完整版）

#### 九、系统设计（15 题）

**176. 设计一个高并发短链服务？**

**答案要点：**
- 发号器（雪花算法/自增）→ base62 编码 → 短链
- 缓存（Redis）加速查询，DB 持久化
- 布隆过滤器防缓存穿透
- 限流保护

**177. 怎么设计一个分布式锁？**

**答案要点：**
- Redis SETNX + 过期时间（防死锁）
- Redlock（多节点）提高可靠性
- 续期机制（watchdog）
- 用 etcd 的租约也可以

（以下 13 题见仓库完整版）

#### 十、项目实战（10 题）

**191. Go 项目怎么组织目录？**

**答案要点：**
```text
project/
├── cmd/        # 入口
├── internal/   # 内部包（不可被外部导入）
├── pkg/        # 可复用包
├── api/        # API 定义
├── config/     # 配置
├── scripts/    # 脚本
└── go.mod
```

**192. 怎么优雅关闭服务？**

**答案要点：**
- 监听 SIGINT/SIGTERM 信号
- http.Server.Shutdown(ctx) 等待请求完成
- 关闭数据库连接池、消息队列连接

（以下 8 题见仓库完整版）

### 附：高频考点速记卡（30 题，面试前 10 分钟）

**200. 并发安全怎么保证？**

**速记：** 锁（Mutex/RWMutex）、channel、原子操作（sync/atomic），选型看场景：读多写少用 RWMutex，通信用 channel，计数器用 atomic。

（其余速记卡见仓库完整版）

## 🚀 快速开始

```bash
# 直接浏览
# 打开 README.md 即可查看全部内容

# 或者克隆到本地
git clone https://github.com/zieang88888/go-interview-zh.git
```

## 🤝 贡献

- 有好的面试题？提 PR
- 有想补充的考点？提 Issue
- 觉得有用？点个 ⭐ Star 就是最大的支持！

## 📄 License

MIT

---

**如果这份题库帮到了你，请点个 ⭐ Star，让更多准备 Go 面试的同学看到！** 🙏

**关注我 [@zieang88888](https://github.com/zieang88888)，持续输出中文开源干货！**