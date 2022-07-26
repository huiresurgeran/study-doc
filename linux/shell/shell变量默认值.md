# 变量默认值

## ${var:-defaultValue}

var没有定义时，使用defaultValue

var依然为空，没有改变值

```shell
# name没有定义，结果为空
echo ${name}
# toto =》name没有定义，使用默认值toto
echo ${name:-toto}
# 依然为空，没有改变name变量的值
echo ${name}
```



## ${var:=defaultValue}

var没有定义时，使用defaultValue

同时var也被赋值为defaultValue

```shell
# name没有定义，结果为空
echo ${name}
# toto =》name没有定义，使用默认值toto
echo ${name:=toto}
# name变量的值已经被改变
echo ${name}
```



## ${var:?value}

var没有定义时，或者定义了值为空时，会报错并且退出

用于检查是否初始化变量以及值是否为空

```shell
# name没有定义，结果为空
echo ${name}
# name没有定义，报错并退出
echo ${name:?toto}
# 变量为空或未设置
echo ${name:?}
# name变量的值已经被改变，正常输出
name=totoro
echo ${name}
```



## ${var:+value}

当var定义并且不为空，用value替换var的值，否则什么也不做

和var:-value相反

```shell
# name没有定义，结果为空
echo ${name:+toto}
# toto => name定义了并且不为空，替换为toto
name=totoro
echo ${name:+toto}
```



# 参考文档

https://blog.51cto.com/happytree007/1900383