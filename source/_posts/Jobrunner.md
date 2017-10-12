---
title: Jobrunner 源码解析
Date: 2017-06-27 11:07:35
Auther: jsharkc
---

## 安装

```go
go get github.com/bamzi/jobrunner
```

## 使用

#### eg1

```Go
package main

import "github.com/bamzi/jobrunner"

func main() {
    jobrunner.Start()
    jobrunner.Schedule("@every 5s", ReminderEmails{})
  	select{}
}

type ReminderEmails struct {}

func (e ReminderEmails) Run() {
    fmt.Printf("Every 5 sec send reminder emails \n")
}
```

***解析：***在 main() 函数中，jobrunner 是导入的包, jobrunner.Start() 代表启动，jobrunner.Schedule() 代表添加定时 job，第一个参数是时间格式，第二个是任务实例，任务必须实现 Run() 方法。

Jobrunner 支持两种时间格式：

* "@XXX"

  1. "@yearly", "@annually" 每年
  2. "@monthly" 每月
  3. "@weekly" 每星期
  4. "@daily", "@midnight" 每天
  5. "@hourly" 每小时
  6. "@every 5s" 支持 "ns", "us" (or "µs"), "ms", "s", "m", "h"。

* "x x x x x x"

  从左到右依次代表 :

  1．秒  

  2．分钟  

  3．小时 

  4．月份中的日期

  5．月份

  6．星期中的日期



| 名            | 支持的数字           | 支持的字符   |
| ------------ | --------------- | ------- |
| Seconds      | 0-59            | - * /   |
| Minutes      | 0-59            | - * /   |
| Hours        | 0-23            | - * /   |
| Day-of-month | 1-31            | - * ? / |
| Month        | 1-12 or JAN-DEC | - * /   |
| Day-of-Week  | 1-7 or SUN-SAT  | - * ? / |

**解释 :**

1. \* 是一个通配符，表示任何值，用在Minutes字段中表示每分钟。  
2. ? 只可以用在 ***day-of-month*** 或者 ***Day-of-Week*** 字段中，用来表示不指定特殊的值。  
3. \- 用来表示一个范围，比如10-12用在Month中表示10到12月。  
4. , 用来表示附加的值，比如 ***MON,WED,FRI*** 在 day-of-week 字段中表示***礼拜一和礼拜三和礼拜五***。  
5. / 用来表示增量，比如0/15用在Minutes字段中表示从0分开始0和15和30和45分。  

***e.g.***

* "0  1  0  1  1-12  ?" 表示每月1号0点1分执行。  
* "0  0  21  ?  \*  1" 表示每个礼拜天21点0分执行。  
* "0  0  0  \*  \*  ?" 表示每天0点0分执行。  
* "0  \*  22  \*  \*  ?" 表示每天22点开始每分钟  
* "0  \*  0-23  \*  \*  ?" 表示每天每分钟 

## 源码解析

#### jobrunner 开始函数 Start()

```go
func Start(v ...int) {
	MainCron = cron.New()

	if len(v) > 0 {
		if v[0] > 0 {
			workPermits = make(chan struct{}, v[0])
		} else {
			workPermits = make(chan struct{}, DEFAULT_JOB_POOL_SIZE)
		}
	}

	if len(v) > 1 {
		if v[1] > 0 {
			selfConcurrent = true
		} else {
			selfConcurrent = false
		}
	}

	MainCron.Start()

	fmt.Printf("%s[JobRunner] %v Started... %s \n",
		magenta, time.Now().Format("2006/01/02 - 15:04:05"), reset)

}
```

**解析：**Start() 接受不定长参数，可以不传，可以传多个，但实际上最多前两个参数有效。首先，创建一个 Cron 定时任务实例 MainCron。如果传入了 > 0 个参数，并且第一个参数 > 0，则用此参数初始化 workPermits 参数，此参数用于控制最多同时运行的 job 数量，如果 <= 0，则用默认 DEFAULT_JOB_POOL_SIZE 常量。如果传入了 > 1 个参数，并且第二个参数 > 0，则用此参数初始化 selfConcurrent 参数，此参数用于控制同一个 job 是否可以并行执行，如果 <= 0，则不可并行。最后，MainCron 开始执行，并打印开始时间。

#### 添加任务的四种方式

##### 1. Schedule(spec string, job cron.Job)

```go
func Schedule(spec string, job cron.Job) error {
	sched, err := cron.Parse(spec)
	if err != nil {
		return err
	}
	MainCron.Schedule(sched, New(job))
	return nil
}
```

**解析：**第一个参数是表示什么时间执行，第二个参数就是任务。首先调用 cron 的Parse 解析函数，判断是否有错误，成功的话添加到 MainCron 中准备执行。

##### 2.Every(duration time.Duration, job cron.Job)

```go
func Every(duration time.Duration, job cron.Job) {
	MainCron.Schedule(cron.Every(duration), New(job))
}
```

**解析：**这个方法是以固定时间间隔执行任务。

##### 3.Now(job cron.Job) 

```go
func Now(job cron.Job) {
	go New(job).Run()
}
```

**解析：**新起一个协程立即运行一次。

##### 4.In(duration time.Duration, job cron.Job)

```go
func In(duration time.Duration, job cron.Job) {
	go func() {
		time.Sleep(duration)
		New(job).Run()
	}()
}
```

**解析：**在第一个参数的时间后，执行一次。

#### job 结构

```go
type Job struct {
	Name    string        // inner 类型名称
	inner   cron.Job      // 实现 Run() 方法的 type
	status  uint32        // 状态数
	Status  string        // 状态解释
	Latency string        // job 运行所用时间
	running sync.Mutex    // 锁
}
```

#### 新建一个 job

```go
func New(job cron.Job) *Job {
	name := reflect.TypeOf(job).Name()
	if name == "Func" {
		name = UNNAMED
	}
	return &Job{
		Name:  name,
		inner: job,
	}
}
```

**解析：**用反射获取一下传入的 job 的类型，并新建一个 Job 。

##### 更新 job 状态

```go
func (j *Job) StatusUpdate() string {
	if atomic.LoadUint32(&j.status) > 0 {
		j.Status = "RUNNING"
		return j.Status
	}
	j.Status = "IDLE"
	return j.Status
}
```

**解析：**以原子方式获取 job.status 的状态数，如果 >0 ，设置 job.Status = “RUNNING”, 否则 job.Status = “IDLE”

##### job 运行

```go
func (j *Job) Run() {
	start := time.Now()
	// If the job panics, just print a stack trace.
	// Don't let the whole process die.
	defer func() {
		if err := recover(); err != nil {
			var buf bytes.Buffer
			logger := log.New(&buf, "JobRunner Log: ", log.Lshortfile)
			logger.Panic(err, "\n", string(debug.Stack()))
		}
	}()

	if !selfConcurrent {
		j.running.Lock()
		defer j.running.Unlock()
	}

	if workPermits != nil {
		workPermits <- struct{}{}
		defer func() { <-workPermits }()
	}

	atomic.StoreUint32(&j.status, 1)
	j.StatusUpdate()

	defer j.StatusUpdate()
	defer atomic.StoreUint32(&j.status, 0)

	j.inner.Run()

	end := time.Now()
	j.Latency = end.Sub(start).String()

}
```

**解析：**先获取一下当前时间，作为任务开始时间记录下来，并添加 defer recover 防止崩溃，并打印错误信息。如果 selfConcurrent 为 false ，不能同一个任务并行执行，则加锁，一个一个执行。否则可以同时执行。如果 workPermits 不为空，则限制同时最大执行数量。以原子方式把 job.status 存储为 1，更新一下状态，以 defer 延迟修改 job.status 存储为 0，并更新状态。最后，run job。