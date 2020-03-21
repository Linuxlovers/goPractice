# 第1节 go项目目录

刚开始看go语言的时候 我们经常会疑惑 项目目录是怎么情况，项目路径怎么规划，导包怎么导

项目目录结构如何组织，一般语言都是没有规定。但Go语言这方面做了规定，这样可以保持一致性，做到统一、规则化比较明确。

#### 1、一般的，一个Go项目在GOPATH下，会有如下三个目录：

```
|--bin

|--pkg

|--src
```

其中，bin存放编译后的可执行文件；pkg存放编译后的包文件；src存放项目源文件。

一般，bin和pkg目录可以不创建，go命令会自动创建（如 go install），只需要创建src目录即可。

对于pkg目录，曾经有人问：我把Go中的包放入pkg下面，怎么不行啊？他直接把Go包的源文件放入了pkg中。

这显然是不对的。pkg中的文件是Go编译生成的，而不是手动放进去的。（一般文件后缀.a）

对于src目录，存放源文件，Go中源文件以包（package）的形式组织。通常，新建一个包就在src目录中新建一个文件夹。

#### 2、举例说明

比如：我新建一个项目，myfirst，开始的目录结构如下：

```
myfirst--
    |--src
```

为了编译方便，增加了一个install文件，目录结构：

```
myfirst
    
    |—install.bat
    |—src
```

之所以加上这个install.bat，是不用配置GOPATH（避免新增一个GO项目就要往GOPATH中增加一个路径）

接下来，增加一个包：config和一个main程序。目录结构如下：

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
myfirst
    |—instal.bat
    |—src
        |-- config    
            |-- config.go
|—myfirst        
            |-- main.go
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

[![image](https://images2015.cnblogs.com/blog/10966/201603/10966-20160310165932069-1323627039.png)](http://images2015.cnblogs.com/blog/10966/201603/10966-20160310165930804-2008554900.png)

注意，config.go中的package名称必须最好和目录config一致，而文件名可以随便。main.go表示main包，文件名建议为main.go。（注：不一致时，生成的.a文件名和目录名一致，这样，在import 时，应该是目录名，而引用包时，需要包名。例如：目录为myconfig，包名为config，则生产的静态包文件是：myconfig.a，引用该包：import “myconfig”，使用包中成员：config.LoadConfig()）

onfig.go和main.go的代码如下：
config.go代码

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
package config


func LoadConfig() {



}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

[![image](https://images2015.cnblogs.com/blog/10966/201603/10966-20160310165933132-958238658.png)](http://images2015.cnblogs.com/blog/10966/201603/10966-20160310165932804-1047384176.png)

main.go代码

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
package main


import (
    
    "config"
    
    "fmt"

)


func main() {
    
    config.LoadConfig()
    
    fmt.Println("Hello, GO!")

}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

#### [![image](https://images2015.cnblogs.com/blog/10966/201603/10966-20160310165933913-1534793964.png)](http://images2015.cnblogs.com/blog/10966/201603/10966-20160310165933585-165152776.png)

install.bat配置说明：

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
@echo off

setlocal

if exist install.bat goto ok
echo install.bat must be run from its folder
goto end

: ok

set OLDGOPATH=%GOPATH%
set GOPATH=%~dp0

gofmt -w src

go install myfirst

:end
echo finished
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

[![image](https://images2015.cnblogs.com/blog/10966/201603/10966-20160310170607382-1611485406.png)](http://images2015.cnblogs.com/blog/10966/201603/10966-20160310170606475-18221860.png)

注：冒号和ok之间不应该有空格。

打开命令行，找到d:\myfirst目录，输入install，执行如下：

[![image](https://images2015.cnblogs.com/blog/10966/201603/10966-20160310170608319-251542272.png)](http://images2015.cnblogs.com/blog/10966/201603/10966-20160310170607913-9657041.png)

[![image](https://images2015.cnblogs.com/blog/10966/201603/10966-20160310170610569-1724711572.png)](http://images2015.cnblogs.com/blog/10966/201603/10966-20160310170610085-603062237.png)

执行完之后，生成pkg目录：

[![image](https://images2015.cnblogs.com/blog/10966/201603/10966-20160310170611757-479662295.png)](http://images2015.cnblogs.com/blog/10966/201603/10966-20160310170611147-333084680.png)

命令行提示：config.loadConfig未定义。

经过反复重试，install.bat修改如下：

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
@echo off

setlocal

if exist install.bat goto ok
echo install.bat must be run from its folder
goto end

: ok

set OLDGOPATH=%GOPATH%
set GOPATH=%cd%

gofmt -w src

go install myfirst

set GOPATH=OLDGOPATH

:end
echo finished
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

config.go修改如下：

```
package config

func LoadConfig() {

}
```

函数名称为大写开头。

[![image](https://images2015.cnblogs.com/blog/10966/201603/10966-20160310170614241-1273632500.png)](http://images2015.cnblogs.com/blog/10966/201603/10966-20160310170613085-1461026442.png)