---
title: "Go: Cron Job"
categories: [Programming, Go, 3P Framework]
tags: [Go, k8s, cron]
mathjax: false
date: 2026-04-10T15:41:31.000Z
---

<!-- placeholder -->
<!-- more -->

# `Go`: 定时任务实现方案

## `K8s CronJob`

- `k8s`的基础能力较强，通过`k8s`的`CronJob`，可以实现稳定可靠的定时任务，`Go`的实现也较为简单
- 使用`k8s`适用于大部分没有复杂依赖的定时任务，包括数据库定时更新、日志转储、消息定时推送等
- 可以在`Go`中进行简单的抽象，实现使用一个容器负责所有任务的定时执行，通过类似`profile`以及`expr`区分不同任务的执行时机
- 也可以不引入标签，把每个任务暴露为子命令，由`CronJob`直接覆盖容器的`args`来指定执行的任务

### 简单示例

- `Job`抽象

  ```go
  package job

  type Job interface {
  	Run() error
  	Profile() string
  }

  var jobs []Job

  // RegisterJob register a job that will be detected once the app start
  func RegisterJob(j Job) {
  	jobs = append(jobs, j)
  }
  ```

- `Job`执行

  ```go
  package job

  import (
  	"fmt"
  	"os"
  	"strings"
  	"sync"

  	"github.com/expr-lang/expr"
  )

  func Run() {
  	var (
  		wg      sync.WaitGroup
  		profile = os.Getenv("PROFILE")
  		env     = map[string]bool{}
  	)
  	for p := range strings.SplitSeq(profile, ",") {
  		env[strings.TrimSpace(p)] = true
  	}
  	for _, j := range jobs {
  		wg.Add(1)
  		go func(j Job) {
  			defer wg.Done()
  			program, err := expr.Compile(j.Profile(),
  				expr.Env(env),
  				expr.AllowUndefinedVariables())
  			if err != nil {
  				fmt.Println(err.Error())
  				return
  			}
  			output, err := expr.Run(program, env)
  			if err != nil {
  				fmt.Println(err.Error())
  				return
  			}
  			if match, ok := output.(bool); ok && match {
  				if err := j.Run(); err != nil {
  					fmt.Println(err.Error())
  				}
  			}
  		}(j)
  	}
  	wg.Wait()
  }
  ```

- `Job`实现

  ```go
  package jobs

  import (
  	"fmt"

  	"example.com/m/job"
  )

  func init() {
  	job.RegisterJob(&UserDataUpdaterJob{})
  }

  type UserDataUpdaterJob struct{}

  func (j *UserDataUpdaterJob) Run() error {
  	fmt.Println("aaa")
  	return nil
  }

  func (j *UserDataUpdaterJob) Profile() string {
  	return "user_data_updater"
  }
  ```

- 容器入口：`init()`注册依赖包被导入，之后只需要执行一次`job.Run()`并退出，由`k8s`负责按周期创建`Pod`

  ```go
  package main

  import (
  	"example.com/m/job"
  	_ "example.com/m/jobs"
  )

  func main() {
  	job.Run()
  }
  ```

### `CronJob`资源配置

- 上述容器对应一份`CronJob`资源，通过环境变量`PROFILE`指定本次触发的任务：

  ```yaml
  apiVersion: batch/v1
  kind: CronJob
  metadata:
    name: example-jobs
  spec:
    schedule: "0 3 * * *" # 每天 3 点触发
    timeZone: Asia/Shanghai # k8s 1.27+ 支持, 不设置时按 UTC 解释
    concurrencyPolicy: Forbid
    successfulJobsHistoryLimit: 3
    failedJobsHistoryLimit: 1
    jobTemplate:
      spec:
        backoffLimit: 2
        activeDeadlineSeconds: 600
        template:
          spec:
            restartPolicy: Never
            containers:
              - name: jobs
                image: example.com/jobs:latest
                env:
                  - name: PROFILE
                    value: "user_data_updater,log_rotator"
  ```

- 常用字段：
  - `schedule`：标准`cron`表达式，最小粒度为分钟；`k8s 1.27+`支持`timeZone`字段指定时区
  - `concurrencyPolicy`：上一次任务还未结束时又到了触发时机的策略，`Allow`（默认，允许并存）、`Forbid`（跳过本次）、`Replace`（终止旧任务并替换）
  - `startingDeadlineSeconds`：错过了触发时机（如调度器不可用）时，允许延迟启动的最大秒数，超过则跳过
  - `successfulJobsHistoryLimit`与`failedJobsHistoryLimit`：保留的已完成任务数量，用于排查问题，避免`Job`与`Pod`无限堆积
  - `jobTemplate.spec.backoffLimit`：单次任务失败后的重试次数
  - `jobTemplate.spec.activeDeadlineSeconds`：单次任务的超时时间，防止任务挂死

### `cron`表达式

- `schedule`使用`5`字段的标准`cron`表达式，从左至右依次为：

  ```text
  ┌───────── 分钟 (0-59)
  │ ┌───────── 小时 (0-23)
  │ │ ┌───────── 日 (1-31)
  │ │ │ ┌───────── 月 (1-12)
  │ │ │ │ ┌───────── 星期 (0-7, 0 与 7 都是周日)
  │ │ │ │ │
  * * * * *
  ```

- 月与星期也可以使用名称，如`JAN-DEC`、`SUN-SAT`
- 特殊字符：
  - `*`：任意值
  - `,`：枚举多个值，如`0,30`
  - `-`：连续范围，如`1-5`
  - `/`：步进，如`*/15`表示每`15`分钟
- 常见示例：
  - `0 3 * * *`：每天`3:00`
  - `*/30 * * * *`：每`30`分钟
  - `0 3 * * 1-5`：工作日的`3:00`
  - `0 0 1 * *`：每月`1`号的`0:00`
  - `30 8 * * 6`：每周六的`8:30`
- 一个容易踩坑的语义：当日与星期都被限制（都不是`*`）时，只要其中一个匹配即触发，是`OR`的关系而非`AND`；只想限制其中一个时，另一个必须写成`*`
- 最小粒度为`1`分钟，秒级调度只能由进程内`cron`（`WithSeconds()`）承担

### `args`子命令方式

- 更贴近`k8s`习惯的方式是不引入标签，把每个任务暴露为子命令，入口按`os.Args`分发（实际项目通常用`cobra`注册子命令，还可以为不同任务声明独立的参数）：

  ```go
  package main

  import (
  	"log"
  	"os"

  	"example.com/m/jobs"
  )

  func main() {
  	if len(os.Args) < 2 {
  		log.Fatal("missing job name")
  	}
  	switch os.Args[1] {
  	case "user-data-updater":
  		if err := (&jobs.UserDataUpdaterJob{}).Run(); err != nil {
  			log.Fatal(err)
  		}
  	default:
  		log.Fatalf("unknown job %s", os.Args[1])
  	}
  }
  ```

- 此时`CronJob`通过覆盖`args`指定本次执行的任务，一个镜像配合多份几乎相同的资源清单即可：

  ```yaml
  containers:
    - name: jobs
      image: example.com/jobs:latest
      args: ["user-data-updater"]
  ```

- 与`PROFILE`方式的对比：
  - 任务内容直接体现在资源清单中，`kubectl get cronjob -o yaml`即可看出该`CronJob`执行什么；`PROFILE`则需要事先了解标签约定
  - 每个`CronJob`只跑一个任务，重试、超时与历史记录的粒度更清晰；`PROFILE`可以把多个小任务合并在一次触发中执行，节省冷启动开销
  - `PROFILE`配合`expr`可以表达组合条件（如`nightly && prod`）；`args`是一对一的显式指定
  - 两者不冲突，也可以用`args`选择任务、用`PROFILE`补充环境分组
- 多份相似的`CronJob`清单不需要手工维护，可以用`Helm`或`kustomize`模板化，或通过`CI/CD`的矩阵批量生成

## 进程内定时任务

- 对于与服务同生命周期的任务，例如需要共享内存状态、连接池的任务，更适合在进程内调度，通常使用[`github.com/robfig/cron/v3`](https://github.com/robfig/cron)

  ```go
  package main

  import (
  	"github.com/robfig/cron/v3"
  )

  func main() {
  	c := cron.New() // 默认 5 字段表达式, 与 k8s 一致
  	c.AddFunc("0 3 * * *", func() { /* ... */ })
  	c.AddFunc("@every 10m", func() { /* ... */ })
  	c.AddJob("@daily", &UserDataUpdaterJob{})
  	c.Start()
  	defer c.Stop()
  	select {}
  }
  ```

- `cron.New(cron.WithSeconds())`：使用`6`字段表达式，支持秒级精度，`k8s`做不到这一点
- 预定义表达式：`@hourly`、`@daily`、`@weekly`、`@monthly`，以及`@every <duration>`（如`@every 1h30m`）
- `cron.WithChain(cron.Recover(logger))`：通过中间件链为任务注入`panic`恢复与日志
- `c.Stop()`返回一个`context`，等待正在执行的任务结束后才会取消，适合优雅退出：

  ```go
  ctx := c.Stop()
  <-ctx.Done()
  ```

## 多副本问题

- `CronJob`每次触发只会创建一个`Pod`，天然单实例，不存在重复执行
- 进程内`cron`随`Deployment`水平扩容会为每个副本都调度一次，需要额外处理：
  - 任务本身幂等，允许重复执行（例如覆盖式的缓存刷新）
  - 保持单副本并使用`Recreate`更新策略，避免滚动更新期间新旧副本并存
  - 使用分布式锁竞争执行权，例如`redis`的`SET NX`：

    ```go
    ok, err := rdb.SetNX(ctx, "lock:user_data_updater", hostID, 10*time.Minute).Result()
    if err != nil || !ok {
    	return nil // 其他副本已在执行
    }
    defer rdb.Del(ctx, "lock:user_data_updater")
    ```

## 方案对比

| 维度     | `K8s CronJob`               | 进程内`cron`             |
| -------- | --------------------------- | ------------------------ |
| 触发精度 | 分钟级                      | 秒级                     |
| 部署形态 | 独立`Pod`，与业务服务解耦   | 随服务部署，共享进程     |
| 资源开销 | 每次触发冷启动，需重建连接  | 常驻内存，复用连接与状态 |
| 多副本   | 天然单实例                  | 需要分布式锁或单副本     |
| 失败处理 | `k8s`负责重试并保留历史记录 | 需要自行实现重试与记录   |

- 总体而言：无状态、可独立重试的任务交给`K8s CronJob`，把调度、重试和可观测性都交给基础设施；强依赖服务内部状态的任务留在进程内，但要处理好多副本下的重复执行问题
