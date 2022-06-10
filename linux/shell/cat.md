cat（英文全拼：concatenate）命令用于连接文件并打印到标准输出设备上

```shell
if [ $(curl -sL -w "%{http_code}" $url -o $temp_file_name) = 200 ]; then
    echo $temp_file_name    
    cat $temp_file_name
    echo -e '\n下载链接 http://9.134.241.154/download?userName=${{BK_CI_START_USER_NAME}}&artifactId=${{artifactId}}'
    exit 0
else
```



输出$temp_file_name文件中所有的数据