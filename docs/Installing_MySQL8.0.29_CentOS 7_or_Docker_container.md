# Installing MySQL 8.0.29 on CentOS 7 or Docker container

@Kerim Narov 

April 7, 2023 12:50 PM

#**The [container] installation section of the host can be ignored**

### **Create Container [Container]**

`docker run -itd --name mycat --privileged=true centos:7.9.2009 /usr/sbin/init
docker exec -it 26a7b7740b39 /bin/bash`

# **Preparation**

### **Install Java [container]**

`yum install java

[root@574ab73efc24 /]# java -version
openjdk version "1.8.0_332"
OpenJDK Runtime Environment (build 1.8.0_332-b09)
OpenJDK 64-Bit Server VM (build 25.332-b09, mixed mode)`

### **Install wget [Container]**

`yum -y install wget`

### **Installing Vim [container]**

`yum -y install vim`

### Uninstall MariaDB

`# View Version：
[root@localhost opt]# rpm -qa|grep mariadb
mariadb-libs-5.5.68-1.el7.x86_64
# Uninstall
[root@localhost opt]# rpm -e --nodeps mariadb-libs-5.5.68-1.el7.x86_64
# Check if it has been uninstalled cleanly
rpm -qa|grep mariadb`

### Uninstall old MySQL

`rpm -qa | grep mysql

[root@localhost opt]# rpm -qa | grep mysql
mysql80-community-release-el8-4.noarch
[root@localhost opt]# yum remove mysql80-community-release-el8-4.noarch`

`[root@localhost opt]# find / -name mysql
/etc/selinux/targeted/active/modules/100/mysql
/usr/lib64/mysql
/usr/share/mysql
[root@localhost opt]# rm -rf /etc/selinux/targeted/active/modules/100/mysql
[root@localhost opt]# rm -rf /usr/lib64/mysql
[root@localhost opt]# rm -rf /usr/share/mysql
[root@localhost opt]# rpm -pa | grep mariadb`

## **Source Code Installation**

### **Download MySQL**

`Acceleration Station
yum update ca-certificates -y
wget https://mirrors.tuna.tsinghua.edu.cn/mysql/downloads/MySQL-8.0/mysql-8.0.29-el7-x86_64.tar

tar -xvf mysql-8.0.29-el7-x86_64.tar
tar -zxvf mysql-8.0.29-el7-x86_64.tar.gz

mv mysql-8.0.29-el7-x86_64 mysql
mv mysql /usr/local/
cd /usr/local/mysql
mkdir mysqldb

Grant Permissions to MySQL Installation Directory
chmod -R 777 /usr/local/mysql

Create Group
[root@localhost mysql]# pwd
/usr/local/mysql

groupadd mysql

Creating user (-s /bin/false parameter specifies that the mysql user has ownership only without login permissions)
useradd -r -g mysql -s /bin/false mysql

Add user to group
chown -R mysql:mysql ./`

### **Fixing Chinese character garbled [Container]**

`[root@a8fedf6f30e8 mysql]# export LANG=en_ZW.utf8
[root@a8fedf6f30e8 mysql]# source /etc/profile

[root@a8fedf6f30e8 mysql]# vim /etc/profile   
LANG=en_ZW.utf8
[root@a8fedf6f30e8 mysql]# source /etc/profile`

### Modify MySQL configuration file using vi /etc/my.cnf

`[mysqld]
# Set port 3306
port=3306
# Set the installation directory for MySQL
basedir=/usr/local/mysql
# Set the storage directory for MySQL database
datadir=/usr/local/mysql/mysqldb
# Maximum allowed connections
max_connections=10000
# Number of allowed connection failures. This is to prevent someone from attempting to attack the database system from that host.
max_connect_errors=10
# The character set used by the server defaults to UTF8
character-set-server=utf8
Default storage engine used when creating a new table
default-storage-engine=INNODB
# Authenticate using the "mysql_native_password" plugin by default
default_authentication_plugin=mysql_native_password
[mysql]
# Set default character set for mysql client
default-character-set=utf8
[client]
Set default port for mysql client to connect to server
port=3306
default-character-set=utf8`

### Install dependency library libaio [container]

`yum install -y libaio
yum -y install numactl.x86_64
rpm -qa|grep libaio`

### **Install MySQL**

`cd /usr/local/mysql/bin/
./mysqld --initialize --console

Password: S)oBeoS;q6LV
2022-07-02T09:04:18.017337Z 6 [Note] [MY-010454] [Server] A temporary password is generated for root@localhost: S)oBeoS;q6LV`

### Start MySQL service

`cd /usr/local/mysql/support-files
chmod -R 777 /usr/local/mysql
./mysql.server start`

### Adding MySQL to the system processes

`cp /usr/local/mysql/support-files/mysql.server /etc/init.d/mysqld`
Now we can use the service process to operate MySQL.

### Setting up MySQL to start automatically

`chmod +x /etc/init.d/mysqld
systemctl enable mysqld

[root@localhost support-files]# chmod +x /etc/init.d/mysqld
[root@localhost support-files]# systemctl enable mysqld
mysqld.service is not a native service, redirecting to /sbin/chkconfig.
Executing /sbin/chkconfig mysqld on`

### Setting Environment Variables

`vim /etc/profile

MYSQL_HOME=/usr/local/mysql
export PATH=$PATH:$MYSQL_HOME/bin

source /etc/profile`

### Change root user login password

`#log in mysql
cd /usr/local/mysql/bin/
./mysql -u root -p

mysql> alter user 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY '123456';`

### **Enable remote login**

`mysql> use mysql
mysql> update user set user.Host='%'where user.User='root';
mysql> flush privileges;
mysql> quit`

### Restart service and test

`systemctl restart mysql 
systemctl status mysql`

### View Open Ports in Firewall

`firewall-cmd --list-all`

### Open port 3306 in the firewall

`firewall-cmd --zone=public --add-port=3306/tcp --permanent
firewall-cmd --reload
//--permanent means permanent effect, without this parameter the configuration will be invalid after the server restarts.

# Disable Firewall
systemctl stop firewalld.service
# Check Firewall Status
firewall-cmd --state`
