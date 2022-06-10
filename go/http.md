参考文档：https://laravelacademy.org/post/21659

# 响应

## http头

### 1. Content-Disposition

指示回复的内容以何种形式展示

- inline：内联：网页或者页面的一部分，Content-Disposition: inline
- attachment，附件：下载并保存到本地，
  - Content-Disposition: attachment
  - Content-Disposition: attachment; filename="filename.jpg"

```go
w.Header().Set("Content-Disposition", "attachment; filename="+artifactId+".tar.gz")
```



### 2. Content-Type

告诉客户端实际返回的内容的内容类型

参考文档：https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Headers/Content-Type

- html：Content-Type: text/html; charset=utf-8
- .tar.gz：x-compressed-tar

```go
w.Header().Set("Content-Type", "application/x-compressed-tar")
```



## 响应状态码

```go
w.WriteHeader(500)
```



## 返回数据

```go
fileName := fmt.Sprintf(FilePath, userName, artifactId)
file, err := ioutil.ReadFile(fileName)
if err != nil {
	fmt.Fprintf(w, err.Error())
	return
}

// 返回文本字符串
w.Write(file)
```



# 请求



# 监听端口

```go
func main() {
   http.HandleFunc("/generate", service.Generate)
   http.HandleFunc("/download", service.Donwload)

   s := &http.Server{
      Addr:         ":8099",
      ReadTimeout:  10 * time.Second,
      WriteTimeout: 60 * time.Second,
   }

   // 打印输出内容
   // 退出应用程序
   // defer函数不会执行
   log.Fatal(s.ListenAndServe())
}
```