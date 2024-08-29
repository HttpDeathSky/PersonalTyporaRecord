# Linux

## Software

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

### Gitea

### FileBrowser

### GitLab

### Docker

## 文件夹含义

1. **/boot**：包含了系统启动时所需的文件，如内核和引导加载程序（bootloader）。这是启动过程的关键部分。
2. **/dev(*device*)**：包含设备文件，用于访问和控制硬件设备，如磁盘驱动器、键盘、鼠标等。
3. **/etc(Editable Text Config)**：存放系统的配置文件和目录，包括系统级别的配置和各种服务的配置。
4. **/home**：包含用户的主目录，每个用户都有一个独立的子目录，通常以用户名命名。
5. **/media**：linux 系统会自动识别一些设备，例如USB驱动器、光盘等等，当识别后，Linux 会把识别的设备挂载到这个目录下。
6. **/mnt**：临时挂载点，用于手动挂载其他文件系统。我们可以将光驱挂载在 /mnt/ 上，然后进入该目录就可以查看光驱里的内容了。
7. **/opt(optional)**：用于安装可选的软件包的目录，通常由用户或第三方应用程序使用。
8. **/proc(processes)**：/proc 是一种伪文件系统（也即虚拟文件系统），存储的是当前内核运行状态的一系列特殊文件，这个目录是一个虚拟的目录，它是系统内存的映射，我们可以通过直接访问这个目录来获取系统信息。包含虚拟文件系统，提供了有关系统和进程状态的信息。
9. **/program**：这不是一个标准的Linux根目录，通常是用户自定义的目录。如果在您的系统中有这个目录，它可能用于存放自己编写的程序或脚本。
10. **/root**：超级用户（root）的主目录，是系统管理员的家目录。
11. **/run**：包含系统运行时的临时文件，如进程ID文件和套接字文件。是一个临时文件系统，存储系统启动以来的信息。
12. **/srv(service)**：用于存放服务的数据文件，如Web服务器、FTP服务器等。该目录存放一些服务启动之后需要提取的数据。
13. **/sys(system)**：包含与内核和系统硬件相关的虚拟文件系统。该目录下安装了 2.6 内核中新出现的一个文件系统 sysfs 。
14. /**bin(binaries)**：
    - 功能：存放系统指令。主要放置一些*系统*的*必备执行档*，例如 cat、cp、chmod df、dmesg、gzip、kill、ls、mkdir、more、mount、rm、su、tar等。
    - 权限：管理员和**一般的用户**都可以使用。
    - 可运行时间：在*系统启动后*挂载到根文件系统中的，所以/bin目录必须和根文件系统在同一分区，在挂载其他文件系统前就可以使用。
15. /**sbin(superuser binaries)**：
    - 功能：存放指超级用户指令，主要放置一些*系统管理*的*必备程式*，例如 cfdisk、dhcpcd、dump、e2fsck、fdisk、halt、ifconfig、ifup、 ifdown、init、insmod、lilo、lsmod、mke2fs、modprobe、quotacheck、reboot、rmmod、 runlevel、shutdown等。
    - 权限：通常只有管理员才可以运行。
    - 可运行时间：在*系统启动后*挂载到根文件系统中的，所以/sbin目录必须和根文件系统在同一分区，在挂载其他文件系统前就可以使用。

16. **/usr(unix system resources)**：包含用户可执行程序、库文件、文档等，是系统的二进制文件和资源的主要位置。用户的很多应用程序和文件都放在这个目录下，类似于 windows 下的 program files 目录。
    1. /**bin(binaries)**：系统用户使用的应用程序。主要供用户维护。
        - 可运行时间：可以和根文件系统不在一个分区。
        - 功能：存放在后期安装的一些软件的*运行脚本*。主要放置一些*应用*的*软体工具*的*必备执行档*，例如 c++、g++、gcc、chdrv、diff、dig、du、eject、elm、free、gnome、 gzip、htpasswd、kfm、ktop、last、less、locale、m4、make、man、mcopy、ncftp、 newaliases、nslookup passwd、quota、smb、wget等。

    2. /**sbin(superuser binaries)**：超级用户使用的比较高级的管理程序和系统守护程序。
        - 可运行时间：可以和根文件系统不在一个分区。
        - 功能：存放用户安装的系统管理的必备程式，例如 dhcpd、httpd、imap、in.*d、inetd、lpd、named、netconfig、nmbd、samba、sendmail、squid、swap、tcpd、tcpdump等。

    3. /**src**：内核源代码默认的放置目录。
    4. /**lib(library)**：依赖资源。
    5. /**lib64(library64)**：64位依赖资源。

17. **/var(variable)**：包含变化频繁的文件，如日志文件、数据库文件、缓存等。
    1. **/tmp(temporary)**：用于存放临时文件的目录，系统启动后会定期清空。

18. **/lost+found**：该目录一般情况下是空的，当系统非法关机后，这里会存放一些文件。
19. /**selinux**：这个目录是 Redhat/CentOS 所特有的目录，Selinux 是一个安全机制，类似于 windows 的防火墙，但是这套机制比较复杂，这个目录就是存放selinux相关的文件的。

这些根目录的组织有助于Linux系统的管理和维护，使文件和资源能够按照特定的目的进行组织和存放。不同的目录有不同的权限和用途，有助于保持系统的安全性和可维护性。
