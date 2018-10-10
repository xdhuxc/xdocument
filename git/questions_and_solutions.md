### 常见问题及解决

1）拉取大工程

使用 git 拉取大工程时，报错如下：
```
xdhuxc@DESKTOP-AFGTEDSW MINGW64 /d/PycharmProject
$ git clone -b private git@github.com:xdhuxc/xpython.git                                                                                                               .git
Cloning into 'xpython'...
remote: Counting objects: 4608, done.
remote: Compressing objects: 100% (71/71), done.
fatal: pack has bad object at offset 10852763787: inflate returned 1
fatal: index-pack failed
```
解决方法如下：
```angularjs
$ git clone -b private --depth=1 git@github.com:xdhuxc/xpython.git  
Cloning into 'xpython'...
remote: Counting objects: 913, done.
remote: Compressing objects: 100% (756/756), done.
remote: Total 913 (delta 143), reused 819 (delta 101)
Receiving objects: 100% (913/913), 1.79 GiB | 10.33 MiB/s, done.
Resolving deltas: 100% (143/143), done.
Checking out files: 100% (798/798), done.
```

2）拉取代码时报错

使用 IDEA 复制 GitHub 上的代码时，提示错误如下：
```
Fetch failed: Could not read from remote repository.
```
在 IDEA 中，点击 “File”，选择 “Settings”，找到 “Version Control”，选择 “Git”，修改 “SSH executable” 为 “Native”，重新拉取代码即可。
