---
title: Cobra - 一个 Golang 命令行项目生成工具
date: 2017-07-17 15:07:14
tags: Golang
thumbnail: https://cloud.githubusercontent.com/assets/173412/10886352/ad566232-814f-11e5-9cd0-aa101788c117.png
---

# Cobra - Golang 命令行库

### 简介：

Cobra 是一个创建 CLI 命令行的 golang 库。

### 组成：

Cobra 结构由三部分组成：命令( Command )、参数( Args )、标志( Flag )。

```go
type Command struct {
    Use   string  // The one-line usage message.
    Short string  // The short description shown in the 'help' output.
    Long  string  // The long message shown in the 'help<this-command>' output.
    Run   func(cmd *Command, args []string)  // Run runs the command.
}
```

前三个是不同场景下的说明，最后一个是要执行的函数。

### 安装

安装 Cobra 很简单，首先，用 `go get` 安装最新版本的库，这个命令会安装 Cobra 框架生成工具和依赖。

```go
go get -u github.com/spf13/cobra/cobra
```

然后，把 Cobra 添加到你的 app 中：

```go
import "github.com/spf13/cobra"
```

### 快速开始

​一般用 cobra 命令生成d的项目结构如下：

```go
 ▾ appName/
    ▾ cmd/
        add.go
        your.go
        commands.go
        here.go
      main.go
```

main 函数中非常简洁，只有一个目的：初始化 Cobra.

```go
package main

import (
	"fmt"
	"os"

	"{pathToYourApp}/cmd"
)

func main() {
	if err := cmd.RootCmd.Execute(); err != nil {
		fmt.Println(err)
		os.Exit(1)
	}
}
```

### 用 Cobra 工具生成项目框架

#### cobra init

`cobra init [yourAppName]` ，这个命令可以让你的程序有一个正确的结构，你立刻能够感受到 Cobra 带给你的快乐，你可以配置它自动生成你需要的开源协议。

```go
cobra init newAppName
```

newAppName 是你的项目名称。它会在你的 GOPATH 目录下面生成项目。

我们看一下 main.go 函数

```go
package main

import "cobra_exp1/cmd"

func main() {
    cmd.Execute()
}
```

main 调用 cmd.Execute()，那我们找到这个地方，cmd/root.go 文件：

```go
package cmd

import (
    "fmt"
    "os"

    "github.com/spf13/cobra"
    "github.com/spf13/viper"
)

var cfgFile string

// RootCmd represents the base command when called without any subcommands
var RootCmd = &cobra.Command{
    Use:   "cobra_exp1",
    Short: "A brief description of your application",
    Long: `A longer description that spans multiple lines and likely contains
examples and usage of using your application. For example:

Cobra is a CLI library for Go that empowers applications.
This application is a tool to generate the needed files
to quickly create a Cobra application.`,
// Uncomment the following line if your bare application
// has an action associated with it:
//    Run: func(cmd *cobra.Command, args []string) { },
}

// Execute adds all child commands to the root command sets flags appropriately.
// This is called by main.main(). It only needs to happen once to the rootCmd.
func Execute() {
    if err := RootCmd.Execute(); err != nil {
        fmt.Println(err)
        os.Exit(-1)
    }
}

func init() {
    cobra.OnInitialize(initConfig)

    // Here you will define your flags and configuration settings.
    // Cobra supports Persistent Flags, which, if defined here,
    // will be global for your application.

    RootCmd.PersistentFlags().StringVar(&cfgFile, "config", "", "config file (default is $HOME/.cobra_exp1.yaml)")
    // Cobra also supports local flags, which will only run
    // when this action is called directly.
    RootCmd.Flags().BoolP("toggle", "t", false, "Help message for toggle")
}

// initConfig reads in config file and ENV variables if set.
func initConfig() {
    if cfgFile != "" { // enable ability to specify config file via flag
        viper.SetConfigFile(cfgFile)
    }

    viper.SetConfigName(".cobra_exp1") // name of config file (without extension)
    viper.AddConfigPath("$HOME")  // adding home directory as first search path
    viper.AutomaticEnv()          // read in environment variables that match

    // If a config file is found, read it in.
    if err := viper.ReadInConfig(); err == nil {
        fmt.Println("Using config file:", viper.ConfigFileUsed())
    }
}
```

我们看到 Execute() 函数中调用 RootCmd.Execute()，RootCmd 是开始讲组成 Command 结构的一个实例。

我们运行看看：

```sh
> go run main.go 
A longer description that spans multiple lines and likely contains
examples and usage of using your application. For example:

Cobra is a CLI library for Go that empowers applications.
This application is a tool to generate the needed files
to quickly create a Cobra application.
```

空空如也，还什么也没有，那接下来我们来添加一些子命令。

#### cobra add

这个命令用来创建子命令，子命令就是像下面这样：

* app serve
* app config
* app config create

在你项目的目录下，运行下面这些命令：

```go
cobra add serve
cobra add config
cobra add create -p 'configCmd'
```

这样以后，你就可以运行上面那些 app serve 之类的命令了。项目目录如下：

```Sh
  ▾ app/
    ▾ cmd/
        serve.go
        config.go
        create.go
      main.go
```

然后再运行程序：

```Sh
❯ go run main.go 
A longer description that spans multiple lines and likely contains
examples and usage of using your application. For example:

Cobra is a CLI library for Go that empowers applications.
This application is a tool to generate the needed files
to quickly create a Cobra application.

Usage:
  llleon [command]

Available Commands:
  config      A brief description of your command
  help        Help about any command
  serve       A brief description of your command

Flags:
      --config string   config file (default is $HOME/.llleon.yaml)
  -h, --help            help for llleon
  -t, --toggle          Help message for toggle

Use "llleon [command] --help" for more information about a command.
```

现在我们有了三个子命令，并且都可以使用，然后只要添加命令逻辑就能真正用了。

#### Flag

cobra 有两种 flag，一个是全局变量，一个是局部变量。全局什么意思呢，就是所以子命令都可以用。局部的只有自己能用。先看全局的：

```go
RootCmd.PersistentFlags().StringVar(&cfgFile, "config", "", "config file (default is $HOME/.cobra_exp1.yaml)")
```

在看局部的：

```go
RootCmd.Flags().BoolP("toggle", "t", false, "Help message for toggle")
```

区别就在 RootCmd 后面的是 Flags 还是 PersistentFlags。

好了，入门教程到此结束，感兴趣的童鞋可以到 [Cobra](https://github.com/spf13/cobra) 深入研究一番。