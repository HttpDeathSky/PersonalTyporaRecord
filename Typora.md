# Typora同步

[TOC]

## 软件下载

> typora
>
> ​	http://httpdeathsky.top:8080/share/28p_o5dh
>
> git
>
> ​	https://git-scm.com/downloads

## 安装服务

### FileBrowser

> 开源文件管理，支持文件上传下载，文件分享

```shell
docker run -d \
--restart always \
-p 8080:80 \
--name filebrowser \
-v /program/files:/srv \
-v /program/filebrowserconfig/filebrowserconfig.json:/etc/config.json \
-v /program/filebrowserconfig/database.db:/etc/database.db \
filebrowser/filebrowser
```

### Gitea

> 开源轻量级Git，相比Gitlab，减少冗余功能，体积小，运行配置要求低，同时具备大部分所需功能

```shell
docker run -d \
--name gitea \
--restart always \
-p 11122:22 \
-p 13000:3000 \
-v /program/dockerconfig/giteaconfig/data:/data \
gitea/gitea:latest
```

## Typora图片自动上传

> 进入Typora偏好设置(ctl+,)，图像

![image-20230627104729005](http://httpdeathsky.top//image-20230627104729005.png)

> 命令

```
@chcp 65001 >nul & cmd /d/s/c D:\Software\Git\bin\bash.exe D:\File\ShellFile\upload_file.sh
```

> D:\Software\Git\bin\bash.exe：自己安装的git路径下的bin/bash.exe
>
> D:\File\ShellFile\upload_file.sh：文件上传脚本
>
> 通过查阅Typora官方文档：https://support.typora.io/Upload-Image/#custom

![image-20230627105312994](http://httpdeathsky.top//image-20230627105312994.png)

> 脚本执行最后需要输出此结构，并附带图片地址，同时脚本需要支持多文件上传

> upload_file.sh文件内容

```shell
#!/bin/bash

# Define variables
servername="httpdeathsky.top"
username="root"
password=""
private_key="D:\File\Keys\id_rsa"
remote_path="/program/files/images"
url_prefix="http://httpdeathsky.top/"

# Create an array to store the URLs of successfully uploaded images
declare -a arr

# Loop over all arguments
for filepath in "$@"; do
    # Only process non-.md files
    if [[ $filepath != *.md ]]; then
        # Upload the file, suppressing scp's output
        scp -i "$private_key" "$filepath" "${username}@${servername}:${remote_path}"
		# > /dev/null 2>&1 //use to hide tran print
		#scp -i "$private_key" "$filepath" "${username}@${servername}:${remote_path}" > /dev/null 2>&1

        # Check if the upload was successful
        if [ $? -eq 0 ]; then
            # If the upload was successful, store the URL in the array
            arr+=("${url_prefix}/$(basename $filepath)")
        fi
    fi
done

# Check if all uploads were successful
if [ ${#arr[@]} -eq $# ]; then
    echo 'Upload Success:'
    for url in "${arr[@]}"; do
        echo $url
    done
else
    echo 'Upload Failed'
fi

#@chcp 65001 >nul & cmd /d/s/c D:\Software\Git\bin\bash.exe D:\File\ShellFile\upload_file.sh
```

> 复制图片到typora后，将自动上传图片至目标服务器，获取返回的图片远程url并替换原始路径

## Gitea配置

> 启动gitea服务后，访问ip/域名+端口进入首次配置页面
>
> 修改红框内容并设置管理员账号

![image-20230627111252932](http://httpdeathsky.top//image-20230627111252932.png)

> 成功启动后创建仓库，于本地打开gitbash执行git初始化

![image-20230627143159571](http://httpdeathsky.top//image-20230627143159571.png)

> 后续通过gitbash进行文件同步，也可以使用gui解决版本冲突，具体可以参考官方文档
>
> https://git-scm.com/book/en/v2

## 图片测试

![05f17432fe5b51ce8c060a13760b38e7](http://httpdeathsky.top//05f17432fe5b51ce8c060a13760b38e7.jpeg)

> 2023/06/27
>
> version:1.0
