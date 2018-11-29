### 常用命令
#### run
用于编译并运行 go 文件
```angularjs
go run main.go 
```

#### 安装 gocode，godef，guru
1、安装 gocode，自动补全守护程序
```angularjs
go get -u github.com/nsf/gocode
```

2、安装 dodef
```angularjs
go get -u github.com/rogpeppe/godef
```

3、安装 guru
```angularjs
go get -u golang.org/x/tools/cmd/guru
```

#### xorm
1、安装 xorm
```angularjs
go get github.com/go-xorm/xorm
```

2、安装 xorm 命令行工具
```angularjs
go get github.com/go-xorm/cmd/xorm
```

#### 基于 make 编译 go 工程

1、进入 GOPATH 目录下
```angularjs
go env
```

2、在 src 目录下的 github.com 目录下创建用户的目录，然后拉取代码，进行编译。
```angularjs
mkdir f1yegor
git clone https://github.com/f1yegor/clickhouse_exporter.git
cd clickhouse_exporter/
make build
```


