


### 学习资料

https://www.cnblogs.com/davenkin/p/vagrant-virtualbox.html


Vagrantfile 

https://blog.csdn.net/u011781521/article/details/80291765

### vagrant 命令
1、初始化 vagrant 工程
```angular2
vagrant init xdhuxc centos/7
```

2、启动虚拟机
```angular2
vagrant up --provider virtualbox
```

3、登录虚拟机
```angular2
vagrant ssh 
```

4、关闭虚拟机
```angular2
vagrant halt
```

5、删除虚拟机
```angular2
vagrant destroy
```

### 注意事项
1、虚拟机的配置信息在 vagrantfile 中，使用 `vagrant up` 启动的虚拟机会重新创建网络等配置，因此最好在 vagrantfile 中指定这些信息。

### box 下载地址

https://app.vagrantup.com/centos/boxes/7