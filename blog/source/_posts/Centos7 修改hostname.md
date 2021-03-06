---
title: Centos7 修改 hostname
date: 2016-11-25 09:49:53
tags: [centos, hostname]
categories: [linux]
---
在CentOS或RHEL中，有三种定义的主机名:a、静态的（static），b、瞬态的（transient），以及 c、灵活的（pretty）。“静态”主机名也称为内核主机名，是系统在启动时从/etc/hostname自动初始化的主机名。“瞬态”主机名是在系统运行时临时分配的主机名，例如，通过DHCP或mDNS服务器分配。静态主机名和瞬态主机名都遵从作为互联网域名同样的字符限制规则。而另一方面，“灵活”主机名则允许使用自由形式（包括特殊/空白字符）的主机名，以展示给终端用户（如Dan's Computer）。

在CentOS/RHEL 7中，有个叫hostnamectl的命令行工具，它允许你查看或修改与主机名相关的配置。
要查看主机名相关的设置：
```
$ hostnamectl status
```
```
[root@zfy-79 zfy]# hostnamectl status
   Static hostname: zfy-79
         Icon name: computer-desktop
           Chassis: desktop
        Machine ID: 89b2147b6a1f4fe1a89148cfd53dd727
           Boot ID: ee5625e25b494acc804e49043dcd8626
  Operating System: CentOS Linux 7 (Core)
       CPE OS Name: cpe:/o:centos:centos:7
            Kernel: Linux 3.10.0-327.36.3.el7.x86_64
      Architecture: x86-64
```

同时修改所有三个主机名：静态、瞬态和灵活主机名：

```
[root@zfy-79 zfy]# sudo hostnamectl set-hostname dev-79
[root@zfy-79 zfy]# hostnamectl status
   Static hostname: dev-79
         Icon name: computer-desktop
           Chassis: desktop
        Machine ID: 89b2147b6a1f4fe1a89148cfd53dd727
           Boot ID: ee5625e25b494acc804e49043dcd8626
  Operating System: CentOS Linux 7 (Core)
       CPE OS Name: cpe:/o:centos:centos:7
            Kernel: Linux 3.10.0-327.36.3.el7.x86_64
      Architecture: x86-64
```
如果你只想修改特定的主机名（静态，瞬态或灵活），你可以使用“--static”，“--transient”或“--pretty”选项。
**注意，你不必重启机器以激活永久主机名修改。上面的命令会立即修改内核主机名。注销并重新登入后在命令行提示来观察新的静态主机名。**

