### 利用结构体进行转换

go 对 json 的解析函数在 encoding/json 里面，主要是编码和解码两个函数

json.Marshal 函数将会递归遍历整个对象，依次按成员类型对这个对象进行编码，类型转换规则如下：

go 数据类型 | JSON 数据类型
---|---
bool | Boolean
int float | Number
string | JSON String 带 ""
struct | JSON Object 再根据成员递归打包
array或slice | JSON Array
[]byte | base64 编码后的 JSON String
map | JSON Object，key 必须是 string
interface{} | 按照内部实际进行转换
nil | null
channel和func | UnsupportedTypeError

 需要注意：go 只能将 json 数据解析到结构体中首字母是大写的变量中，结构体中首字母是小写的变量将被忽略。
 json 格式是大小写字母不敏感的，解析过程中，golang 将会自动匹配。
 
 
 ### 参考资料
 1、使用 encoding/json 编解码
 
 https://blog.csdn.net/u011304970/article/details/70769949
 