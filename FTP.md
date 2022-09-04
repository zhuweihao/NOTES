# FTP

https://blog.csdn.net/cuichongxin/article/details/116519184

# Linux基础篇——ftp的安装与配置

https://blog.csdn.net/LXWalaz1s1s/article/details/123779435

ftp的配置文件

```
#打开配置文件
vim /etc/vsftpd/vsftpd.conf
```



Windows自己搞了个，没用到。

![捕获](E:\FTPShare\捕获.JPG)



```
ftp> ls
227 Entering Passive Mode (127,0,0,1,49,132).
150 Here comes the directory listing.
-rw-r--r--    1 0        0               5 Jul 10 13:22 test
226 Directory send OK.
ftp> put student.txt 
local: student.txt remote: student.txt
227 Entering Passive Mode (127,0,0,1,172,13).
150 Ok to send data.
226 Transfer complete.
98 bytes sent in 0.105 secs (0.94 Kbytes/sec)
ftp> ls
227 Entering Passive Mode (127,0,0,1,239,156).
150 Here comes the directory listing.
-rw-------    1 14       50             98 Jul 10 13:44 student.txt
-rw-r--r--    1 0        0               5 Jul 10 13:22 test
226 Directory send OK.
ftp> ^Z
[1]+  Stopped                 ftp hadoop102
[root@hadoop102 datas]# cd ..
[root@hadoop102 local]# ftp hadoop102
Connected to hadoop102 (127.0.0.1).
220 Welcome to blah FTP service.
Name (hadoop102:root): ftp
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
ftp> put /usr/local/datas/student.txt 
local: /usr/local/datas/student.txt remote: /usr/local/datas/student.txt
227 Entering Passive Mode (127,0,0,1,186,216).
553 Could not create file.
```

原因：文件权限的问题

解决办法1：对用户ftp进行处理

解决办法2：解决每一层的文件夹的权限

两者应该都能解决问题

我们来看一下这几个文件的权限

```

drwxr-xr-x.  22 root root 4096 Mar 11 03:48 var
rwxr-xr-x.  3 root root   18 Jul  7 21:59 ftp
drwxrwxrwx. 2 root root 37 Jul 10 06:48 pub
drwxr-xr-x. 2 root root 6 Jul  7 21:59 test




drwxr-xr-x.  13 root root  158 Mar 11 03:40 usr
drwxr-xr-x.  27 root root  4096 Jul  7 21:59 local
drwxr-xr-x. 39 root root 4096 Jul  8 01:00 datas
-rwxrwxrwx. 1 root root  98 Jun 20 01:21 student.txt
```

## 测试上传功能

```
package com.atsugon;

import cn.hutool.core.io.FileUtil;
import cn.hutool.extra.ftp.Ftp;
import cn.hutool.extra.ftp.FtpMode;
import sun.net.ftp.FtpClient;

import java.io.File;
import java.io.IOException;


public class FtpUpload {
    public static void main(String[] args) throws IOException {
        final Ftp ftp = new Ftp("hadoop102",21,"ftp",null);
        //ftp.upload("/home/vsftpd/ftp-user1/", FileUtil.file("/usr/local/datas/student.txt"));

        
       // ftp = ftp.reconnectIfTimeout();
        ftp.cd("pub");
        System.out.println(ftp.ls("pub"));
        System.out.println("pwd:"+ftp.pwd());
        ftp.setMode(FtpMode.Passive);
        //ftp.getClient().enterLocalPassiveMode();
       //上传本地文件
       // File file = FileUtil.file("/usr/local/datas/student.txt");

       //文件都要是绝对路径
        boolean flag = ftp.upload(null, new File("/usr/local/datas/student.txt"));
        System.out.println(flag);
        System.out.println(ftp.ls("pub"));
        ftp.close();
    }
}

```

第一次

```
pwd:/pub
[test]
true
[student.txt, test]
```

第二次

```
[student.txt, test]
pwd:/pub
false
[student.txt, test]
```

从上述结果可以看出，普通用户的上传重名的文件不会覆写，不会上传。

## 测试下载功能

![image-20220711121600931](https://s2.loli.net/2022/07/11/rgw429KHqPexO1I.png)

```
ftp> get test
local: test remote: test
227 Entering Passive Mode (127,0,0,1,65,51).
150 Opening BINARY mode data connection for test (5 bytes).
226 Transfer complete.
5 bytes received in 0.00738 secs (0.68 Kbytes/sec)
ftp> ls 
227 Entering Passive Mode (127,0,0,1,55,72).
150 Here comes the directory listing.
-rw-------    1 14       50             98 Jul 11 04:06 student.txt
-rw-r--r--    1 0        0               5 Jul 10 13:22 test
226 Directory send OK.

```

上传和下载，put和get必须得要在当前文件文件夹进行，比如我们想要将/usr/local/datas/student.txt文件上传到ftp服务器上，我们须在/usr/local/datas/目录下进行ftp连接，下载同理。

程序实现

```

```

