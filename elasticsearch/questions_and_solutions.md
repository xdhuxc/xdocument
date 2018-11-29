### 常见问题及解决
1、启动 elasticsearch 容器时，直接映射文件或目录时，报权限错误
```angular2html

```
问题原因说明：https://discuss.elastic.co/t/elastic-elasticsearch-docker-not-assigning-permissions-to-data-directory-on-run/65812/4

解决：先创建一个卷，然后再挂载该目录
```angular2html
docker volume create --opt device=/root/es/data --name=xdhuxc 
docker run -p 9200:9200 -v xdhuxc:/usr/share/elasticsearch/data docker.elastic.co/elasticsearch/elasticsearch:6.5.0
```
如果仍然出现错误，创建 elasticsearch 用户，给目录赋予权限，然后启动容器
```angular2html
groupadd -g 1000 elasticsearch                                                       
useradd -u 1000 -g 1000 elasticsearch  
chown -R 1000 es
chgrp -r 1000 es
```



