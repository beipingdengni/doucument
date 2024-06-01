# 面向Java程序员的Go工程开发入门流程

参考博客：[面向Java程序员的Go工程开发入门流程](https://www.toutiao.com/article/7374616178674729506/?app=news_article&timestamp=1717208302&use_new_style=1&req_id=20240601101822C225008C8D17F0651B96&group_id=7374616178674729506&share_token=7A33CD18-24EC-4AC6-B944-58E843CE9688&tt_from=weixin&utm_source=weixin&utm_medium=toutiao_ios&utm_campaign=client_share&wxshare_count=1&source=m_redirect&wid=1717209677089)

## 假如拿到一台新电脑

> 安装brew： /bin/bash-c"$(curl-fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"

### ▐安装go编译器

brew install go

> 这将安装go的最新版本。在发文的这一刻，go的最新版本是1.22。
>
> 如果想要安装某个特定的go历史版本，可以使用

brew install go@1.21

> 这将安装go1.21
>
> 可以使用brew search go来查看当前brew仓库中可用的go版本列表。

安装完成后可以执行 go version 查看go是否安装成功。

### ▐安装go调试器

brew install delve

>  delve可以理解为是go版本的gdb。

brew install gdb

> 如果遇到delve搞不定的场景，就需要gdb出来救场了。

安装完成后可以执行 dlv version 查看delve是否安装成功，执行 gdb --version 查看gdb是否安装成功。

### ▐配置 go 在编译产物中附加上调试信息

在shell脚本（.bash_profile或.zshrc）中添加一行

export GOFLAGS="-ldflags=-compressdwarf=false"

### ▐配置github gfw代理

在shell脚本（.bash_profile或.zshrc）中添加一行

export GOPROXY=https://goproxy.io,direct

### ▐配置私有代码仓库

在shell脚本（.bash_profile或.zshrc）中添加一行 export GOPRIVATE="你的私有gitlab地址.com/*,另外的私有gitlab地址.com/*" ，不要带http或https前缀

## 安装ide

隆重推荐goland，真的好用，但是要花钱。 其次推荐idea安装go插件，但是要求idea是Ultimate版本。

不想花钱的话可以用vscode。vscode的必装插件：Go 认准Microsoft出品。

#### 工程结构

如果开发的是一个独立运行的程序（存在main函数），推荐的目录结构如下：

```
your_app_name/
├── cmd/
│   ├── your_app_name/            # 应用程序的入口点，包含main包
│   │   └── main.go               # 应用程序的主函数所在
│   └── ...                       # 如果有多个可执行文件，可以在这里添加
├── internal/                     # 可选，存放仅供本应用内部使用的包
│   └── ...                       # 内部包结构，比如业务逻辑、工具函数等
├── config/                       # 配置文件或配置加载逻辑
│   └── ...                       # 如yaml, toml, env等配置文件
├── scripts/                      # 构建、部署脚本等
│   └── ...                       
├── tests/                        # 单元测试、集成测试等
│   └── ...                       
├── README.md                     # 项目说明文档
├── go.mod                        # Go模块定义文件
├── go.sum                        # Go模块依赖的校验和文件
├── .gitignore                    # Git忽略规则文件
└── ...                           # 其他必要的文件，如Dockerfile、Makefile等
```

如果开发的是一个给别人用的类库，推荐的目录结构如下：

```
your_library_name/
├── internal/                      # 可选，用于存放不希望对外暴露的包
│   └── ...                        # 内部使用的包
├── pkg/                           # 存放库的公开对外接口。pkg 这一层可以省略，直接把源代码放在/your_library_name 目录下
│   └── your_library_name/
│       ├── your_library.go
│       └── ...                    # 库的各个源文件
├── tests/                        # 单元测试、集成测试等
│   └── ...                       
├── README.md                      # 项目说明文档，包括安装、使用方法等
├── go.mod                         # Go模块定义文件
├── go.sum                         # Go模块的校验和文件
├── .gitignore                     # Git忽略文件列表
└── ...                            # 其他必要的文件，如Dockerfile、Makefile等
```

## 依赖管理

go的依赖项通过 go.mod 文件描述，类似于java的 pom.xml 或py的 requirements.txt

go是源代码依赖，没有制品的概念。可以理解为每次编译时，go都会从git上面把所有的依赖项的源代码全部拉下来，放在一起编译。官方的源代码仓库就是github。

一个典型的go.mod示例如下：

```
module github.com/dapr/dapr
^^^ 声明该模块的名字，类似于 groupId:artifactId
    强烈建议在这里填写这个工程的实际可访问的git地址，否则会带来无穷无尽的麻烦

go 1.21
^^^ 指定要求的最低 go 版本，类似于 <java.version>1.8</java.version>

require (
^^^ 这里列出所有直接依赖项，类似于 <dependencies>
    contrib.go.opencensus.io/exporter/prometheus v0.4.2
    github.com/PaesslerAG/jsonpath v0.1.1
    github.com/PuerkitoBio/purell v1.2.1
)

require (
^^^ 这一大片后面带有 // indirect 的是 go 自动生成的所有间接依赖，类似 mvn dependency:tree 的产出。
    千万不要手工修改这部分。

    cloud.google.com/go v0.110.10 // indirect
    cloud.google.com/go/compute v1.23.3 // indirect
    cloud.google.com/go/compute/metadata v0.2.3 // indirect
    cloud.google.com/go/datastore v1.15.0 // indirect
    cloud.google.com/go/iam v1.1.5 // indirect
)

replace (
^^^ 这一段用于模块的名称和实际 git 地址不匹配的情况。
    上面提到的无穷无尽的麻烦就是指，所有引用你的包的项目都需要使用 replace 来将你的包指向实际的 git 地址
    github.com/toolkits/concurrent => github.com/niean/gotools v0.0.0-201512
    replace github.com/dapr/components-contrib => ../components-contrib
    ^^^ 也可以将模块的路径指向本地，以便于二方包开发过程中进行调试
)
```

当工程的多个依赖项发生冲突时，比如说app依赖lib1和lib2，lib1依赖libaaa的 1.0版本，lib2依赖libaaa的2.0版本，go不会进行仲裁，会直接报编译错。你需要在go.mod中手工指定libaaa到底应当使用哪个版本。

当依赖的二方包并没有放在github上，而是放在自建的gitlab上时，直接编译是编不过的，即使你有对应 gitlab 仓库的权限。

这是因为go在编译时并不是真的去直接拉git，而是会去尝试访问一个代理服务器（类似于cdn）。你需要指定go在访问我们自己的git时不要走官方的cdn，因为那上面肯定没有我们集团内部的包。

如果遇到这种情况，就需要在shell脚本（.bash_profile或.zshrc）中添加一行 export GOPRIVATE="你的私有gitlab地址.com/*,另外的私有gitlab地址.com/*"，这在第1节中已经讲过。

当依赖的二方包的owner修改了git的访问权限配置，导致你没有权限访问其代码仓库时，编译会失败。除了找项目owner开权限之外没有任何办法。

在代码中引用二方包

首先必须要说，go的import体系对于java程序员来说是非常难以上手的。主要体现在如下几个方面：

### ▐时刻注意区分清楚【模块 / module】和【包 / package】的概念

这个在java中根本不是问题，没人会弄混。但是在go中经常会造成混淆，因为这两者在go里面看起来长得差不多。以及，go的import语句和java的import有本质的区别，你更应当把go的import理解为c语言的#include。

【模块】指的是一个库，类似于一个jar包，其名称是groupId:artifactId对应的git地址，只在go.mod中出现。

模块的名称（通常）等同于其git地址，形式为github.com/groupId/artifactId

具体来说，一个典型的模块名大概类似于github.com/apache/dubbo-go，在域名后面只有两段（groupId 和 artifactId）

【包】指的是把相关联的文件放在一起的集合，类似于java概念中的package 

具体来说，一个在import语句后面跟的典型的包路径大概类似于 github.com/apache/dubbo-go/remoting 其中的 github.com/apache/dubbo-go 部分是模块名，后面的 /remoting 表示这个包所处的具体git路径。

再次强调，**请一定要抛开java的import语义，将go的import直接理解成#include**

如果遇到依赖相关的难以理解的编译错误，请检查是否误将模块路径放在了import语句里面，或者是不是把包路径误放在了go.mod里。

### ▐import包时是import包的git地址，但是这个地址和包的名称可以不一致

对于 java 程序员来说有一个冷知识：在常见的语言中，只有java强行规定了包名必须和目录结构一致、类名必须和文件名一致，而go并没有这条规定。所以import一个包时，这个包在实际代码中使用的名称和包的路径可以完全看不出任何关系。

从人道主义角度来讲，非常不推荐向别人提供这样的包，但也只是不推荐而已。

比如：

```
import (
    "github.com/go-playground/validator/v10"
)
...
func foo() {
    validator.Xxx()
}
```

引用这个包时需要依赖包的路径（"github.com/go-playground/validator/v10"），但是在具体使用的时候使用的包名是validator，这中间看不出任何对应关系，只能去翻源代码。
个人建议：**除了标准库的包之外，对所有import的包都自己定义一个别名**，例如：

```
import (
    validator "github.com/go-playground/validator/v10"
)
```

### ▐go在import中引入的包在代码中必须被使用，否则编译会报错

在ide中，默认会把没使用的包对应的import语句直接删掉。

所以如果临时注释掉一条语句，再回头把注释去掉时，很有可能因为import语句被删掉了而编译不过。

个人建议：**当临时需要禁用掉某些代码时，不要直接注释，要使用if false {} 给包起来。**

```
if false {
    validator.Xxx() // 不要直接注掉，免得ide把对应的import语句也给删了
}
```

### ▐修改go.mod后必须执行一次go mod tidy

当要新增/删除依赖时，需要修改go.mod文件，类似于修改pom.xml。在修改之后，需要手工执行 go mod tidy，否则会编译失败。go不会在编译时帮你更新依赖。

顺便，受上一条规则影响，所有的import都必须是实际被使用的。如果代码中去掉了某个import，而这一行import是整个工程中唯一引用某个依赖的位置，那么就意味着这个依赖不再被需要。此时需要从go.mod中去掉该依赖并执行 go mod tidy，否则会导致编译不通过。

个人建议：**直接把go mod tidy命令放在编译脚本中，每次编译时都执行一下。**下面【编译】一节会详细解释。

### ▐执行过go mod tidy之后， go.mod文件本身会被修改

go mod tidy会原地修改go.mod，把所有间接依赖也明确地标在原文件中。

如果go.mod中声明的依赖没有被实际使用，go mod tidy会直接把这一行require给删掉。

这一条规则尤其坑爹。前面提到了，go要求代码中所有import进来的包都必须被使用。如果你注掉了一行代码，很有可能导致IDE把对应的import删掉，然后go mod tidy会把go.mod中的依赖声明给删掉。这样你再把注释恢复回去之后，就需要从go.mod改起。

个人建议：**每次执行过 go mod tidy 之后都要记得检查一下go.mod文件，以免被go mod tidy命令改坏。**

以及，再重申一次，**千万不要碰go.mod里面标了 // indirect的那一片自动生成的require指令。**

最后再重复一遍之前提到过的个人建议：**当临时需要禁用掉某些代码时，不要直接注释，要使用if false {} 给包起来。**

向别人提供二方包和maven一样，当向别人提供二方包时，我们需要向用户提供二方包的GAV坐标。

其中GroupId、ArtifactId是被包含在git repo的地址中的，形式为 gitlab.mycompany.com/${groupId}/${artifactId}之前已经强调过，这个地址必须是可以被直接访问的，里面放的就是二方包的源代码，并且要配置好合适的访问权限。

不建议将git repo设置为除了PUBLIC之外的任何级别，除非你能明确地管控所有下游使用场景。否则但凡你的包被别人引用了，就永远会有不知道哪的人过来找你开权限。

Version比较特殊，我们分成正式包和snapshot包两种情况来讨论。

### ▐正式包的发版是通过git的tag机制来实现的

当一个版本开发完成时，我们需要在git repo里面把最新版本的代码打一个tag，名称需要遵循SemVer规范。

业务方可以使用下述语法在go.mod中依赖这个包：require gitlab.mycompany.com/${groupId}/${artifactId} v$tag

例如：require gitlab.mycompany.com/shop/libaaa v0.3.6-rc9

### ▐snapshot包的发版是通过git的branch机制实现的

当一个版本仍在持续开发过程中时，我们可以把正在开发的分支直接公布出去，业务方可以使用下述语法在go.mod中依赖这个包：

```
require gitlab.mycompany.com/${groupId}/${artifactId} v0.0.0-notexist
replace gitlab.mycompany.com/${groupId}/${artifactId} => gitlab.mycompany.com/${groupId}/${artifactId} v0.0.0-${branch}
```

例如：

```
require gitlab.mycompany.com/shop/libaaa v0.0.0-notexist
replace gitlab.mycompany.com/shop/libaaa => gitlab.mycompany.com/shop/libaaa v0.0.0-develop2
```

类似于maven的snapshot包，如果上游向分支里push了新代码，下游需要重新执行 go get -u（类似于mvn -U命令）更新所snapshot。

这里也可以看到，go的snapshot机制叠加上基于源代码的依赖机制，比mvn的 snapshot二方包要危险得多。你甚至可以向分支中提交一段编译不过的代码，从而让所有下游的编译过程全部炸掉。maven好歹总得先编译个jar包出来。

总结一下，如果我们开发好了一个二方包，需要提供给别人使用，那么需要公布两个信息：

1. 二方包的git repo地址，形式为: gitlab.mycompany.com/${groupId}/${artifactId}
2. 如果是正式包，需要公布其tag，使用SemVer格式；如果是snapshot，需要公布其branch，没有命名格式要求

编译

```
沧海月明珠有泪，蓝田日暖玉生烟。
此情可待成追忆，只是当时已惘然。
            —— （唐）李商隐
```

离开了maven的温室之后，我们必须直面go build的冰冷现实了。

go的编译命令是 go build。它有很多参数，非常灵活，但是每次编译都把完整命令全部敲一遍的话，精神方面可能就有点问题了。这个时候我们需要用make来简化构建工作，这又是一个大坑。

在java世界中，我们熟悉的构建工具就是Maven。Maven预先定义了 validate compile package testdeploy 之类的阶段，直接敲 mvn clean build 即可让 maven 完成预先定义的动作。

而make没有提供任何预定义的阶段，在敲下 make clean build 之后要执行什么操作都需要在Makefile中手工声明。这份Makefile应当放在工程根目录下，文件名即为 Makefile 八个字母，注意大小写。

这里仅给出一份个人推荐的Makefile文件样例，具体使用时请酌情修改：

```makefile
# 定义一些常量
APPNAME := your_application_name
BINDIR := /build/bin
GOBIN := $(shell go env GOPATH)/bin
export GO111MODULE := on

# 目标: 清理
clean:
  @rm -rf $(BINDIR)/*
  @echo "Cleanup completed."

# 目标: 构建
build: go_mod_tidy compile

# 执行go mod tidy
go_mod_tidy:
  @go mod tidy
  @echo "go mod tidy completed."

# 编译项目
compile: 
  @mkdir -p $(BINDIR)
  @go build -o $(BINDIR)/${APPNAME}
  @echo "Build completed."

# 使用帮助信息
help:
  @echo "Usage:"
  @echo "make build - 整理模块依赖，编译项目，将产出物放到 /build/bin 目录下"
  @echo "make clean - 清理构建产物"

.PHONY: clean build go_mod_tidy compile help
```

- make build 整理模块依赖，编译项目，将产出物放到 /build/bin目录下
- make clean 清理构建产物
- make help 输出上述说明

#### 调试

首先需要重申一个常识，对于go这种编译型语言来说，“调试”这个动作是面向可执行程序，即编译的产物的，而不是面向源代码的。

如果是在vscode中调试，需要修改项目 /.vscode目录中的launch.json文件，以指定待调试的可执行程序的位置、运行该程序时需要传递的参数和需设置的环境变量等信息。

```
{
    "version": "0.2.0",
    "configurations": [
        {
            "name": "Launch Go Program with Args",
            "type": "go",
            "request": "launch",
            "mode": "debug",
            "program": "${fileDirname}", // 当前文件所在的目录作为工作目录，如果你需要指定特定的可执行文件路径，可以改为该路径
            "args": ["arg1", "arg2", "--option=value"], // 在这里添加你的命令行参数
            "env": {"env1":"env_value1", "env2":"env_value2"}, // 可以在这里添加环境变量，如果需要
            "showLog": true, // 是否在调试控制台显示Delve的日志，默认为false
            "trace": "verbose" // 设置为"verbose"可以获取更详细的调试信息，根据需要调整
        }
    ]
}
```

实际的调试过程无需赘述。只能说vscode + go插件又不是不能用，其提供的断点和watch功能相比，goland来说还是有些差距。

<img src="https://p3-sign.toutiaoimg.com/tos-cn-i-6w9my0ksvp/ab91699f87554ed1b11f508f7490181d~noop.image?_iz=58558&from=article.pc_detail&lk3s=953192f4&x-expires=1717814579&x-signature=DC3RALGOl4Uz90SFVigI4dGgFNs%3D" alt="img" style="zoom:50%;" />



如果发现调试过程非常跳跃甚至不可用，请检查一下是否在编译选项中配置了 -ldflags=-compressdwarf=false，这个参数用于在编译产物中保留debug信息。

如果因为各种原因，无法在vscode环境内进行调试，我们可以尝试使用delve命令行工具来进行调试。

调试命令为 dlv exec xxxx -- 要传给xxx的命令行参数，要记得它调试的是编译出来的可执行文件。

delve提供的是基于命令行的交互方式。使用exec命令启动时，整个程序不会开始运行，而是会停在整个main函数的入口外面。

此时可以敲命令与程序进行交互，常用的命令如下：

- b 命令 设置断点
- c 命令 执行至下一个断点
- n 命令 step over
- s 命令 step in
- so 命令 step out
- args 命令 查看方法参数
- p 命令 查看变量的值

<img src="https://p3-sign.toutiaoimg.com/tos-cn-i-6w9my0ksvp/7ac09067d166459897a2a3b0073d1554~noop.image?_iz=58558&from=article.pc_detail&lk3s=953192f4&x-expires=1717814579&x-signature=H59ItOqWU4gB3NxlvyHpaxuzARc%3D" alt="img" style="zoom:50%;" />



delve同样支持 dlv debug xxx.go 的语法，可以直接指定源文件，此时它会调用go编译器把这个文件编译出来之后再去调试。对于特别小的工程来说较为方便。

如果delve也搞不定，就只能上gdb了。gdb具体的用法我就不讲了，因为我也不熟。

#### 打包

脱离了java/maven后，我们需要重新熟悉一下linux世界中“正统”的软件分发方式。

集团最初选择了将各系统运行在centos上，也就意味着集团的软件包分发是基于rpm/yum体系的。与之类似的还有Debian系的deb/apt、Archlinux使用的 tar.xz/pacman之类。

我们需要将我们的构建产物打成rpm包（类似于jar包），具体打rpm包的过程请学习 RPM打包操作手册。