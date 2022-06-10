# os.stat(文件路径)

此方法用于获取文件的描述

描述信息如下：

```go
FileInfo：文件信息
			Name()，文件名
			Size()，文件大小，字节为单位
			IsDir()，是否是目录
			ModTime()，修改时间
			Mode()，权限
			Sys()，文件详细信息
```



## 使用方法

```go
fileInfo,err :=  os.Stat(fileName)
if err != nil{
	fmt.Println("err :",err)
	return
}
fmt.Printf("%T\n",fileInfo)
//文件名
fmt.Println(fileInfo.Name())
//文件大小
fmt.Println(fileInfo.Size())
//是否是目录
fmt.Println(fileInfo.IsDir()) //IsDirectory,true/false
//修改时间
fmt.Println(fileInfo.ModTime())
//权限
fmt.Println(fileInfo.Mode()) //-rw-rw-rw-

```



## 其他作用

可以用于判断文件或者文件夹是否存在

```go
func IsExists(fileName string) (bool, error) {
   // os.Stat(文件路径)获取文件的描述
   _, err := os.Stat(fileName)
   // 返回错误为nil，说明文件存在
   if err == nil {
      return true, err
   }
   // 返回的错误类型使用os.IsNotExist()判断为true,说明文件不存在
   if os.IsNotExist(err) {
      return false, err
   }
   // 返回错误为其他类型，不确定文件是否存在
   return false, err
}
```

