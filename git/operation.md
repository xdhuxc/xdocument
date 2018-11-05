### github

#### 常用命令

#### 切换分支
1、使用本地 git 命令创建新分支
```angular2html
git branch xdhuxc
```
2、切换到新分支，master 分支上的内容自动同步
```angular2html
git checkout xdhuxc
```
3、将新分支发布到 github 上
```angular2html
git push origin xdhuxc
```

#### git 将一个本地工程推向多个远程仓库
推送时，可以同时推送至多个远程仓库
1、添加另外一个远程仓库
```angularjs
git remote set-url --add origin git@github.com:xdhuxc/aws-python.git
```
2、推送
```angularjs
git push origin master:master
```


