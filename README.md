# ws215i-rootfs
本项目包含ws215i的rootfs和系统烧录U盘的制作工具。



## Quick Start

```bash
git clone https://github.com/wisnuc/ws215i-rootfs
npm i
node wisnuc.js --all
sudo ./build-rootfs-emmc-base.sh
sudo ./build-burn-base.sh
sudo ./imagify.sh
```





## 合成过程

```
wisnuc.js           build-rootfs-emmc-base.sh           build-burn-base.sh
   |                      |                                   |
   v                      v                                   |
wisnuc directory  + ws215i-rootfs-emmc-base.tar.gz            |
                  |                                           |
                  v                                           v
        ws215i-rootfs-emmc.tar.gz   +   ws215i-rootfs-burn-base(-debug).tar.gz
                                    |
                                    v
                                imagefile(-debug)
```



## wisnuc.js

该脚本合成在目标系统上预部署的`/wisnuc`目录，包括：

1. 建立目录结构
2. node
3. wisnuc-bootstrap
4. wisnuc-bootstrap-update
5. wetty
6. appifi

输出为当前目录的`wisnuc`目录。

运行该脚本无须root权限。

该脚本支持参数`--all`，首次使用应该使用该参数。

```bash
node wisnuc.js --all	# 更新全部
node wisnuc.js			# 仅更新appifi
```



## build-rootfs-emmc-base.sh

该脚本创建ws215i的rootfs (emmc)压缩包文件。执行该脚本需要root权限。

过程如下：

1. 创建`target_emmc`目录
2. 在目录下安装ubuntu base
3. 修改apt源为中国镜像
4. 创建如下systemd unit
   1. firstboot
   2. timesyncd
   3. wisnuc-bootstrap-update，包括timer和service
   4. wisnuc-bootstrap
   5. wetty
   6. wired.network
   7. resolv.conf，先放入一个临时版本，在chroot最后更新其为生产环境版本
   8. hosts
   9. hostname
5. chroot
   1. 安装deb包
   2. 创建wisnuc用户，加入sudo
   3. 安装ws215i内核并阻止内核升级
   4. 使能所有需要的systemd服务，禁用samba和minidlna服务
   5. 创建ws215i启动需要的boot文件（包括符号链）
   6. 修改fstab
6. post-install (leave chroot)
   1. 清理内核deb文件
   2. 清理apt
   3. 更新resolv.conf (symlink to systemd resolv.conf)
7. 最后把rootfs打包成`ws215i-rootfs-emmc-base.tar.gz`

最终输出的压缩包文件不包含目标系统`/wisnuc`目录下所有的预先部署文件。



## build-burn-base.sh

该脚本生成USB烧录盘的最小文件系统，USB烧录盘本身也是一个包含完整rootfs的ubuntu运行系统，不只是ramdisk。其内容和emmc镜像相仿，做了裁剪。



该脚本接受参数`--debug`，debug模式下最终输出的镜像文件会包含openssh server，方便开发者调试。

生产模式下输出的文件是`ws215i-rootfs-burn-base.tar.gz`

debug模式下输出的文件是`ws215i-rootfs-burn-debug.tar.gz`



## imagify.sh

`imagify.sh` generates the USB drive image file.



It requires:

1. `rootfs-emmc-base.tar.gz`
2. `ws215i-rootfs-burn-base.tar.gz` for production or `ws215i-rootfs-burn-debug.tar.gz` for debug or development.











This project includes files to build rootfs for ws215i, including:

1. rootfs for emmc
2. rootfs for usb drive burner.







Build rootfs for ws215i