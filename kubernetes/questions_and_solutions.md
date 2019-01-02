1、使用 kubectl 命令时，要求输入用户名和密码

解决：删除~/.kube目录下的config文件。


2、 kubernetes 禁用 Swap 的原因

当前的 QOS 策略都是假定主机不启用内存 Swap。如果主机启用了 Swap，那么 QOS 策略可能会失效。

例如，两个 Pod 都刚好达到了内存限制上限，由于内存 Swap 机制，它们还可以继续申请使用更多内存，如果 Swap 空间不足，那么最终这两个 Pod 中的进程可能会被杀掉。

目前 Kubernetes 和 Docker 尚不支持内存 Swap 空间的隔离机制。