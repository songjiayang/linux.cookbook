# linux.cookbook

个人工作中用到的 Linux 琐碎知识点汇总，所有例子都能运行在 Ubuntu 系统上。

## 索引

- [磁盘分区](https://github.com/songjiayang/linux.cookbook#磁盘分区)
- [磁盘格式化](https://github.com/songjiayang/linux.cookbook#磁盘格式化)
- [挂载分区](https://github.com/songjiayang/linux.cookbook#挂载分区)
- [修改 fstab](https://github.com/songjiayang/linux.cookbook#修改-fstab)
- [创建 deploy 账户](https://github.com/songjiayang/linux.cookbook#创建-deploy-账户)

### 磁盘分区

```bash
sudo fdisk -l // 查看磁盘分区情况
sudo fdisk /dev/vdb //磁盘分区
```

### 磁盘格式化

```bash
sudo mkfs.ext4 /dev/vdb1 
```

### 挂载分区

```bash
mkdir /db 
mkdir /web
mount /dev/vdb1 /db  
mount /dev/vdc1 /web 
```

### 修改 fstab

vi `/etc/fstab`

```bash
# <file system> <mount point>   <type>  <options>       <dump>  <pass>
/dev/vdb1 /data ext4 defaults 0 0
```
参考 [fstab](https://wiki.archlinux.org/index.php/Fstab)

### 创建 deploy 账户

```bash
sudo adduser deploy  # Add a user for deployment
sudo usermod -a -G sudo deploy
```
