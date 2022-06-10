fmt包实现了类似C语言printf和scanf的格式化I/O。

分为向外输出内容和获取输入内容两大部分



# 向外输出

## 1. Print

`Print`函数会将内容输出到系统的标准输出

- Print，直接输出内容
- Printf，支持格式化：`fmt.Printf("我是：%s\n", name)`
- Println，输出内容的结尾添加一个换行符



## 2. Fprint

`Fprint`系列函数会将内容输出到一个`io.Writer`接口类型的变量`w`中，我们通常用这个函数往文件中写入内容。

只要满足`io.Writer`接口的类型都支持写入

- Fprint
- Fprintf
- Fprintln

```go
// 向标准输出写入内容
fmt.Fprintln(os.Stdout, "向标准输出写入内容")
```

```go
// 向打开的文件句柄中写入内容
fileObj, err := os.OpenFile("./xx.txt", os.O_CREATE|os.O_WRONLY|os.O_APPEND, 0644)
if err != nil {
	fmt.Println("打开文件出错，err:", err)
	return
}
name := "沙河小王子"
fmt.Fprintf(fileObj, "往文件中写如信息：%s", name)
```

```go
// 向网络流写入内容
func Generate(w http.ResponseWriter, r *http.Request) {
  userName := strings.TrimSpace(r.FormValue("userName"))
	if userName == "" {
		w.WriteHeader(500)
		fmt.Fprintf(w, "please enter userName")
		return
	}
}
```



## 3. Sprint

`Sprint`系列函数会把传入的数据生成并返回一个字符串。

- print
- Sprintf
- Sprintln

```go
var FilePath = "tmp/%s/%s.tar.gz"
var userName = "JSamuel"
var artifactId = "java-demo"
fileName := fmt.Sprintf(FilePath, userName, artifactId)

// fileName: tmp/JSamuel/java-demo.tar.gz
```



# 获取输入



# 参考文档

https://www.liwenzhou.com/posts/Go/go_fmt/

