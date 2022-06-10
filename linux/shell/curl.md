```shell
if [ $(curl -sL -w "%{http_code}" $url -o $temp_file_name) = 200 ]; then
	echo $temp_file_name
fi
```



# -s

-s参数将不输出错误和进度信息。

$ curl -s https://www.example.com

上面命令一旦发生错误，不会显示错误信息。不发生错误的话，会正常显示运行结果。



## -L

-L参数会让 HTTP 请求跟随服务器的重定向。curl 默认不跟随重定向。

$ curl -L -d 'tweet=hi' https://api.twitter.com/tweet



# -o

-o参数将服务器的回应保存成文件，等同于wget命令。

$ curl -o example.html https://www.example.com

上面命令将www.example.com保存成example.html。



## -w

-w参数返回响应码

--write-out '%{http_code}' to the command line and curl will print the HTTP_CODE to stdout