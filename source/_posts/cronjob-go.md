---
title: 'Go: Cron Job'
categories:
  - Programming
  - Go
  - 3P Framework
tags:
  - Go
  - k8s
  - cron
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

