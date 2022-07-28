

# 语法

```shell
if [ condition1 ]
then
	command1
elif condition2
then
	command2
else
	command3
fi
```



- `[ ]`，表示条件测试，空格很重要，`[`的后面和`]`的前面都必须要有空格
- `if then else fi`是分开的语句，在同一行需要用分号分隔开
- `if [ condition1 ]; the command1; else command2; fi`



# 变量表达式



## 整数

- -eq，等于
- -ne，不等于
- -gt，大于
- -ge，大于等于
- -lt，小于
- -le，小于等于



# 字符串

- =，字符串相等
- ==，字符串相等，等同于=
- !=，不等于

```shell
cmd_line="cmd"

if [ "$cmd_line" = "cmd" ]
then
	echo " == "
else
	echo " != "
fi
```



## 文件/文件夹（目录）

- -e filename，如果filename存在，则为真，判断文件是否存在
- -d filename，如果filename是目录，则为真
- -f filename，如果filename为常规文件，则为真，判断文件类型



```shell
if [ -e ${filename} ]
then
	echo "file is exist"
else
	echo "file is not exist"
fi
```

