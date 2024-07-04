# Linux

## Install Software

### JDK

#### install by yum

> 检查yum java 版本

```shell
yum -y list java*
```

> 安装jdk

```shell
yum install -y `java-1.8.0-openjdk-devel.x86_64`
```

> 安装完成后检查jdk版本

```shell
java -version
```

> 默认安装在 usr/lib/jvm 下

#### install by tar

> 手动安装jdk

1. 到官网下载对应jdk版本 url:https://www.oracle.com/java/technologies/downloads/
    ![image-20230630161713995](http://httpdeathsky.top//image-20230630161713995.png)

2. 将文件移动到/usr/local/jdk
    ```shell
    mv `jdk` /usr/local/jdk
    ```

3. 解压压缩包
    ```shell
    tar zxvf jdk-8u181-linux-x64.tar.gz
    ```

4. 配置环境变量
    ```shell
    vim /etc/profile
    ```

5. 在文件最后添加
    ```shell
    export JAVA_HOME=/usr/local/jdk/jdk1.8.0_181
    export CLASSPATH=$:CLASSPATH:$JAVA_HOME/lib/
    export PATH=$PATH:$JAVA_HOME/bin
    ```

6. 刷新环境变量
    ```shell
    source /etc/profile
    ```

7. 查看jdk版本
   
    ```shell
    java -version
    ```

### Nginx

### Gitea

### FileBrowser

### GitLab

### Docker

## 根目录文件夹

1. **/boot**：包含了系统启动时所需的文件，如内核和引导加载程序（bootloader）。这是启动过程的关键部分。

2. **/dev**：包含设备文件，用于访问和控制硬件设备，如磁盘驱动器、键盘、鼠标等。

3. **/etc**：存放系统的配置文件和目录，包括系统级别的配置和各种服务的配置。

4. **/home**：包含用户的主目录，每个用户都有一个独立的子目录，通常以用户名命名。

5. **/media**：用于挂载可移动媒体设备（如USB驱动器、光盘等）的挂载点。

6. **/mnt**：临时挂载点，用于手动挂载其他文件系统。

7. **/opt**：用于安装可选的软件包的目录，通常由用户或第三方应用程序使用。

8. **/proc**：包含虚拟文件系统，提供了有关系统和进程状态的信息。

9. **/program**：这不是一个标准的Linux根目录，通常是用户自定义的目录。如果在您的系统中有这个目录，它可能用于存放自己编写的程序或脚本。

10. **/root**：超级用户（root）的主目录，是系统管理员的家目录。

11. **/run**：包含系统运行时的临时文件，如进程ID文件和套接字文件。

12. **/srv**：用于存放服务的数据文件，如Web服务器、FTP服务器等。

13. **/sys**：包含与内核和系统硬件相关的虚拟文件系统。

14. **/tmp**：用于存放临时文件的目录，系统启动后会定期清空。

15. **/usr**：包含用户可执行程序、库文件、文档等，是系统的二进制文件和资源的主要位置。

16. **/var**：包含变化频繁的文件，如日志文件、数据库文件、缓存等。

这些根目录的组织有助于Linux系统的管理和维护，使文件和资源能够按照特定的目的进行组织和存放。不同的目录有不同的权限和用途，有助于保持系统的安全性和可维护性。
