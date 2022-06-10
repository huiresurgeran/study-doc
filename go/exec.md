os/exec，用于运行外部命令以及和外部命令交互.

参考文档：https://colobu.com/2020/12/27/go-with-os-exec/#Pipe



# Command

最简单的运行一个外部命令的方式就是调用[Command](https://golang.org/pkg/os/exec/#Command)方法，需要传入要执行的程序和参数，它会返回一个*Cmd的数据结构，代表了一个执行的外部命令

```go
commandStr := fmt.Sprintf("./generate_archetype_code.sh -u %s -g %s -a %s -v %s -c %s ", userName, groupId, artifactId, version, archetype)

// exec.Command 函数返回一个 Cmd 对象
command := exec.Command("sh", "-c", commandStr)
```



# 命令类型

可以将命令的执行分为三种情况

1. 只执行命令，不获取结果

   `err := cmd.Run()`，返回的只有成功和失败，获取不到任何输出的结果

2. 执行命令，并获取结果（不区分stdout和stderr）
   `out, err := cmd.CombinedOutput()`，只返回out，不区分stdout和stderr

3. 执行命令，并获取结果（区分stdout和stderr）
   ```go
   var stdout, stderr bytes.Buffer
   cmd.Stdout = &stdout  // 标准输出
   cmd.Stderr = &stderr  // 标准错误
   err := cmd.Run()
   outStr, errStr := string(stdout.Bytes()), string(stderr.Bytes())
   ```



# 启动

```go
// 执行Cmd中包含的命令，阻塞直到命令执行完成
// Run方法会执行外部命令并等待命令完成，如果命令正常执行，没有错误，返回码为0,那么Run返回的err == nil
command.Run(),
// 执行Cmd中包含的命令，该方法立即返回，并不等待命令执行完成
command.Start()
```



# 输入和输出

`Cmd`命令包含输入和输出字段，你可以设置这些字段，实现定制输入和输出：

```go
Stdin io.Reader
Stdout io.Writer
Stderr io.Writer
```



## 1. Stdin

如果`Stdin`为空，那么进程会从null device(os.DevNull)中读取。
如果`Stdin`是*os.File对象，那么会从这个文件中读取。
如果`Stdin`是os.Stdin,那么会从标准输入比如命令行中读取数据。



## 2.Stdout

`Stdout`和`Stderr`代表外部程序进程的标准输出和错误输出，如果为空，那么输出到null device中。

如果`Stdout`和`Stderr`是*os.File对象，那么会往文件中输出数据。
如果`Stdout`和`Stderr`分别设置为`os.Stdout`、`os.Stderr`的话，会输出到命令行中。

```go
command := exec.Command("sh", "-c", commandStr)
// returns a pipe that will be connected to the command's standard output when the command starts
// 返回一个管道，该管道会在Cmd中的命令被启动后连接到其标准输出
stdout, err := command.StdoutPipe()
if err != nil {
   return err
} else {
   // defer用于资源的释放，会在函数返回之前进行调用
   defer stdout.Close()
}
```



## 3. Stderr

`Stdout`和`Stderr`代表外部程序进程的标准输出和错误输出，如果为空，那么输出到null device中。

如果`Stdout`和`Stderr`是*os.File对象，那么会往文件中输出数据。
如果`Stdout`和`Stderr`分别设置为`os.Stdout`、`os.Stderr`的话，会输出到命令行中。



```go
command := exec.Command("sh", "-c", commandStr)
// returns a pipe that will be connected to the command's standard error when the command starts
// 返回一个管道，该管道会在Cmd中的命令被启动后连接到其标准错误
stdErr, err := command.StderrPipe()
if err != nil {
   return err
} else {
   // defer用于资源的释放，会在函数返回之前进行调用
   defer stdErr.Close()
}
```



# Pipe

你可以将一个命令的输出作为下一个命令的输入，以此类推，将多个命令串成一个管道。

`os/exec`提供了`StderrPipe`、`StdinPipe`、`StdoutPipe`方法，获取管道对象。

比如下面的命令，将`cat main.go`的输出作为`wc -l`命令的输入:

```go
cmdCat := exec.Command("cat", "main.go")
catout, err := cmdCat.StdoutPipe()
if err != nil {
	log.Fatalf("failed to get StdoutPipe of cat: %v", err)
}

cmdWC := exec.Command("wc", "-l")
cmdWC.Stdin = catout
cmdWC.Stdout = os.Stdout

err = cmdCat.Start()
if err != nil {
	log.Fatalf("failed to call cmdCat.Run(): %v", err)
}

err = cmdWC.Start()
if err != nil {
	log.Fatalf("failed to call cmdWC.Start(): %v", err)
}

cmdCat.Wait()
cmdWC.Wait()
```

首先将管道创建好，然后依次调用各命令的Start方法和Wait方法。