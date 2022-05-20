> 20220520

这是帮助王权涛制作iso系统镜像时的一个记录，一步一步也算有所收获

首先在网络上查询方法，找到一下方法及要解决的问题

+ 需要使用SystemBack工具
  + 但是该工具16年便停止更新，因此ubuntu 18-04不支持
  + 完成SystemBack工具下载及iso制作后，systemback软件自身无法处理超过4G的iso
  + 下载第三方工具cdrtools-3.02a07.tar可以解决这个问题
  + 我自己出现了因为在VMWare中操作但开辟磁盘空间太小的问题

下面我针对这些问题一一解决



## SystemBack下载

To install systemback on Ubuntu 18.04/19.10, first remove the PPA.

```
sudo add-apt-repository --remove ppa:nemh/systemback
```

The Systemback binary for Ubuntu 16.04 is compatible with Ubuntu 18.04/19.10/20.04, so we can add the Ubuntu 16.04 PPA on 18.04/19.10. Run the following command to import the GPG signing key of this PPA so that the package manager can verify signature. The signing key can be found on [launchpad.net](https://launchpad.net/~nemh/+archive/ubuntu/systemback).

```
sudo apt-key adv --keyserver keyserver.ubuntu.com --recv-keys 382003C2C8B7B4AB813E915B14E4942973C62A1B
```

If you see the following error:

```
gpg: keyserver receive failed: no keyserver available
```

or

```
gpg: keyserver receive failed: No data
```

You can fix this error by using a different keyserver. So instead of `keyserver.ubuntu.com` you can use `pgp.mit.edu`.

```
sudo apt-key adv --keyserver pgp.mit.edu --recv-keys 382003C2C8B7B4AB813E915B14E4942973C62A1B
```

Then add the PPA on Ubuntu 18.04/19.10/20.04.

```
sudo add-apt-repository "deb http://ppa.launchpad.net/nemh/systemback/ubuntu xenial main"
```

Update package list and install Systemback.

```
sudo apt update

sudo apt install systemback
```

Then you can start Systemback from your application menu. You need to enter your password to use this software. After you enter the password, click OK button.

## SystemBack操作制作ISO

选择Live system create：

![image-20220520113331341](../../../../../md图床/image-20220520113331341.png)

勾选左侧的 include the user data files，这样自己主文件夹内的文件都会被包含在系统镜像中。要保证 /home有足够的空间：

![image-20220520113403986](../../../../../md图床/image-20220520113403986.png)

点击Create New按钮就开始创建了，等待创建完成：



![image-20220520113435770](../../../../../md图床/image-20220520113435770.png)

右侧的列表中就是已经创建的备份。我已经创建了两个相关的备份，所以有两个在右侧显示。此时文件没有转换成iso格式，选中你要转换的备份，点击convert to ISO 就可以开始转换了。转换完成后，在你的工作目录下就能找到生成的iso文件



## 超过4G的问题

如果生成的系统镜像小于4G，才能直接转存为光盘镜像。否则要使用下面的方法：

### 压缩系统镜像

Systemback在使用时会发现当生成的sblive文件大于4G的时候是没有办法生成iso文件的。这是由于iso文件自身的限制，iso9600对于文件有限制，单个文件不能超过2G，总的iso文件不能超过4G。
	所以当上面生成的系统镜像如果大于4G，不能直接转存为iso文件，就要使用采用udf文件系统压缩再转存为光盘文件。
	进入计算机的home文件夹，可以看到这里面有一个systemback生成的文件：
![img](../../../../../md图床/1075214-20190528172934527-1454722012.png)
	**第一步：解压 .sblive 文件：**

```bash
mkdir sblive
tar -xf /home/systemback_live_2018-10-15.sblive -C sblive
```

**第二步：重命名syslinux 至 isolinux：**

```bash
mv sblive/syslinux/syslinux.cfg sblive/syslinux/isolinux.cfg
mv sblive/syslinux sblive/isolinux
```

**第三步：安装 cdtools：**

```go
sudo apt install aria2
//可以在浏览器打开下面的链接手动下载
aria2c -s 10 https://nchc.dl.sourceforge.net/project/cdrtools/alpha/cdrtools-3.02a07.tar.gz
tar -xvf cdrtools-3.02a07.tar.gz

cd cdrtools-3.02
make
sudo make install
```

**第四步：生成ISO文件：**

```bash
/opt/schily/bin/mkisofs -iso-level 3 -r -V sblive -cache-inodes -J -l -b isolinux/isolinux.bin -no-emul-boot -boot-load-size 4 -boot-info-table -c isolinux/boot.cat -o sblive.iso sblive
```