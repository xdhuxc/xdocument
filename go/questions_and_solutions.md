1、使用 *restful.Request.ReadEntity 方法读取 JSON 格式数据到结构体中时，出现如下错误：
```angular2html
json: cannot unmarshal string into Go struct field Receiver.resolved of type bool
```
原因：输入格式有错误，将 JSON 格式中 bool 值写成了 字符串 true，改为 JSON 格式的 bool 值即可解决该问题。