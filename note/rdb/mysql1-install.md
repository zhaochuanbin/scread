windows下MySQL 5.7及以上版本压缩包
### 下载解压
mysql的下载地址  https://www.mysql.com/downloads/ ,下载为64位,解压的位置是D:\green\mysql-5.7.10-winx64
### 添加环境变量
 右键计算机->属性->高级系统设置->环境变量；在系统变量里添加MYSQL_HOME环境变量，变量值为MySQL的根目录，例如我的是D:\green\mysql-5.7.10-winx64,
 找到path，选择编辑，在原有值末尾添加;%MYSQL_HOME%\bin;
###  添加配置文件
在MySQL的安装目录,修改my.ini文件
>[mysqld]
basedir=D:\green\mysql-5.7.10-winx64
datadir=D:\green\mysql-5.7.10-winx64\data
port = 3306

### 初始化数据库
以管理员自身份打开CMD执行以下命令（注意必须以管理员身份打开，否则报错）
mysqld --initialize --user=mysql --console
在控制台消息尾部会出现随机生成的初始密码，记下来（因为有特殊字符，很容易记错，最好把整个消息保存在记事本里）
如果上述命令运行不成功请用以下命令代替：
>%MYSQL_HOME%\bin\mysqld --initialize --user=mysql --console

### 将MySQL添加到系统服务
以管理员自身份打开CMD执行以下命令（注意必须以管理员身份打开，否则报错）
mysqld --install MySQL
net start MySQL
安装成功，则显示“服务已启动成功”
如果上述命令运行不成功，可以用以下命令代替：
%MYSQL_HOME%\bin\mysqld --install MySQL
net start MySQL

### 启动MySQL并修改密码
在CMD控制台里执行命令  mysql -u root -p 
回车执行后，输入刚才记录的随机密码
执行成功后，控制台显示 mysql>，则表示进入mysql
输入命令set password for root@localhost = password('root'); （注意分号）
此时root用户的密码修改为root