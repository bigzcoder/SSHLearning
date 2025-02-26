# Redis&Linux装应用

## 在linux安装jdk、tomcat、mysql、redis

准备工作

* 创建目录

    ```txt
    mkdir /usr/local/src/java
    mkdir /usr/local/src/mysql
    mkdir /usr/local/src/tomcat
    mkdir /usr/local/src/redis
    mkdir /usr/local/src/redis-install
    ```

* 上传包解压

    FileZilla_3.7.3_win32 上传jdk tar包  
    FileZilla_3.7.3_win32 上传mysql包  
    FileZilla_3.7.3_win32 上传tomcat包  
    FileZilla_3.7.3_win32 上传redis包  

### 安装jdk

* 先卸载open-jdk
  * 查看linux上是否存在已经安装好的JDK
    * javac
    * java –version
    * rpm -qa | grep java  
        查看本机上所有已经安装成功的软件,只查看和java相关的

* 删除linux自带jdk

  ```txt
  rpm -e --nodeps java-1.6.0-openjdk-1.6.0.35-1.13.7.1.el6_6.i686
  rpm -e --nodeps java-1.7.0-openjdk-1.7.0.79-2.5.5.4.el6.i686
  ```

* 开始安装

    ```txt
    cd /usr/local/src/java

    # 1. 将jdk压缩包进行解压
    tar  -zxvf   jdk-7u71-linux-i586.tar.gz

    # 2. 安装依赖包 - 需要联网
    yum install glibc.i686

    # 3. 配置环境变量
    vim /etc/profile

    # 末尾添加以下
    #set java environment
    JAVA_HOME=/usr/local/src/java/jdk1.7.0_71
    CLASSPATH=.:$JAVA_HOME/lib.tools.jar
    PATH=$JAVA_HOME/bin:$PATH
    export JAVA_HOME CLASSPATH PATH

    # 4. 保存退出后，使更改的配置生效
    source /etc/profile

    # 5. 验证
    java -version
    ```

* 总结，其实安装jdk就是将相应的文件解压的某一路径下，然后添加依赖，添加环境变量即可

### 安装tomcat

* 开始安装

    ```txt
    # tomcat只要解压就可以使用

    # 1. 进入目录
    cd /usr/local/src/tomcat

    # 2. 解压
    tar -zxvf apache-tomcat-7.0.57.tar.gz

    # 3. 重命名 - 为了好记路径
    mv apache-tomcat-7.0.57 tomcat

    # 4. 启动tomcat -  sh startup.sh | ./startup.sh
    ./startup.sh

    # 5. 查看日志
    tail -f ../logs/catalina.out

    # 6. 打开防火墙
    /sbin/iptables -I INPUT -p tcp --dport 8080 -j ACCEPT
    /etc/rc.d/init.d/iptables save

    # 这里centos 7 不用iptables命令了，用firewall命令
    firewall-cmd --state                       # firewall状态
    firewall-cmd --permanent --add-port=80/tcp # 开放端口
    firewall-cmd --permanent --list-ports      # 查看端口
    firewall-cmd --reload                      # 修改配置后要重启

    # 安装成功 去访问 http://ip:8080
    ```

* 总结，解压-->启动startup.sh-->打开防火墙

### 安装mysql

* 检测是否安装
  
  ```txt
  # 1. rpm检测是否已经安装了MySQL
  rpm -qa | grep mysql

  # 2. rpm如果已经安装了，将其卸载
  rpm -e --nodeps  mysql-libs-5.1.73-5.el6_6.i686

  # 3. yum检测是否安装了mysql
  yum list | grep mysql

  # 4. yum卸载 - 解决冲突问题
  yum remove mysql-libs
  ```

* 安装mysql - 这里善用whereis
  
    ```txt
    # 1. 切换文件位置
    cd /usr/local/src/mysql

    # 2. 解压
    tar -xvf MySQL-5.6.22-1.el6.i686.rpm-bundle.tar

    # 3. 安装server
    rpm -ivh MySQL-server-5.6.22-1.el6.i686.rpm

    # 注意！安装的时候提示缺什么依赖，补什么
    yum -y install libaio.so.1 libgcc_s.so.1 libstdc++.so.6 perl net-tools autoconf

    # 4. 安装
    rpm -ivh MySQL-client-5.6.22-1.el6.i686.rpm

    # 这里也要注意，缺什么补什么
    yum -y install libncurses.so libtinfo.so.5

    # 5. 查询mysql服务运行状态
    service mysql status

    # 启动mysql服务
    service mysql start
    # 注意这里启动不出来，可能是权限问题

    # 这里的解决办法是
    chmod -R 777 /var/lib/mysql

    # 6. 去直接设置密码 - 在安装mysql-server的时候提示
    /usr/bin/mysqladmin -u root password 'new-password'

    # 这个命令也可以在 mysql_install_db 重置 执行后找到

    # 7.系统启动时自动启动MySQL服务
    chkconfig --add mysql # 加入到系统服务
    chkconfig mysql on    # 自动启动
    chkconfig             # 查询列表
    ```

* 开启远程访问

    ```txt
    # 1. 在mysql下执行以下命令
    grant all privileges on *.* to 'root' @'%' identified by '1234';
    flush privileges;

    # 2. 防火墙打开3306端口
    # 这里centos 7 不用iptables命令了，用firewall命令
    firewall-cmd --state                         # firewall状态
    firewall-cmd --permanent --add-port=3306/tcp # 开放端口
    firewall-cmd --permanent --list-ports        # 查看端口
    firewall-cmd --reload                        # 修改配置后要重启
    ```

---

* 关于chkconfig命令 - 参考以下

    ```txt
    [root@localhost ~]$ ls /etc/init.d/httpd     # /etc/init.d/目录下必须有启动脚本
    [root@localhost ~]$ chkconfig --add httpd    # 添加服务，以便让chkconfig指令管理它
    [root@localhost ~]$ chkconfig httpd on       # 设置开机运行该服务，默认是设置2345等级开机运行服务

    [root@localhost ~]$ chkconfig --list                 # 列出所有被chkconfig管理的服务
    [root@localhost ~]$ chkconfig --add httpd            # 添加指定的服务，让chkconfig指令管理它
    [root@localhost ~]$ chkconfig --del httpd            # 删除指定的服务，不再让chkconfig指令管理它
    [root@localhost ~]$ chkconfig httpd on               # 设置开机运行服务，需要先执行 --add 才能执行该命令
    [root@localhost ~]$ chkconfig httpd off              # 设置开机不运行服务，需要先执行 --add 才能执行该命令
    [root@localhost ~]$ chkconfig --level 35 httpd on    # 设置服务在等级3和5时开机运行服务，默认是设置2345等级开机运行服务

    复制代码

    [root@localhost ~]$ chkconfig --list                                      # 等级0：关机
    atop            0:off   1:off   2:off   3:off   4:off   5:off   6:off     # 等级1：单用户模式/救援模式
    auditd          0:off   1:off   2:off   3:off   4:on    5:off   6:off     # 等级2：无网络连接的多用户命令行模式
    crond           0:off   1:off   2:on    3:on    4:on    5:on    6:off     # 等级3：有网络连接的多用户命令行模式
    ipset           0:off   1:off   2:on    3:on    4:on    5:on    6:off     # 等级4：不可用
    iptables        0:off   1:off   2:off   3:off   4:on    5:off   6:off     # 等级5：带图形界面的多用户模式
    mysql           0:off   1:off   2:on    3:on    4:on    5:on    6:off     # 等级6：重启
    ```

### 安装redis

* 开始安装

    ```txt
    # 1. 切换有压缩包的目录
    cd /usr/local/src/redis

    # 2. 解压redis
    tar -zxvf redis-3.0.7.tar.gz

    # 3. 安装gcc
    yum -y install gcc

    # 4. 切换redis解压后的目录，编译
    cd redis-5.0.7
    make # 编译

    # 5. 安装到/usr/local/redis目录
    make PREFIX=/usr/local/redis install

    # 6.由于redis启动需要一个配置文件，将配置文件复制到
    cp /usr/local/src/redis/redis-5.0.7/redis.conf /usr/local/redis/bin

    # 7. 修改conf # daemonize yes
    # 在vi里面命令行模式 /string --->可以查找某个字符串
    vi redis.conf

    # 8. 启动服务器
    ./bin/redis-server   ./redis.conf

    # 9.测试是否成功
    ping         # 服务器返回 pong
    set name tom # 设置key=name value=tom的键值对
    get name     # 获取key为name的值
    keys *       # 向服务器查看服务器一共有多少键值对
    ```

* 远程连接

    ```txt
    # 1.在redis.conf文件找到如下配置 - bind后面写上ip，并设置密码
    bind 127.0.0.1           # 设置IP
    requirepass yourpassword # 设置密码

    # 要开防火墙 - redis默认端口 6379
    firewall-cmd --permanent --add-port=6379/tcp # 开放端口
    firewall-cmd --permanent --list-ports        # 查看端口
    firewall-cmd --reload                        # 重新加载

    # windos -h主机 -p端口 -a密码
    E:\redis\redis-cli.exe -h 192.168.80.10 -p 6379 -a helloworld
    ```

> 关于windows客户端去github下
