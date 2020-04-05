# [beego和bee安装问题解决](https://www.cnblogs.com/aguncn/p/11930588.html)

如果使用go mod模式，直接安装bee时会报错：

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
go: github.com/beego/bee imports
github.com/beego/bee/cmd imports
github.com/beego/bee/cmd/commands/dlv imports
github.com/derekparker/delve/service: github.com/derekparker/delve@v1.3.1: 
parsing go.mod:
module declares its path as: github.com/go-delve/delve
                but was required as: github.com/derekparker/delve
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

 

主要参考URL：

http://www.678fly.cn/d/3-go-mod-bee

# 一、创建一个 go mod 下的文件夹

```
mkdir test
cd test
go mod init test
```

# 二、在 go.mod 内把 bee 的源替换掉，如下所示：

github.com/realguan/bee 是我 fork 了 github.com/beego/bee 的源码，进行了源代码更改：

```
module test
replace github.com/beego/bee v1.10.0 => github.com/realguan/bee v1.12.1
go 1.12
```

# 三、开始正式安装 beego 和 bee

```
export GOPROXY=https://goproxy.io	
go get -u github.com/astaxie/beego
go get -u github.com/beego/bee
```

以上如果没报错，那就成功了。

# 四、接下来测试：

```
cd test
bee new hello	// 创建项目
cd src/hello	// 进入项目目录
go mod init hello
bee run	// 大功告成
module demo` `replace github.com/beego/bee v1.10.0 => github.com/realguan/bee v1.12.1` `go` `1.13` `require (``  ``github.com/astaxie/beego v1.12.0 ``// indirect``  ``github.com/beego/bee v1.10.0 ``// indirect``  ``github.com/cosiner/argv v0.0.1 ``// indirect``  ``github.com/``go``-delve/delve v1.3.2 ``// indirect``  ``github.com/gorilla/websocket v1.4.1 ``// indirect``  ``github.com/konsorten/``go``-windows-terminal-sequences v1.0.2 ``// indirect``  ``github.com/lib/pq v1.2.0 ``// indirect``  ``github.com/mattn/``go``-colorable v0.1.4 ``// indirect``  ``github.com/mattn/``go``-isatty v0.0.10 ``// indirect``  ``github.com/mattn/``go``-runewidth v0.0.6 ``// indirect``  ``github.com/peterh/liner v1.1.0 ``// indirect``  ``github.com/shiena/ansicolor v0.0.0-20151119151921-a422bbe96644 ``// indirect``  ``github.com/sirupsen/logrus v1.4.2 ``// indirect``  ``go``.starlark.net v0.0.0-20191113183327-aaf7be003892 ``// indirect``  ``golang.org/x/arch v0.0.0-20191101135251-a0d8588395bd ``// indirect``  ``golang.org/x/crypto v0.0.0-20191122220453-ac88ee75c92c ``// indirect``  ``golang.org/x/net v0.0.0-20191125084936-ffdde1057850 ``// indirect``  ``golang.org/x/sys v0.0.0-20191120155948-bd437916bb0e ``// indirect``  ``google.golang.org/appengine v1.6.5 ``// indirect``  ``gopkg.in/yaml.v2 v2.2.7 ``// indirect``)
```

　　

```

```