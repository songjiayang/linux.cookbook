# linux.cookbook

个人工作中用到的 Linux 琐碎知识点汇总，所有例子都能运行在 Ubuntu 系统上。

## 索引
 - 系统环境
    - [磁盘分区](https://github.com/songjiayang/linux.cookbook#磁盘分区)
    - [磁盘格式化](https://github.com/songjiayang/linux.cookbook#磁盘格式化)
    - [挂载分区](https://github.com/songjiayang/linux.cookbook#挂载分区)
    - [修改 fstab](https://github.com/songjiayang/linux.cookbook#修改-fstab)
    - [创建 deploy 账户](https://github.com/songjiayang/linux.cookbook#创建-deploy-账户)
    - [配置 ssh](https://github.com/songjiayang/linux.cookbook#配置-ssh)
    - [配置系统语言环境](https://github.com/songjiayang/linux.cookbook#配置系统语言环境)
- 应用软件
    - [安装 Nginx](https://github.com/songjiayang/linux.cookbook#安装-nginx)
    - [安装 Supervisord](https://github.com/songjiayang/linux.cookbook#安装-supervisord)
    - [安装 MongoDB](https://github.com/songjiayang/linux.cookbook#安装-mongodb)
    - [安装 MySQL](https://github.com/songjiayang/linux.cookbook#安装-mysql)
    - [安装 SSL 证书](https://github.com/songjiayang/linux.cookbook#安装-ssl-证书)

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

### 配置 ssh

```bash
ssh deploy@host 'mkdir -p .ssh && cat >> .ssh/authorized_keys' < ~/.ssh/id_rsa.pub
cd /etc/ssh
sudo vi sshd_config
# modify sshd_config
PermitRootLogin no
RSAAuthentication yes
PubkeyAuthentication yes
AuthorizedKeysFile %h/.ssh/authorized_keys
# restart ssh server
sudo service ssh restart
```

### 配置系统语言环境

```bash
# ls 初始化月份字符串出错
sudo vi /var/lib/locales/supported.d/local
# 改成如下内容
en_US.UTF-8 UTF-8
zh_CN.UTF-8 UTF-8
zh_CN.GBK GBK
zh_CN GB2312
# 执行
sudo locale-gen
sudo vi /etc/default/locale
# 改成如下内容
LANG="zh_CN.UTF-8"
LANGUAGE="zh_CN.UTF-8"
LC_ALL="zh_CN.UTF-8"
```

### 安装 Nginx

```bash
sudo apt-key add nginx_signing.key

deb http://nginx.org/packages/ubuntu/ codename nginx
deb-src http://nginx.org/packages/ubuntu/ codename nginx

apt-get update
apt-get install nginx
```

codename 指不同平台的特定版本，具体参考 [nginx download](http://nginx.org/en/linux_packages.html#stable)。

前后端分离服务简易模版：

```vi

log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                  '$request_time-$upstream_response_time $status $body_bytes_sent "$http_referer" '
                  '"$http_user_agent" "$http_x_forwarded_for"';
upstream example-backend {
    keepalive 60;
    server 127.0.0.1:9090;
}

server {
    listen 80;
    server_name example.coom;

    client_max_body_size 2m;
    client_body_buffer_size 512K;
    client_header_buffer_size 4k;

    large_client_header_buffers 4 8k;

    access_log /var/log/nginx/example.log main;
    
    location / {
      root /var/www/example;
    }

    location /v1.0 {
        proxy_http_version 1.1;
        proxy_set_header Host $http_host;
        proxy_set_header Connection "";
        proxy_set_header X-Scheme $scheme;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_buffers 64 4k;
        proxy_buffer_size 32k;
        proxy_redirect off;
        proxy_pass http://example-backend;
        break;
    }
}
```

### 安装 Supervisord

```bash
apt-get update
apt-get install supervisord
```

supervisrod 配置模版:

```vim
# vi /etc/supervisor/conf.d/example.conf
[program:example]
command=/var/www/example/bin/example -srcPath=/var/www/example -runMode=production
directory=/var/www/example
process_name=%(program_name)s ; process_name expr (default %(program_name)s)
numprocs=1                    ; number of processes copies to start (def 1)
startsecs=1                   ; number of secs prog must stay running (def. 1)
startretries=3                ; max # of serial start failures (default 3)
user=deploy                     ; setuid to this UNIX account to run the program
redirect_stderr=true          ; redirect proc stderr to stdout (default false)
stdout_logfile=/var/www/example/logs/production.log
stdout_logfile_maxbytes=50MB  ; max # logfile bytes b4 rotation (default 50MB)
stdout_logfile_backups=10     ; # of stdout logfile backups (default 10)
stdout_capture_maxbytes=1MB   ; number of bytes in 'capturemode' (default 0)
stdout_events_enabled=false   ; emit events on stdout writes (default false)
stderr_logfile=/web/www/gateway/logs/gateway.stderr.log
stderr_logfile_maxbytes=10MB  ; max # logfile bytes b4 rotation (default 50MB)
stderr_logfile_backups=10     ; # of stderr logfile backups (default 10)
stderr_capture_maxbytes=1MB   ; number of bytes in 'capturemode' (default 0)
stderr_events_enabled=false   ; emit events on stderr writes (default false)
```

常用命令

```bash
sudo supervisorctl reread
sudo supervisorctl add example
sudo supervisorctl restart example
```

### 安装 MongoDB

```
# install on ubuntu 14.04
echo "deb [ arch=amd64 ] https://repo.mongodb.org/apt/ubuntu trusty/mongodb-org/3.6 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-3.6.list

sudo apt-get update
sudo apt-get install -y mongodb-org
```

参考 [install MongoDB](https://docs.mongodb.com/manual/tutorial/install-mongodb-on-ubuntu/)。

### 安装 MySQL

comming soon

### 安装 SSL 证书

这里我们使用 [acme.sh](https://github.com/Neilpang/acme.sh) 来安装。

acme.sh 涉及命令：

```
curl https://get.acme.sh | sh  # 安装 acme.sh

0 0 * * * "/home/user/.acme.sh"/acme.sh --cron --home "/home/user/.acme.sh" > /dev/null # 定期跟新证书，因为证书3个月会失效

acme.sh --issue -d example.com -w /home/wwwroot/example.com # 申请特定域名证书

# 安装域名证书
acme.sh --install-cert -d example.com \
--key-file       /path/to/keyfile/in/nginx/key.pem  \
--fullchain-file /path/to/fullchain/nginx/cert.pem \
--reloadcmd     "service nginx force-reload"
```

nignx 配置：

```

# 80 端口跳转 443
server {
  listen 80;
  
  server_name example.com;
  
  return 301 https://$host$request_uri;
}

# 443 端口配置
server {
  listen 443 ssl;

  server_name example.com;;
  ssl_certificate     /etc/nginx/ssl/example.com;.cert.pem;
  ssl_certificate_key /etc/nginx/ssl/example.com;.key.pem;
}

```
