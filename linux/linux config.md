linux

### 修改root密码

``` sh
su # 如果需要切换到root用户
# 输入当前root密码

# 或者
sudo -i # 如果你有sudo权限
# 输入你的用户密码

passwd # 修改root密码
# 输入新的root密码
# 再次输入新的root密码以确认
```



### 修改hostname

``` sh
sudo hostnamectl set-hostname new_hostname

# 或者手动修改
sudo nano /etc/hostname
sudo nano /etc/hosts

```



### ssh 免密登录

1. 客户端生成密钥对

``` sh
ssh-keygen -t rsa -b 4096

```

2. 安装公钥到服务器

生成密钥对之后，你需要将公钥（默认是`~/.ssh/id_rsa.pub`）安装到远程服务器上的`~/.ssh/authorized_keys`文件中。这样，服务器就能识别尝试连接的客户端。

有多种方法可以完成这一步，最简单的方式是使用`ssh-copy-id`命令：

```sh
ssh-copy-id 用户名@服务器地址
```



### 更新包管理和安装软件

``` sh
apt-get update
apt install name
```



### zsh配置

``` sh
echo $SHELL
apt install zsh
which zsh
sudo chsh -s /bin/zsh

#oh my zsh
apt install git
sh -c "$(curl -fsSL https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"

#去 https://github.com/zsh-users/zsh-syntax-highlighting 安装插件
#去 https://github.com/zsh-users/zsh-autosuggestions 安装插件
# https://github.com/zsh-users/zsh-completions
```



### 防火墙

``` sh
ufw status

ufw enable #开启
ufw disable #关闭
ufw allow 22/tcp #开放某端口

```



### mysql 安装

````sh
#安装
wget 官网
sudo dpkg -i /path/to/package.deb
udo apt install mysql-server
sudo apt install libaio1 libmecab2
#启动
sudo systemctl status mysql.service
sudo systemctl start mysql.service
#配置
sudo mysql_secure_installation #配置密码

sudo mysql -u root -p # 进入执行下面(使用MySQL命令行，修改你想要能够远程连接的用户的主机设置。以下命令以your_password为密码创建或更新一个root用户，允许该用户从任何主机连接。)
```
CREATE USER 'root'@'%' IDENTIFIED BY 'your_password';
GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' WITH GRANT OPTION;
FLUSH PRIVILEGES;
```

vi /etc/mysql/mysql.conf.d/mysqld.cnf
#找到bind-address行，更改若无则添加为：
[mysqld]
bind-address = 0.0.0.0

#重启
sudo systemctl restart mysql.service

````



### Redis安装

``` sh
#安装
wget http://download.redis.io/redis-stable.tar.gz
tar -xzvf redis-stable.tar.gz
cd redis-stable
make #编译
make install

#配置
cp redis.conf /etc/redis.conf
redis-server /etc/redis.conf #使用配置文件启动server
redis-cli #启动客户端
redis-cli shutdown #关闭

#配置 Redis 为后台服务
将配置文件中的 daemonize no 改成 daemonize yes，配置 redis 为后台启动。

#Redis 设置访问密码
在配置文件中找到 requirepass，去掉前面的注释，并修改后面的密码。

```

配置文件举例

``` sh
#默认端口6379
port 6379
#绑定ip，如果是内网可以直接绑定 127.0.0.1, 或者忽略, 0.0.0.0是外网
bind 0.0.0.0
#守护进程启动
daemonize yes
#超时
timeout 300
loglevel notice
#分区
databases 16
save 900 1
save 300 10
save 60 10000
rdbcompression yes
#存储文件
dbfilename dump.rdb
#密码 abcd123
requirepass abcd123
```



### nginx

``` sh
wget https://nginx.org/download/nginx-1.22.0.tar.gz

cp nginx-1.22.0.tar.gz /usr/local/
cd /usr/local/
tar -zxvf nginx-1.22.0.tar.gz

cd nginx-1.22.0   

sudo apt update
sudo apt install build-essential		#安装c编译器
sudo apt install libpcre3 libpcre3-dev
sudo apt install zlib1g zlib1g-dev


./configure       #执行配置文件 默认安装到/usr/local/nginx
./configure --prefix=/usr/local/nginx --with-http_ssl_module #这个可以指定路径

make               #手动安装
make install       #若不确定再执行次文件

# 配置
cd /usr/local/nginx/conf         //进入配置目录
vim nginx.conf                   //编辑配置文件

#运行
cd ./sbin

./nginx -v										//查看版本
./nginx                       //启动
./nginx -s stop               //停止
./nginx -s quit               //安全退出
./nginx -s reload             //重载配置文件（修改了配置文件需要执行此命令 比较常用）
ps aux|grep nginx             //查看ngnix进程

```



### java

``` sh
wget https://download.oracle.com/java/17/latest/jdk-17_linux-x64_bin.deb
apt install ./jdk-17_linux-x64_bin.deb

#添加配置环境
export JAVA_HOME=/usr/lib/jvm/jdk-17
export JRE_HOME=${JAVA_HOME}/jre
export CLASSPATH=.:${JAVA_HOME}/lib:${JRE_HOME}/lib
export PATH=${JAVA_HOME}/bin:$PATH

```
