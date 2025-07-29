# 02-Oracle


oracl 使用

<!--more-->

# 1 Oracle-server 安装

------------------------------------------------------------------------

## 1.1 下载 linux 安装包

------------------------------------------------------------------------

==略......注意 32 位系统还是 64 位系统，选择对应位数的安装包==

Oracle 下载地址：<http://www.oracle.com/technetwork/indexes/downloads/index.html#database>

下载完成后，移动到你觉得方便的位置，进行 unzip 解压：

<figure>
<img
src="https://lbwyjll-public.oss-cn-hangzhou.aliyuncs.com/img/20210205164038.png"
alt="image-20210205164033089" />
<figcaption aria-hidden="true">image-20210205164033089</figcaption>
</figure>

默认会在当前目录下生成一个 database 的目录，解压在其中：

<figure>
<img
src="https://lbwyjll-public.oss-cn-hangzhou.aliyuncs.com/img/20210205164440.png"
alt="image-20210205164437971" />
<figcaption aria-hidden="true">image-20210205164437971</figcaption>
</figure>

## 1.2 创建用户、用户组

------------------------------------------------------------------------

> root 用户下操作

``` bash
# groupadd oinstall                                         #创建用户组
# groupadd dba
# useradd -g oinstall -G dba -m oracle                      #创建oracle用户并指定初始组oinstall,附加组dba
# echo "oracle" > passwd oracle --stdin                     #设置密码
# id oracle                                                 #查看用户相关信息
    uid=1000(oracle) gid=1001(oinstall) 组=1001(oinstall),1002(dba)
```

## 1.3 创建相关目录

------------------------------------------------------------------------

> root 用户下操作

- 创建目录过程可简化：

  `mkdir -pv /u01/app/oracle/{oracle/product/11.2.0/db_1,oraInventory,oradata,fast_recovery_area}`

<figure>
<img
src="https://lbwyjll-public.oss-cn-hangzhou.aliyuncs.com/img/20210205150400.png"
alt="image-20210205150350234" />
<figcaption aria-hidden="true">image-20210205150350234</figcaption>
</figure>

- 以下为单步执行

``` bash
# mkdir -p /u01/app/oracle/oracle/product/11.2.0/db_1       #home目录，安装时指定，base目录                                                                                为/u01/app/oracle/oracle
# mkdir -p /u01/app/oracle/oraInventory                     #配置目录，安装时指定
# mkdir -p /u01/app/oracle/oradata                          #数据目录，创建数据库时指定
# mkdir -p /u01/app/oracle/fast_recovery_area               #数据恢复目录，创建数据库时指定
```

- 属主属组及权限修改

``` bash
# chown -R oracle.oinstall /u01/app/oracle                  #修改属主属组
# chmod -R 755 /u01/app/oracle                              #修改权限
```

## 1.4 修改系统标识（可选）

------------------------------------------------------------------------

Oracle 默认不支持 centos

``` bash
# cat /etc/redhat-release               #没啥影响
```

## 1.5 RAM 与 SWAP 大小要求

------------------------------------------------------------------------

官网说明：<https://docs.oracle.com/cd/E11882_01/install.112/e47689/pre_install.htm#LADBI1085>

<figure>
<img
src="https://lbwyjll-public.oss-cn-hangzhou.aliyuncs.com/img/20210220160940.png"
alt="image-20210220160922071" />
<figcaption aria-hidden="true">image-20210220160922071</figcaption>
</figure>

- 内存要求

  内存最小 1G，推荐 2G 或者更高；

- swap 大小要求

| **RAM**   | **Swap**    |
|-----------|-------------|
| 1G 至 2G  | 1.5 倍      |
| 2G 至 16G | 同 RAM 相等 |
| 16G 以上  | 16G         |

查看如下：`free -h`

<figure>
<img
src="https://lbwyjll-public.oss-cn-hangzhou.aliyuncs.com/img/20210220162416.png"
alt="image-20210220161637552" />
<figcaption aria-hidden="true">image-20210220161637552</figcaption>
</figure>

swap 大小扩展：

- 存储设备无论是分区还是 lvm，若无可再分配的分区、lvm，亦可生成所需大小的文件，再格式化为 swap 分区：

``` bash
# dd if=/dev/zero of=/swapadd bs=1MB count=1024             #此处为1G
# mkswap /swapadd
# chmod 600 /swapadd
# swapon /swapadd                                           #使其生效
    重启系统虚拟空间会消失，打开/etc/fstab文件，加入/swapadd swap swap defaults 0 0
```

- 如下为临时启用，fstab 文件挂载时，可以文件名，亦可 UUID 挂载：

<figure>
<img
src="https://lbwyjll-public.oss-cn-hangzhou.aliyuncs.com/img/20210220164805.png"
alt="image-20210220164801923" />
<figcaption aria-hidden="true">image-20210220164801923</figcaption>
</figure>

## 1.6 安装依赖

------------------------------------------------------------------------

具体的依赖包参考说明：<https://docs.oracle.com/cd/E11882_01/install.112/e47689/pre_install.htm#LADBI1085>

<figure>
<img
src="https://lbwyjll-public.oss-cn-hangzhou.aliyuncs.com/img/20210220162425.png"
alt="image-20210205165914654" />
<figcaption aria-hidden="true">image-20210205165914654</figcaption>
</figure>

``` bash
# yum -y install binutils* compat-libcap1* compat-libstdc++* gcc* gcc-c++* glibc* glibc-devel* ksh* libaio* libaio-devel* libgcc* libstdc++* libstdc++-devel* libXi* libXtst* make* sysstat* elfutils* unixODBC* 
```

## 1.7 修改内核参数

------------------------------------------------------------------------

官方参数设置指南：<https://docs.oracle.com/cd/E11882_01/install.112/e47689/pre_install.htm#LADBI1187>

- 以下示例中部分参数以 2G 内存值进行计算

``` bash
# vim /etc/sysctl.conf                              #修改内核参数，添加以下内容
##############################################################
net.ipv4.icmp_echo_ignore_broadcasts = 1            
fs.file-max = 6815744                               
fs.aio-max-nr = 1048576                             
kernel.shmall = 2097152                             
kernel.shmmax = 1073741824                          #2G*1024*1024*1024*0.5
kernel.shmmni = 4096                                
kernel.sem = 250 32000 100 128
net.ipv4.ip_local_port_range = 9000 65500           
net.core.rmem_default = 262144
net.core.rmem_max= 4194304
net.core.wmem_default= 262144
net.core.wmem_max= 1048576
##############################################################
# sysctl -p                                         #使配置生效
```

执行如下

<figure>
<img
src="https://lbwyjll-public.oss-cn-hangzhou.aliyuncs.com/img/20210223134914.png"
alt="image-20210223134905314" />
<figcaption aria-hidden="true">image-20210223134905314</figcaption>
</figure>

参数详解：

| 参数 | 含义及计算方式 |
|----|----|
| net.ipv4.icmp_echo_ignore_broadcasts = 1 | 允许 ping 操作； |
| fs.file-max = 6815744 | 系统中可以同时打开的文件数目。其值相当于 6.5×1024×1024=6.5M； |
| fs.aio-max-nr = 1048576 | 同时可以拥有的的异步 IO 请求数目，推荐值是：1048576，其实它等于 1024\*1024 也就是 1024K 个； |
| kernel.shmall = 2097152 | 共享内存的总量（所有内存大小，单位：页，1 页=4KB=4000 字节），2G 内存设置计算公式：2\*1024\*1024\*1024/4000(页)；==此处安装所需最小值== |
| kernel.shmmax = 1073741824 | 单个共享内存段的大小（单位：字节）限制，一般为共享内存总量的 50%\~80%，2G 内存设置计算公式 2\*1024\*1024\*1024\*0.5； |
| kernel.shmmni = 4096 | 表示单个共享内存段的最小值，一般为 4kB，即 4000bit； |
| kernel.sem = 250 32000 100 128 | 表示设置的信号量，这 4 个参数内容大小固定；（每个信号对象集的最大信号对象数；系统范围内最大信号对象数；每个信号对象支持的最大操作数；系统范围内最大信号对象集数） |
| net.ipv4.ip_local_port_range = 9000 65500 | 可使用的 IPv4 端口范围（临时端口范围至少从 9000 及以上开始）； |
| net.core.rmem_default = 262144 | 表示接收套接字缓冲区大小的缺省值（以字节为单位）； |
| net.core.rmem_max = 4194304 | 表示内核套接字接受缓存区的最大大小（以字节为单位）； |
| net.core.wmem_default = 262144 | 表示内核套接字发送缓存区的缺省值（以字节为单位）； |
| net.core.wmem_max = 1048576 | 表示内核套接字发送缓存区的最大大小（以字节为单位）； |

## 1.8 修改资源限制

------------------------------------------------------------------------

``` bash
# vim /etc/security/limits.conf             #修改资源限制，添加以下参数配置
oracle soft nproc 16384                     #单个用户可用的最大进程数量(软限制，默认4096)
oracle hard nproc 16384                     #单个用户可用的最大进程数量(硬限制，默认4096)
oracle soft nofile 65536                    #用户进程可打开的文件描述符的最大数(软限制，默认1024)
oracle hard nofile 65536                    #用户进程可打开的文件描述符的最大数(硬限制，默认1024)
oracle soft stack 10000                     #软堆栈数(默认8192，单位KB)
oracle hard stack 10000                     #软堆栈数(默认8192，单位KB)
```

## 1.9 修改用户环境变量

------------------------------------------------------------------------

``` bash
# vim /home/oracle/.bash_profile                        #修改oracle用户家目录下的.bash_profile文件，添加以下内容
export ORACLE_BASE=/u01/app/oracle                      #oracle数据库安装目录
export ORACLE_HOME=$ORACLE_BASE/product/11.2.0/db_1     #oracle数据库路径
export ORACLE_SID=orcl                                  #oracle启动数据库实例名
export ORACLE_UNQNAME=orcl
###export ORACLE_TERM=xterm                             #xterm窗口模式安装
export PATH=$ORACLE_HOME/bin:/usr/sbin:$PATH            #添加系统环境变量
export LD_LIBRARY_PATH=$ORACLE_HOME/lib:/usr/lib        #添加系统环境变量
export NLS_LANG='SIMPLIFIED CHINESE_CHINA.AL32UTF8'     #设置Oracle客户端字符集，必须与Oracle安装时设置的字符集保持一致
# source /home/oracle/.bash_profile                     #使配置文件立即生效
```

## 1.10 静默安装配置

------------------------------------------------------------------------

在安装包解压目录/utxt/oracle/database/database 下的 reponse 目录下，存在三个应答文件，用来作为静默安装时的应答文件的模板。三个文件作用分别是:

- db_install.rsp：安装应答
- dbca.rsp：创建数据库应答
- netca.rsp：建立监听、本地服务名等网络设置的应答

<img src="https://lbwyjll-public.oss-cn-hangzhou.aliyuncs.com/img/20210121002356.png" alt="image-20201224162442886" style="zoom:200%;" />

### 1.10.1 修改安装应答文件 db_install.rsp

``` bash
# cd /utxt/oracle/database/database/reponse         #此处可变，安装包解压在哪就去哪里
# vim db_install.rsp

oracle.install.responseFileVersion=/oracle/install/rspfmt_dbinstall_response_schema_v11_2_0
oracle.install.option=INSTALL_DB_SWONLY
ORACLE_HOSTNAME=oracle      #当前主机名      
UNIX_GROUP_NAME=oinstall    #用户组名
INVENTORY_LOCATION=/u01/app/oracle/oraInventory   #oracle数据库配置文件目录
SELECTED_LANGUAGES=en,zh_CN         #安装编码
ORACLE_HOME=/u01/app/oracle/oracle/product/11.2.0/db_1      #oracle数据库路径
ORACLE_BASE=/utxt/oracle/oracle     #Oracle安装目录
oracle.install.db.InstallEdition=EE
oracle.install.db.isCustomInstall=false
oracle.install.db.DBA_GROUP=dba
oracle.install.db.OPER_GROUP=dba
oracle.install.db.CLUSTER_NODES=
oracle.install.db.config.starterdb.type=GENERAL_PURPOSE
oracle.install.db.config.starterdb.globalDBName= orcl
oracle.install.db.config.starterdb.SID= orcl
oracle.install.db.config.starterdb.characterSet=AL32UTF8
oracle.install.db.config.starterdb.memoryOption=true
oracle.install.db.config.starterdb.memoryLimit=1500
oracle.install.db.config.starterdb.installExampleSchemas=false
oracle.install.db.config.starterdb.enableSecuritySettings=true
oracle.install.db.config.starterdb.password.ALL=Witsky_2020
oracle.install.db.config.starterdb.password.SYS=
oracle.install.db.config.starterdb.password.SYSTEM=
oracle.install.db.config.starterdb.password.SYSMAN=
oracle.install.db.config.starterdb.password.DBSNMP=
oracle.install.db.config.starterdb.control=DB_CONTROL
oracle.install.db.config.starterdb.gridcontrol.gridControlServiceURL=
oracle.install.db.config.starterdb.dbcontrol.enableEmailNotification=false
oracle.install.db.config.starterdb.dbcontrol.emailAddress=
oracle.install.db.config.starterdb.dbcontrol.SMTPServer=
oracle.install.db.config.starterdb.automatedBackup.enable=false
oracle.install.db.config.starterdb.automatedBackup.osuid=
oracle.install.db.config.starterdb.automatedBackup.ospwd=
oracle.install.db.config.starterdb.storageType=FILE_SYSTEM_STORAGE
oracle.install.db.config.starterdb.fileSystemStorage.dataLocation=
oracle.install.db.config.starterdb.fileSystemStorage.recoveryLocation=
oracle.install.db.config.asm.diskGroup=
oracle.install.db.config.asm.ASMSNMPPassword=
MYORACLESUPPORT_USERNAME=
MYORACLESUPPORT_PASSWORD=
SECURITY_UPDATES_VIA_MYORACLESUPPORT=
DECLINE_SECURITY_UPDATES=true    #一定要设为true
PROXY_HOST=
PROXY_PORT=
PROXY_USER=
PROXY_PWD=
```

### 1.10.2 静默安装

- 在上级路径下执行安装脚本：`runInstaller`

<img src="https://lbwyjll-public.oss-cn-hangzhou.aliyuncs.com/img/20210121002357.png" alt="image-20201224170049026" style="zoom:200%;" />

``` bash
# rpm -i --force --nodeps pdksh-5.2.14-30.x86_64.rpm
# ./runInstaller -silent -responseFile /u01/app/oracle/database/response/db_install.rsp
```

- 执行安装脚本后，根据提示查看日志，有些警告可以忽略，如果有报错，要定位并解决问题：

<figure>
<img
src="https://lbwyjll-public.oss-cn-hangzhou.aliyuncs.com/img/20210223164015.png"
alt="image-20210223164014125" />
<figcaption aria-hidden="true">image-20210223164014125</figcaption>
</figure>

- 以上示例已安装成功，但是有警告，虽然可忽略，但最好是警告都不要有！！！

`less /u01/app/oracle/oraInventory/logs/installActions2021-02-23_04-33-24PM.log`

- 以关键字==Error== 或 ==FAILED== 进行搜索查找，锁定问题后进行对应修改即可

<figure>
<img
src="https://lbwyjll-public.oss-cn-hangzhou.aliyuncs.com/img/20210223163906.png"
alt="image-20210223163859376" />
<figcaption aria-hidden="true">image-20210223163859376</figcaption>
</figure>

- 修改后重新安装如下：

<figure>
<img
src="https://lbwyjll-public.oss-cn-hangzhou.aliyuncs.com/img/20210223165218.png"
alt="image-20210223165134222" />
<figcaption aria-hidden="true">image-20210223165134222</figcaption>
</figure>

- 切换至 root 用户执行对应脚本：

``` bash
# sh /u01/app/oracle/oraInventory/orainstRoot.sh
# sh /u01/app/oracle/oracle/product/11.2.0/db_1/root.sh
```

<figure>
<img
src="https://lbwyjll-public.oss-cn-hangzhou.aliyuncs.com/img/20210223165429.png"
alt="image-20210223165427066" />
<figcaption aria-hidden="true">image-20210223165427066</figcaption>
</figure>

- `sqlplus / as sysdba` 测试

# 2 Oracle-server 配置

------------------------------------------------------------------------

## 2.1 创建数据库

------------------------------------------------------------------------

### 2.1.1 DBCA 建库

- 编辑==dbca.rsp== 响应文件设置以下参数：

``` bash
# cd /u01/app/oracle/database/response       ##此处可变，安装包解压在哪就去哪里
# vim dbca.rsp
GDBNAME= "orcl"                                   ##78行
SID ="orcl"                                   ##170行
SYSPASSWORD= "Witsky_2020"                        ##211行
SYSTEMPASSWORD= "Witsky_2020"                     ##221行
SYSMANPASSWORD= "Witsky_2020"                     ##252行
DBSNMPPASSWORD= "Witsky_2020"                     ##262行
DATAFILEDESTINATION=/u01/app/oracle/oradata       ##360行
RECOVERYAREADESTINATION=/u01/app/oracle/fast_recovery_area      ##370行
CHARACTERSET= "AL32UTF8"                          ##与安装时一致,418行
TOTALMEMORY= "1638"                               ##其中TOTALMEMORY ="1638" 为1638MB，物理内存2G*80%,553行
```

- 修改完成后执行以下命令

``` bash
# dbca -silent -responseFile /u01/app/oracle/database/response/dbca.rsp             #dbca命令为安装后生成，环境变量正确即可使用
```

<figure>
<img
src="https://lbwyjll-public.oss-cn-hangzhou.aliyuncs.com/img/20210223171656.png"
alt="image-20210223171653145" />
<figcaption aria-hidden="true">image-20210223171653145</figcaption>
</figure>

- 实例测试：`sqlplus / as sysdba` ，`select status from v$instance;`

<figure>
<img
src="https://lbwyjll-public.oss-cn-hangzhou.aliyuncs.com/img/20210223172248.png"
alt="image-20210223172245839" />
<figcaption aria-hidden="true">image-20210223172245839</figcaption>
</figure>

- 实例进程查看：`ps -ef | grep ora`

<figure>
<img
src="https://lbwyjll-public.oss-cn-hangzhou.aliyuncs.com/img/20211128194140.png"
alt="image-20210223172024219" />
<figcaption aria-hidden="true">image-20210223172024219</figcaption>
</figure>

### 2.1.2 手工建库

以 oracle 用户登录，进入到 \$ORACLE_HOME/dbs/下，创建 initcrnophq.ora 文件

``` bash
# cd $ORACLE_HOME/dbs/
# vim initcrnophq.ora
db_name=orcl                # 数据库名
memory_target=20G           #
```

启动实例

``` bash
# sqlplus

```

## 2.2 配置监听

------------------------------------------------------------------------

使用静默模式运行 netca 命令去配置并启动 Oracle 网络监听（ listener.ora ）、配置命名方式和配置配置网络服务名（ tnsnames.ora ）。

### 2.2.1 动态注册

数据库建立后无需修改

参考：[oracle 动态注册和静态注册【图文】\_rm_rf_db_51CTO博客](https://blog.51cto.com/u_12185273/2061784)

### 2.2.2 静态注册 listener.ora 文件

修改==netca.rsp==文件（其实默认就好）

``` bash
# vim netca.rsp 
[GENERAL]
RESPONSEFILE_VERSION="11.2"
CREATE_TYPE="CUSTOM"
# 是否安装图形界面
#SHOW_GUI=false
# 设置日志文件
#LOG_FILE=""/oracle11gHome/network/tools/log/netca.log""

[oracle.net.ca]
#INSTALLED_COMPONENTS;StringList;list of installed components
# The possible values for installed components are:
# "net8","server","client","aso", "cman", "javavm" 
# 安装的组件
INSTALLED_COMPONENTS={"server","net8","javavm"}

#INSTALL_TYPE;String;type of install
# The possible values for install type are:
# "typical","minimal" or "custom"
# 安装的类型
INSTALL_TYPE=""typical""

#LISTENER_NUMBER;Number;Number of Listeners
# A typical install sets one listener 
# 监听器的个数
LISTENER_NUMBER=1

#LISTENER_NAMES;StringList;list of listener names
# The values for listener are:
# "LISTENER","LISTENER1","LISTENER2","LISTENER3", ...
# A typical install sets only "LISTENER" 
# 监听器的名字，默认是监听器1，监听器2...
LISTENER_NAMES={"LISTENER"}

#LISTENER_PROTOCOLS;StringList;list of listener addresses (protocols and parameters separated by semicolons)
# The possible values for listener protocols are:
# "TCP;1521","TCPS;2484","NMP;ORAPIPE","IPC;IPCKEY","VI;1521" 
# A typical install sets only "TCP;1521" 
# 监听地址，默认监听tcp的1521
LISTENER_PROTOCOLS={"TCP;1521"}

#LISTENER_START;String;name of the listener to start, in double quotes
# 启动监听器的名字
LISTENER_START=""LISTENER""

#NAMING_METHODS;StringList;list of naming methods
# The possible values for naming methods are: 
# LDAP, TNSNAMES, ONAMES, HOSTNAME, NOVELL, NIS, DCE
# A typical install sets only: "TNSNAMES","ONAMES","HOSTNAMES" 
# or "LDAP","TNSNAMES","ONAMES","HOSTNAMES" for LDAP
NAMING_METHODS={"TNSNAMES","ONAMES","HOSTNAME"}

#NOVELL_NAMECONTEXT;String;Novell Directory Service name context, in double quotes
# A typical install does not use this variable. 
#NOVELL_NAMECONTEXT = ""NAMCONTEXT""

#SUN_METAMAP;String; SUN meta map, in double quotes
# A typical install does not use this variable. 
#SUN_METAMAP = ""MAP""

#DCE_CELLNAME;String;DCE cell name, in double quotes
# A typical install does not use this variable. 
#DCE_CELLNAME = ""CELL""

#NSN_NUMBER;Number;Number of NetService Names
# A typical install sets one net service name
NSN_NUMBER=1

#NSN_NAMES;StringList;list of Net Service names
# A typical install sets net service name to "EXTPROC_CONNECTION_DATA"
NSN_NAMES={"EXTPROC_CONNECTION_DATA"}

#NSN_SERVICE;StringList;Oracle11g database's service name
# A typical install sets Oracle11g database's service name to "PLSExtProc"
NSN_SERVICE={"PLSExtProc"}

#NSN_PROTOCOLS;StringList;list of coma separated strings of Net Service Name protocol parameters
# The possible values for net service name protocol parameters are:
# "TCP;HOSTNAME;1521","TCPS;HOSTNAME;2484","NMP;COMPUTERNAME;ORAPIPE","VI;HOSTNAME;1521","IPC;IPCKEY"  
# A typical install sets parameters to "IPC;EXTPROC"
NSN_PROTOCOLS={"TCP;HOSTNAME;1521"}
```

执行命令：`netca -silent -responsefile /u01/app/oracle/database/response/netca.rsp`

    [oracle@localhost response]$ netca -silent -responsefile /u01/app/oracle/database/response/netca.rsp
    正在对命令行参数进行语法分析:
    参数"silent" = true
    参数"responsefile" = /u01/app/oracle/database/response/netca.rsp
    完成对命令行参数进行语法分析。
    Oracle Net Services 配置:
    完成概要文件配置。
    Oracle Net 监听程序启动:
        正在运行监听程序控制: 
          /u01/app/oracle/oracle/product/11.2.0/db_1/bin/lsnrctl start LISTENER
        监听程序控制完成。
        监听程序已成功启动。
    监听程序配置完成。
    成功完成 Oracle Net Services 配置。退出代码是0

> 执行完毕后会生成 listener.ora 监听文件

查看监听状态：`lsnrctl status`

<figure>
<img
src="https://lbwyjll-public.oss-cn-hangzhou.aliyuncs.com/img/20211128194052.png"
alt="image-20211126111112021" />
<figcaption aria-hidden="true">image-20211126111112021</figcaption>
</figure>

若出现==" 监听程序不支持服务 "==，则是在 listener.ora 文件中缺失了配置信息，手动加入如下信息：

问题 listener.ora 文件内容

<figure>
<img
src="https://lbwyjll-public.oss-cn-hangzhou.aliyuncs.com/img/20211128194056.png"
alt="image-20211126112113869" />
<figcaption aria-hidden="true">image-20211126112113869</figcaption>
</figure>

加入如下信息：

``` bash
SID_LIST_LISTENER =
  (SID_LIST =
    (SID_DESC=
     (GLOBAL_DBNAME = orcl)
     (ORACLE_HOME = )
     (SID_NAME = orcl)
    )
  )
```

GLOBAL_DBNAME：oracle 数据库实例名（==区分大小写==）

ORACLE_HOME：oracle 数据库安装路径

SID_NAME：sid（==区分大小写==）

### 2.2.3 网络服务名 tnsname.ora 文件

``` bash
cd $ORACLE_HOME/network/admin
cp samples/tnsnames.ora ./
```

编辑 tnsnames.ora 文件

    orcl =
      (DESCRIPTION =
        (ADDRESS = (PROTOCOL = TCP)(HOST = 192.168.137.129)(PORT = 1521))
        (CONNECT_DATA =
          (SERVER = DEDICATED)
          (SERVICE_NAME = orcl)
        )
      )

### 2.2.4 监听问题------监听程序不支持服务

------------------------------------------------------------------------

参考链接：

[Linux Oracle监听配置文件修改IP及添加实例监听](https://blog.csdn.net/weixin_30616969/article/details/98420501)

[Oracle监听出现的问题总结，以及解决办法](https://blog.csdn.net/forever_river/article/details/55662161)

# 3 sqlplus 使用

------------------------------------------------------------------------

## 3.1 登录 oracle

------------------------------------------------------------------------

> 1.  ==sqlplus / as sysdba==

操作系统认证，不需要数据库服务器启动 listener，也不需要数据库服务器处于可用状态。比如我们想要启动数据库就可以用这种方式进入 sqlplus，然后通过==startup==命令来启动。

> 2.  ==sqlplus username/password==

连接本机数据库，不需要数据库服务器的 listener 进程，但是由于需要用户名密码的认证，因此需要数据库服务器处于可用状态才行。

> 3.  ==sqlplus usernaem/password@host==

通过网络连接，这是需要数据库服务器的 listener 处于监听状态。此时建立一个连接的大致步骤如下:

- 查询 sqlnet.ora，看看名称的解析方式，默认是 TNSNAME（安装时指定的 hostname，反解 hosts 文件）；
- 查询 tnsnames.ora 文件，从里边找 orcl 的记录，并且找到数据库服务器的主机名或者 IP，端口和 service_name；　
- 如果服务器 listener 进程没有问题的话，建立与 listener 进程的连接；
- 根据不同的服务器模式如专用服务器模式或者共享服务器模式，listener 采取接下去的动作。默认是专用服务器模式，没有问题的话客户端就连接上了数据库的 server process；
- 这时连接已经建立，可以操作数据库了。

> 4.  ==sqlplus username/password@host:port/service_name==

注意： 以 sys 用户登陆的话 必须要加上 as sysdba 子句 ，例如：`sqlplus sys/sys@127.0.0.1:1521/orcl as sysdba`

## 3.2 数据库 server 启停

------------------------------------------------------------------------

### 3.2.1 启动数据库

- 启动数据库命令为==startup;==，完整的启动过程分为三个步骤：启动实例 -----\> 加载数据库 ------\> 打开数据库

- 启动数据库分为以下几种模式：

  1.  nomount 状态------启动实例，但不加载数据库，但会自动创建跟踪文件。命令为 ==startup nomount;==通过语句查询出数据库当前的状态：`select status from v$instance;`

  > 数据库在 nomount 状态下可以做以下事情：
  >
  > - 只能访问那些与 SGA 区相关的数据字典视图，包括 V$PARAMETER、V$SGA、V$PROCESS 和V$SESSION 等，这些视图中的信息都是从 SGA 区中获取的，与数据库无关；
  > - 可以创建新数据库；
  > - 可以重建控制文件；

  2.  mount 模式------加载数据库却不打开数据库，命令为==alter database mount;== 如果在数据库完全没有启动的情况下是可以直接使用==startup mount==; 来把数据库启动到 mount 状态

  > 数据库在 mount 状态下可以做以下事情：
  >
  > - 在 mount 模式下，只能访问那些与控制文件相关的数据字典视图，包括 V$THREAD、V$CONTROLFILE、V$DATABASE、V$DATAFILE 和 V\$LOGFILE 等；
  > - 重命名数据文件；
  > - 添加、删除或重命名重做日志文件；
  > - 执行数据库完全备份与恢复操作；
  > - 改变数据库的归档模式。

  3.  open 模式------打开数据库，命令==alter databse open==; 如果在数据库完全没有启动的情况下是可以直接使用==startup== (open); 来把数据库直接启动的。

  > 数据库启动到 open 状态后可以做以下事情：
  >
  > - 创建数据库对象，比如表空间、视图、序列 、用户等，并根据权限对所创建的对象进行修改和删除操作；

### 3.2.2 关闭数据库与实例

- 关闭数据库与实例也分为 3 步：关闭数据库 --\>实例卸载数据库 ---\>终止实例，关闭模式分为以下几种：

  1.  Normal（正常关闭方式）， 建议使用这种方式进行数据库的关闭操作。命令：==shutdown normal==

  > 正常方式关闭数据时，Oracle 执行如下操作：
  >
  > - 阻止任何用户建立新的连接。
  >
  > - 等待当前所有正在连接的用户主动断开连接
  >
  > - 一旦所有的用户都断开连接，则立即关闭、卸载数据库，并终止实例

  2.  Immediate（立即关闭方式），命令：==shutdown immediate==

  > 立即关闭数据时，Oracle 执行如下操作：
  >
  > - 阻止任何用户建立新的连接，同时阻止当前连接的用户开始任何新的事务。
  >
  > - Oracle 不等待在线用户主动断开连接，强制终止用户的当前事务，将任何未提交的事务回退
  >
  > - 直接关闭、卸载数据库，并终止实例。

  3.  Transactional（事务关闭方式），命令：==shutdown transactional== ，这种方式介于正常关闭方式跟立即关闭方式之间，响应时间会比较快，处理也将比较得当。

  > Oracle 执行如下操作：
  >
  > - 阻止任何用户建立新的连接，同时阻止当前连接的用户开始任何新的事务。
  >
  > - 等待所有未提交的活动事务提交完毕，然后立即断开用户的连接。
  >
  > - 直接关闭、卸载数据库，并终止实例。

  4.  Abort（终止关闭方式），命令：==shutdown abort== ，这是比较粗暴的一种关闭方式，当前面 3 种方式都无法关闭时，可以尝试使用终止方式来关闭数据库。但是以这种方式关闭数据库将==会丢失一部份数据信息==，当重新启动实例并打开数据库时，后台进程 SMON 会执行实例恢复操作。一般情况下，应当尽量避免使用这种方式来关闭数据库。

  > 执行过程如下：
  >
  > - 阻止任何用户建立新的连接，同时阻止当前连接的用户开始任何新的事务；
  >
  > - 立即终止当前正在执行的 SQL 语句；
  >
  > - 任何未提交的事务均不被退名；
  >
  > - 直接断开所有用户的连接，关闭、卸载数据库，并终止实例；

## 3.3 执行单次指令后退出

------------------------------------------------------------------------

通过脚本实现：

``` plsql
#!/bin/bash
export ORACLE_HOME=/oracle/product/10.2.0/db_1
export NLS_LANG='SIMPLIFIED CHINESE_CHINA.AL32UTF8'

sqlplus -s 'USER/PASSWORD@[IP:PORT]SERVICE_NAME' [>> PATH_FILE]  <<EOF   
set feed off;
set heading off;
set feedback off;
set verify off;
select * from TABLE;
exit;
EOF
```

> 可借助重定向导出 sql 语句结果至本地文件中

==注意==：脚本使用时记得声明环境变量信息，以防出现编码错误

## 3.4 常用 set 指令

``` plsql
set pagesize 0              --输出每页行数，缺省为24,为了避免分页，可设定为0
set linesize 2000           --输出一行字符个数，缺省为80
set long 2000               --设置CLOB、LONG、NLOB和XMLType类型的最大显示字节数,该参数的最大值是2000000000（2G）
set longchunksize 255       --sqlplus会按照这个参数的值来一段一段的获取上述类型的数据，直到达到long的值或者数据获取完毕。
set head off                --数据库查询结果中不显示列标题，而是以空白行代替，也就是不显示字段名了
set termout off             --输出的内容只保存在输出文件中，不会显示在屏幕上，提高SPOOL输出速度。
set trims on                --去掉空字符
set trim on                 --查询结果既显示于假脱机文件中，又在SQLPLUS中显示
set feedback off;           --默认的当一条sql发出的时候，oracle会给一个反馈，比如说创建表的时候，如果成功命令行会返回类                             似;Table created的反馈,off后不显示反馈
set feed off;               --同set feedback off，显示数量 当查询选择至少n条记录时，查询返回的记录

set newpage 0;              --设置页与页之间的分隔。0时，会在每页的开头有一个小的黑方框；n时，会在页版和页之间隔着n个空                             行
set space 0;                --设置各列间的空格数，默认不用设置、不用写此参数
set line 1000;              --设置行的长度
set echo off;               --显示start启动的脚本中的每个sql命令，缺省为on

set feedback on;            --设置显示“已选择XX行”
set heading off;            --输出域标题，缺省为on；不显示表头信息;pagesize为0时不显示表头
set term off;               --查询结果仅仅显示于假脱机文件中
set termout off;            --不在屏幕上显示结果
set trimout on;             --每一显示行的末端去掉空格
set trimspool on;           --去除重定向（spool）输出每行的拖尾空格，缺省为off
set timing off;             --显示每条sql命令的耗时，缺省为off
set timing on;              --设置显示“已用时间：XXXX”
set time on;                --设置显示当前时间
set trimout on;             --去除标准输出每行的拖尾空格，缺省为off
set autotrace on;           --设置允许对执行的sql进行分析
set colsep ' ';             --设置分隔符为空格
set numwidth 12;            --输出number类型域长度，缺省为10
set serveroutput on;        --设置允许显示输出类似dbms_output
set verify off;             --可以关闭和打开提示确认信息old 1和new 1的显示.
set sqlblanklines on;       --正常情况下，在SQLPLUS中输入命令时，可以换行，但不能有空格，否则不能执行，会直接返回到SQL>下。但通过命令设置可以实现语句换行时允许有空行的情况出现。

--导出表数据为xls或html文件
set term off verify off feedback off pagesize 999
set markup html on entmap ON spool on preformat off


set markup html on          --是否生成html文件格式
set entmap ON               --指定是否用HTML字符实体如&lt;, &gt;, &quot; and &amp;等替换特殊字符<,>,"and & 。默认设置是ON
set spool on                --指定是否生成HTML标签<HTML> 和<BODY>, </BODY> 和</HTML>。默认是OFF。
set preformat off           --指定生成HTML时输出<PRE>标签还是HTML表格，默认是OFF，因此默认输出是写HTML表格。



spool 路径 + 文件名           --记录数据到路径+文件名
select ... from tablename;  --导出数据SQL语句 　　
spool off                   --收集完毕
...
```

# 4 编码格式设置

------------------------------------------------------------------------

当客户端环境中的编码格式、或操作系统和服务端的编码格式不一致时，可能会出现乱码问题：

- 操作系统与服务器一致，但客户端与服务器字符集不一致；
- 客户端与服务器一致，但操作系统与服务器不一致

## 4.1 服务端编码

------------------------------------------------------------------------

### 4.1.1 编码查看

查看服务端字符集及编码：`select userenv('language') from dual;`

![](https://lbwyjll-public.oss-cn-hangzhou.aliyuncs.com/img/20210121002404.png)

### 4.1.2 编码修改

## 4.2 客户端编码

------------------------------------------------------------------------

### 4.2.1 编码查看

查看客户端编码格式：`echo $NLS_LANG`

### 4.2.2 编码修改

在==.bash_profile==中添加相关声明：

``` bash
# vim .bash_profile
    export NLS_LANG='SIMPLIFIED CHINESE_CHINA.AL32UTF8'  
# source .bash_profile
```

## 4.3 操作系统编码

------------------------------------------------------------------------

### 4.3.1 编码查看

操作系统字符集及编码格式查看：`echo $LANG`

### 4.3.2 编码修改

`ll /etc/profile.d`

![](https://lbwyjll-public.oss-cn-hangzhou.aliyuncs.com/img/20210121002403.png)

`vim /etc/profile.d/lang.sh`

![](https://lbwyjll-public.oss-cn-hangzhou.aliyuncs.com/img/20210121102636.png)

修改此处文件即可：`vim /etc/locale.conf`

<img src="https://lbwyjll-public.oss-cn-hangzhou.aliyuncs.com/img/20210121002400.png"  />

# 5 Oracle 用户密码过期时间修改

------------------------------------------------------------------------

Oracle 数据库默认 profile 的密码有效期规则是 default，180 天有效期，到期了之前的密码就不能使用了，必须经过一次修改。这个是为了安全，提示和强制用户每隔一段时间进行一次修改的，但如果只是测试环境或者其它方面原因，也可设置成密码永久有效期。

## 5.1 修改所有用户密码过期时间

------------------------------------------------------------------------

1.  查询当前所有的用户和对应的 profile，默认都是 default，找到我们过期的用户对应的 profile，如果之前没有修改过的话，就是 default

    `select username, profile from dba_users;`

<figure>
<img
src="https://lbwyjll-public.oss-cn-hangzhou.aliyuncs.com/img/20210204111118.png"
alt="image-20210204111116460" />
<figcaption aria-hidden="true">image-20210204111116460</figcaption>
</figure>

2.  查询密码默认过期时间

    `select * from dba_profiles where profile='DEFAULT' and resource_name='PASSWORD_LIFE_TIME';`

<figure>
<img
src="https://lbwyjll-public.oss-cn-hangzhou.aliyuncs.com/img/20210204104307.png"
alt="image-20210204104258944" />
<figcaption aria-hidden="true">image-20210204104258944</figcaption>
</figure>

3.  修改 profile 密码有效期为永久

    `alter profile default limit password_life_time unlimited;`

4.  查询确认修改是否已经完成

    `select * from dba_profiles where profile='DEFAULT' and resource_name='PASSWORD_LIFE_TIME';`

<figure>
<img
src="https://lbwyjll-public.oss-cn-hangzhou.aliyuncs.com/img/20210204104912.png"
alt="image-20210204104909930" />
<figcaption aria-hidden="true">image-20210204104909930</figcaption>
</figure>

- **以上修改之后，不需要重启服务，==立即生效==的。如果之前还没有提醒到期，则当前密码都变成永久了。==如果当前已经提醒过到期了，则需要修改一次密码，才可以正常使用==。**

## 5.2 修改指定用户密码过期时间

------------------------------------------------------------------------

- **上述修改永久密码的方式是==默认针对所有用户==的，如果只想让一个用户的密码为永久，而其它用户不受影，则需要新建一个 profile 给这个特定的用户，然后再修改这个新建的 profile 的密码有效期为永久。**

1.  新建 profile 文件，为了方便，规则可以和 default 一致，只修改密码过期规则：

``` plsql
CREATE PROFILE "TEST_PROFILE" LIMIT
    SESSIONS_PER_USER UNLIMITED
    CPU_PER_SESSION UNLIMITED
    CPU_PER_CALL UNLIMITED
    CONNECT_TIME UNLIMITED
    IDLE_TIME UNLIMITED
    LOGICAL_READS_PER_SESSION UNLIMITED
    LOGICAL_READS_PER_CALL UNLIMITED
    COMPOSITE_LIMIT UNLIMITED
    PRIVATE_SGA UNLIMITED
    FAILED_LOGIN_ATTEMPTS 10
    PASSWORD_LIFE_TIME UNLIMITED                    ----密码过期规则
    PASSWORD_REUSE_TIME UNLIMITED
    PASSWORD_REUSE_MAX UNLIMITED
    PASSWORD_LOCK_TIME 1
    PASSWORD_GRACE_TIME 7
    PASSWORD_VERIFY_FUNCTION NULL;
```

2.  查看新创建的 profile

    `select * from dba_profiles where profile='TEST_PROFILE' and resource_name='PASSWORD_LIFE_TIME';`

3.  将新创建的 TEST_PROFILE 应用到用户上

    `alter user USER profile TEST_PROFILE;` -- 替换 USER

4.  查看修改是否已经完成

    `select username,profile,expiry_date from dba_users where username = 'USERNAME';` -- 替换用户名

5.  删除 profile 文件

- 如果想要删除概要文件，这时因为有用户在使用，需要添加 CASCADE 才能删除，删除之后，使用这个概要文件的用户就会重新使用默认的概要文件:

  `DROP PROFILE TEST_PROFILE CASCADE;` -- 替换 profile 名

- 此时查看

  `select username,profile,expiry_date from dba_users where username = 'USERNAME';` -- 替换用户名

# 6 表空间、用户

------------------------------------------------------------------------

## 6.1 创建 tablespace

------------------------------------------------------------------------

``` plsql
# create tablespace 表空间名 datafile '数据文件名' size 表空间大小;  ---数据文件名需绝对路径
    --例：create tablespace test datafile '/test.dbf' size 5120M;
```

## 6.2 表空间使用量查询

------------------------------------------------------------------------

忽略自动扩容

``` plsql
SELECT df.tablespace_name "表空间名",
       totalspace "总空间M",
       freespace "剩余空间M",
       round((1 - freespace / totalspace) * 100, 2) "使用率%"
  FROM (SELECT tablespace_name, round(sum(bytes) / 1024 / 1024) totalspace
          FROM dba_data_files
         GROUP BY tablespace_name) df,
       (SELECT tablespace_name, round(sum(bytes) / 1024 / 1024) freespace
          FROM dba_free_space
         GROUP BY tablespace_name) fs
 WHERE df.tablespace_name = fs.tablespace_name
 ORDER BY round((1 - freespace / totalspace) * 100, 2) desc;
```

考虑自动扩容

``` plsql
select tablespace_name as "表空间",
       round(sum_max / 1024 / 1024, 1) as "总空间/M",
       round((sum_alloc - nvl(sum_free, 0)) / 1024 / 1024, 1) as "已用空间/M",
       round(100 * (sum_alloc - nvl(sum_free, 0)) / sum_max, 1) As "使用率/%"
  FROM (SELECT tablespace_name,
               sum(bytes) AS sum_alloc,
               sum(decode(maxbytes, 0, bytes, maxbytes)) AS sum_max
          FROM dba_data_files
         GROUP BY tablespace_name),
       (SELECT tablespace_name AS fs_ts_name, sum(bytes) AS sum_free
          FROM dba_free_space
         GROUP BY tablespace_name)
 WHERE tablespace_name = fs_ts_name(+)
 order by "使用率/%" desc;
```

当前表空间下各表占用大小：

``` plsql
select segment_name "表名",
       sum(bytes) / 1024 / 1024 "占用大小"
  from user_extents
 group by segment_name
 order by sum(bytes) / 1024 / 1024 desc
```

## 6.3 表空间扩容

------------------------------------------------------------------------

``` plsql
ALTER TABLESPACE SYSTEM ADD DATAFILE 'test01.dbf' SIZE 2048M;       --SYSTEM为表空间名
```

## 6.4 创建用户

------------------------------------------------------------------------

``` plsql
# create user 用户名 identified by 密码 default tablespace 表空间名;
    --例：create user test identified by Witsky_2020 default tablespace test;
```

## 6.5 表空间实体文件 dbf 查看

------------------------------------------------------------------------

``` plsql
select file_id,
       tablespace_name 表空间,
       file_name DBF文件,
       round(bytes / (1024 * 1024), 0) 文件大小M
  from dba_data_files
 --where tablespace_name = 'UNDOTBS2'
 order by file_id desc;
```

## 6.6 数据文件 dbf 位置修改

------------------------------------------------------------------------

参考链接：<https://www.jb51.net/article/127923.htm>

``` plsql
--表空间脱机
alter tablespace wocao offline;

--复制数据文件至新路径
cp /u01/app/oracle/1.dbf /u01/app/oracle/oradata/orcl/2.dbf

--表空间中dbf文件rename
alter tablespace wocao rename datafile '/u01/app/oracle/1.dbf' to '/u01/app/oracle/oradata/orcl/2.dbf';

--表空间装载
alter tablespace wocao online;
```

## 6.7 授权用户

------------------------------------------------------------------------

``` plsql
 select * from user_sys_privs;                          ---查看当前登录用户权限
 grant create session to xh_asf;                        ---登录授权
 grant connect, resource to xh_asf; 
 grant dba to xh_asf;
```

## 6.8 异常账号查询

------------------------------------------------------------------------

``` plsql
select a.name,
       b.user_id,
       a.password,
       b.account_status,
       a.ctime,
       a.ptime,
       b.profile,
       b.default_tablespace
  from sys.user$ a, dba_users b
 where a. name = b.username
 order by a.ctime desc; 
```

## 6.9 异常 session 会话查询

------------------------------------------------------------------------

``` plsql
select r.root_sid,
       s.serial#,
       r.blocked_num,
       r.avg_wait_seconds,
       s.username,
       s.status,
       s.event,
       s.MACHINE,
       s.PROGRAM,
       s.sql_id,
       s.prev_sql_id
  from (select root_sid,
               avg(seconds_in_wait) as avg_wait_seconds,
               count(*) - 1 as blocked_num
          from (select CONNECT_BY_ROOT sid as root_sid, seconds_in_wait
                  from v$session
                 start with blocking_session is null
                connect by prior sid = blocking_session)
         group by root_sid
        having count(*) > 1) r,
       v$session s
 where r.root_sid = s.sid
 order by r.blocked_num desc, r.avg_wait_seconds desc
```

## 6.10 登录日志查询

------------------------------------------------------------------------

tail -f ./oracle/diag/tnslsnr/witsky-service-bp3/listener/trace/listener.log

## 6.11 查询最近所做操作

------------------------------------------------------------------------

``` plsql
select * from v$sql;
```

## 6.12 UNDOTBS 表空间清理

------------------------------------------------------------------------

UNDOTBS 表空间不能直接清空，当该表空间使用率告警时，要么对其进行扩容，要么新建 UNDO 表空间：

1.  增加 dbf 文件扩容

``` sql
ALTER TABLESPACE UNDOTBS ADD DATAFILE 'UNDOTBS01.dbf' SIZE 2048M;        --UNDOTBS为表空间名
```

2.  重新建立一个新的 undo 表空间

``` sql
--表空间创建
create undo tablespace undotbs2 datafile '/user/oracle/oradata/undotbs02.dbf' size 8048m;
```

``` sql
--设置数据库的undo表空间为新的undotbs2表空间
alter system set undo_tablespace=undotbs2;
```

``` sql
--删除旧的undo表空间及其内容
drop tablespace undotbs1 including contents;
```

``` bash
# 在服务器上删除undotbs1对应的文件undotbs01.dbf
rm undotbs01.dbf
```

参考：<https://blog.csdn.net/ziele_008/article/details/104792752>

# 7 时间类型 date 和 timestamp

------------------------------------------------------------------------

参考链接：<https://www.cnblogs.com/java-class/p/4742740.html>

## 7.1 date 类型

------------------------------------------------------------------------

<figure>
<img
src="https://lbwyjll-public.oss-cn-hangzhou.aliyuncs.com/img/20210425195433.gif"
alt="表情包浏览" />
<figcaption aria-hidden="true">表情包浏览</figcaption>
</figure>

# 8 Oracle-client 安装

------------------------------------------------------------------------

官网下载地址：<http://www.oracle.com/technetwork/database/features/instant-client/index-097480.html>

选择对应版本及客户端安装包：

- instantclient-basic-linux.x64-11.2.0.4.0.zip

<figure>
<img
src="https://lbwyjll-public.oss-cn-hangzhou.aliyuncs.com/img/20210425195232.png"
alt="image-20210425195221804" />
<figcaption aria-hidden="true">image-20210425195221804</figcaption>
</figure>

- instantclient-sqlplus-linux.x64-11.2.0.4.0.zip

<figure>
<img
src="https://lbwyjll-public.oss-cn-hangzhou.aliyuncs.com/img/20210425200740.png"
alt="image-20210425200736200" />
<figcaption aria-hidden="true">image-20210425200736200</figcaption>
</figure>

## 8.1 解压缩安装

------------------------------------------------------------------------

- 安装解压缩到 ==/usr/local/oracle== 自定义

``` bash
useradd oracle && echo "123456" | passwd --stdin oracle
mkdir -p /usr/local/oracle_client
mv ./instantclient* /usr/local/oracle_client
cd /usr/local/oracle_client
unzip instantclient-basic-linux.x64-11.2.0.4.0.zip
unzip instantclient-sqlplus-linux.x64-11.2.0.4.0.zip
cd /usr/local/oracle/instantclient_11_2
chown -R oracle:oracle /usr/local/oracle_client
```

<figure>
<img
src="https://lbwyjll-public.oss-cn-hangzhou.aliyuncs.com/img/20210425202618.png"
alt="image-20210425202616447" />
<figcaption aria-hidden="true">image-20210425202616447</figcaption>
</figure>

## 8.2 添加环境变量

------------------------------------------------------------------------

- `vim /home/oracle/.bash_profile`

``` bash
export ORACLE_HOME=/usr/local/oracle_client/instantclient_11_2      #ORACLE_HOME
export NLS_LANG=AMERICAN_AMERICA.AL32UTF8                           #客户端编码,与服务端一致
export LD_LIBRARY_PATH=$ORACLE_HOME                                 #动态库文件路径
export PATH=$ORACLE_HOME:$PATH                                      #添加PATH变量
source /home/oracle/.bash_profile
```

## 8.3 TNS 解析

------------------------------------------------------------------------

创建相关路径及 ==tnsnames.ora== 文件：

- `mkdir -p $ORACLE_HOME/network/admin`
- `vim $ORACLE_HOME/network/admin/tnsnames.ora`

``` bash
orcl =
  (DESCRIPTION =
    (ADDRESS = (PROTOCOL = TCP)(HOST = 192.168.137.129)(PORT = 1521))
    (CONNECT_DATA =
      (SERVER = DEDICATED)
      (SERVICE_NAME = orcl)
    )
  )
```

- 环境变量中添加：`export TNS_ADMIN=$ORACLE_HOME/network/admin`

<figure>
<img
src="https://lbwyjll-public.oss-cn-hangzhou.aliyuncs.com/img/20210425204130.png"
alt="image-20210425204128921" />
<figcaption aria-hidden="true">image-20210425204128921</figcaption>
</figure>

- 重新读取环境变量文件：`source /home/oracle/.bash_profile`
- 测试连接：`sqlplus USERNAME/PASSWD@IP:PORT/orcl`

# 9 sqlldr 使用

------------------------------------------------------------------------

## 9.1 sqlldr 安装（oracle 客户端）

------------------------------------------------------------------------

sqlldr 命令在 oracle 服务端版本安装后该命令已生成，客户端版本可参考官网说明下载安装，此处针对 11.2 版本，通过已有完整版安装至客户端做安装说明

1.  将完整版安装下的 sqlldr 二进制文件复制至客户端

    服务端：`$ORACLE_HOME/bin/sqlldr`

    客户端：`$ORACLE_HOME/sqlldr`

2.  在客户端 `$ORACLE_HOME` 下创建目录 `rdbms/mesg`

    `mkdir -p $ORACLE_HOME/rdbms/mesg`

3.  将完整版安装下的 `ulus.msb` 文件复制至客户端

    服务端：`$ORACLE_HOME/rdbms/mesg/ulus.msb`

    客户端：`$ORACLE_HOME/rdbms/mesg/ulus.msb`

4.  测试

<figure>
<img
src="https://lbwyjll-public.oss-cn-hangzhou.aliyuncs.com/img/20210708104452.png"
alt="image-20210708104434382" />
<figcaption aria-hidden="true">image-20210708104434382</figcaption>
</figure>

## 9.2 sqlldr 部分参数说明

------------------------------------------------------------------------

参考链接：<https://blog.csdn.net/weixin_33937499/article/details/93112648>

 <https://www.cnblogs.com/JIKes/p/12709664.html>

**`Usage: SQLLDR keyword=value [,keyword=value,…]`**

> 必需要有一个 数据文件和一个控制文件，日志文件可选（不指定的话，会自动生成的），数据文件的分隔符需要指定合适，否则会出现混乱，控制文件是告诉 sqlldr 数据文件和表的对应关系以及一些处理规则

``` bash
  userid        --  表示Oracle的 username/password[@servicename]
  control       --  表示控制文件，可能包含表的数据、处理规则等.ctl                     
  log           --  表示记录导入时的日志文件，默认为 控制文件(去除扩展名).log，默认自动生成       
  bad           --  表示坏数据文件，默认为 控制文件(去除扩展名).bad             
  data          --  表示数据文件，data参数只能指定一个数据文件，如果控制文件也通过infile指定了数据文件,并且指定多个，则                     sqlldr在执行时，先加载data参数指定的数据文件，控制文件中第一个infile指定的数据文件被忽略，但后续的                    infile指定的数据文件继续有效
  discard       --  丢弃的数据文件,默认情况不产生
  discardmax    --  允许丢弃数据的最大值
  errors        --  表示允许的错误记录数，表示出错N次后，停止加载                 
  rows          --  表示多少条记录提交一次，默认为 64                      
  skip          --  表示跳过的行数，比如导出的数据文件前面几行是表头或其他描述           
  load          --  表示并不导入所有的数据，只导入跳过skip参数后的N条数据           
  bindsize      --  表示每次提交记录缓冲区的大小，默认256k，单位字节                   
  readszie      --  缓冲区大小,默认值:1048576单位字节,最大不超过20m,该参数仅当从数据文件读取时有效，如果是从近制文件读取                    数 据，则默认为64k                                    
  silent        --  表示静默执行，即禁止输出信息                                   
  direct        --  使用直通路径方式导入,不走buffer cache,通过direct path api发送数据到服务器端的加载引擎，加载引擎按                    照数据块的格式处理数据并直接写向数据文件,因此效率较高(默认FALSE)
```

## 9.3 sqlldr 控制文件参数说明

------------------------------------------------------------------------

参考：<https://www.cnblogs.com/lirenhe/p/9774447.html>

 <https://www.cnblogs.com/xiaohuilong/p/5541571.html>

    OPTIONS (skip=1,rows=128)       --  sqlldr命令显示的选项可以写到这里边来,skip=1用来跳过数据中的第一行  
    LOAD DATA                       --  控制文件标识
    INFILE  "FILE"                  --  指定外部数据文件，可以写多个 INFILE "another_data_file.csv" 指定多个数据文                                     件，"*"时以sqlldr命令行选项为主
    BADFILE  "FILE"                 --  坏数据文件
    DISCARDFILE                     --  丢弃数据的文件
    TRUNCATE                        --  操作类型，用 truncate table来清除表中原有记录；
                                        INSERT--为缺省方式，在数据装载开始时要求表为空；
                                        APPEND--在表中追加新记录；
                                        REPLACE--删除旧记录(用 delete from table 语句)，替换成新装载的记录
    INTO TABLE users                --  要插入记录的表  
    Fields terminated by ","        --  数据中每行记录用 "," 分隔  
    Optionally enclosed by '"'      --  数据中每个字段用 '"' 框起，比如字段中有","分隔符时，界定范围  
    trailing nullcols               --  表的字段没有对应的值时允许为空  
    (  
      virtual_column FILLER,        --这是一个虚拟字段，用来跳过由 PL/SQL Developer 生成的第一列序号  
      user_id number,               --字段可以指定类型，否则认为是 CHARACTER 类型, log文件中有显示  
      user_name,  
      login_times,  
      last_login DATE "YYYY-MM-DD HH24:MI:SS" -- 指定接受日期的格式，相当用to_date()函数转换 
    )

## 9.4 示例脚本

------------------------------------------------------------------------

``` bash
#!/bin/bash
connect_info="USER/PASSWORD@SERVICE_NAME"

WORK_DIR="/usr/local/working/"
XDR_DIR="${WORK_DIR}xdr_load/"
BACKUP_DIR="${WORK_DIR}backup/"
ERROE_DIR="${WORK_DIR}error/"
PID=$$

ctl_file="${WORK_DIR}load_xdr.ctl"
log_file="${WORK_DIR}load_records.log"
bad_file="${WORK_DIR}load_bad.bad"


[[ ! -d ${WORK_DIR} ]] && echo "Error, WORK_DIR:${WORK_DIR} is not exist!" && exit 1
[[ ! -d ${XDR_DIR} ]] && echo "Error, XDR_DIR:${XDR_DIR} is not exist!" && exit 1
[[ ! -d ${BACKUP_DIR} ]] && mkdir -p ${BACKUP_DIR}
[[ ! -d ${ERROE_DIR} ]] && mkdir -p ${ERROE_DIR}
[[ -f ${ctl_file} ]] && rm -f ${ctl_file}
cat > ${ctl_file} << EOF
load data            
infile *
append
into table boss_request_history
FIELDS TERMINATED BY '|'
TRAILING NULLCOLS
(
    field1,
    field2,
    field3,
    field4,
    field5,
    ...
)
EOF

sqlldr userid="${connect_info}" control="${ctl_file}" data="${tempfile}" log="${log_file}" bad="${bad_file}" silent="header" rows=50 ERRORS=50
```

# 10 数据导出、导入

------------------------------------------------------------------------

> oracle 的 exp/imp 命令用于实现对数据库的导出/导入操作
>
> EXP 和 IMP 是客户端工具程序，它们既可以在客户端使用，也可以在服务端使用

> EXPDP 和 IMPDP 是服务端的工具程序，他们只能在 ORACLE 服务端使用，不能在客户端使用
> IMP 只适用于 EXP 导出的文件，不适用于 EXPDP 导出文件；IMPDP 只适用于 EXPDP 导出的文件，而不适用于 EXP 导出文件

## 10.1 EXP/IMP 命令安装（oracle 客户端）

------------------------------------------------------------------------

exp/imp 命令在 oracle 服务端版本安装后该命令已生成，客户端版本可参考官网说明下载安装，此处针对 11.2 版本，通过已有完整版安装至客户端做安装说明：

1.  将完整版安装下的==exp/imp==二进制文件复制至客户端

    服务端：`$ORACLE_HOME/bin/exp` ，\``$ORACLE_HOME/bin/imp`

    客户端：`$ORACLE_HOME/exp` ，`$ORACLE_HOME/imp`

2.  在客户端 `$ORACLE_HOME` 下创建目录 `rdbms/mesg`

    `mkdir -p $ORACLE_HOME/rdbms/mesg`

3.  将完整版安装下的 `ulus.msb` 文件复制至客户端

    服务端：`$ORACLE_HOME/rdbms/mesg/expus.msb`，`$ORACLE_HOME/rdbms/mesg/impus.msb`

    客户端：`$ORACLE_HOME/rdbms/mesg/expus.msb` ，`$ORACLE_HOME/rdbms/mesg/impus.msb`

4.  测试

<figure>
<img
src="https://lbwyjll-public.oss-cn-hangzhou.aliyuncs.com/img/20220224160309.png"
alt="image-20220224160245942" />
<figcaption aria-hidden="true">image-20220224160245942</figcaption>
</figure>

## 10.2 EXP 数据导出

------------------------------------------------------------------------

> exp 命令用于把数据从远程数据库 server 导出至本地，生成 dmp 文件

``` bash
[root@localhost ~]$ exp help=y
Export: Release 11.2.0.4.0 - Production on 星期一 11月 22 09:29:37 2021
Copyright (c) 1982, 2011, Oracle and/or its affiliates.  All rights reserved.

通过输入 EXP 命令和您的用户名/口令, 导出
操作将提示您输入参数: 
     例如: EXP SCOTT/TIGER
或者, 您也可以通过输入跟有各种参数的 EXP 命令来控制导出的运行方式。要指定参数, 您可以使用关键字: 
     格式:  EXP KEYWORD=value 或 KEYWORD=(value1,value2,...,valueN)
     例如: EXP SCOTT/TIGER GRANTS=Y TABLES=(EMP,DEPT,MGR)
               或 TABLES=(T1:P1,T1:P2), 如果 T1 是分区表USERID 必须是命令行中的第一个参数。
```

| 关键字（高亮为常用） | 说明 (默认值) |
|----|----|
| USERID | `USER/PASSWORD[@IP:PORT/SERVICE_NAME as ROLE]`，用户名、口令等登录信息 |
| FULL | 导出整个文件 (N)，全库导出时使用，可导出整个数据库的结构，FULL=N |
| BUFFER | 数据缓冲区大小，BUFFER=10000000，单位字节 |
| ==OWNER== | 所有者用户名列表 |
| ==FILE== | 输出文件 (EXPDAT.DMP)，FILE=/PATH/TO/SOME.DMP |
| ==TABLES== | 表名列表，TABLES=(table1，table2，...) |
| ==COMPRESS== | 导出时进行压缩 (Y)，建议使用，COMPRESS=Y |
| RECORDLENGTH | IO 记录的长度 |
| ==GRANTS== | 导出权限 (Y)，GRANTS=Y |
| INCTYPE | 增量导出类型 |
| INDEXES | 导出索引 (Y) |
| RECORD | 跟踪增量导出 (Y) |
| DIRECT | 直接路径 (N)，启用直接路径导出，这样可以减少 undo 日志的生成，加快速度 |
| TRIGGERS | 导出触发器 (Y) |
| ==LOG== | 屏幕输出的日志文件 ，LOG=/PATH/TO/SOMEFILE |
| STATISTICS | 分析对象 (ESTIMATE) |
| ==ROWS== | 导出数据行 (Y) ，ROWS=Y，默认导出表结构和数据；ROWS=N 只导出表结构，不导出数据 |
| PARFILE | 参数文件名 |
| CONSISTENT | 交叉表的一致性 (N) ，CONSISTENT=N |
| CONSTRAINTS | 导出的约束条件 (Y)，CONSTRAINTS=Y |
| OBJECT_CONSISTENT | 只在对象导出期间设置为只读的事务处理 (N)，OBJECT_CONSISTENT=N |
| FEEDBACK | 每 x 行显示进度 (0) |
| ==FILESIZE== | 每个转储文件的最大大小 |
| FLASHBACK_SCN | 用于将会话快照设置回以前状态的 SCN |
| ==QUERY== | 用于导出表的子集的 select 子句，或 where 子句 |
| RESUMABLE | 遇到与空格相关的错误时挂起 (N)，RESUMABLE=N |
| RESUMABLE_NAME | 用于标识可恢复语句的文本字符串 |
| RESUMABLE_TIMEOUT | RESUMABLE 的等待时间 |
| TTS_FULL_CHECK | 对 TTS 执行完整或部分相关性检查 |
| VOLSIZE | 写入每个磁带卷的字节数 |
| ==TABLESPACES== | 要导出的表空间列表 |
| TRANSPORT_TABLESPACE | 导出可传输的表空间元数据 (N) |
| TEMPLATE | 调用 iAS 模式导出的模板名 |

示例：

- 将数据库 TEST==完全导出==，用户名 system 密码 manager，实例名 TEST 导出到/tmp/test.dmp 中

``` bash
exp system/manager@TEST file=/tmp/test.dmp full=y
```

- 将数据库中 system 用户与 sys 用户的表导出

``` bash
exp system/manager@TEST file=/tmp/test.dmp owner=(system,sys)
```

- 将数据库中的表 table1 、table2 导出

``` bash
exp system/manager@TEST file=/tmp/test.dmp tables=(table1,table2) 
```

- 将数据库中的表 table1 中的字段 filed1 以 "00" 打头的数据导出

<!-- -->

    exp system/manager@TEST file=/tmp/test.dmp tables=(table1) query="where filed1 like '00%'"

- 将数据库中的表 table1 中指定时间段的数据导出（==格式存疑==）

``` bash
exp system/manager@TEST file=/tmp/test.dmp tables=(table1) grants=n query=\"where time\>= to_date'20211221000000' and time\<'20211222000000'\"
```

> 注意：如果需要使用到日期字符串格式等单引号，需要使用双引号将 where 条件括起来，而且双引号要用`\做转义`{=tex}

- 将数据库中的表 table1 中指定时间分区的数据导出，字段为 timestamp 格式

``` bash
exp system/manager@TEST file=/tmp/test.dmp tables=table1 compress=y grants=n query=\"where time \>= to_date\(\'2021-12-21 00:00:00\', \'yyyy-mm-dd hh24:mi:ss\'\) and time \< to_date\(\'2021-12-22 00:00:00\', \'yyyy-mm-dd hh24:mi:ss\'\)\"
```

> 注意：query 条件中，双引号、单引号、\>、\<、（、）均需要进行转义

- 将数据库中的表 table1 中指定分区的数据导出

``` bash
exp system/manager@TEST file=/tmp/test.dmp tables=table1:P20211221  grants=n compress=y
```

## 10.3 IMP 数据导入

------------------------------------------------------------------------

> imp 命令用于把本地的数据库 dmp 文件从本地导入到远程的 Oracle 数据库中。

``` bash
[root@loaclhost ~]$ imp help=y
Import: Release 11.2.0.4.0 - Production on 星期一 11月 22 09:25:20 2021
Copyright (c) 1982, 2011, Oracle and/or its affiliates.  All rights reserved.

通过输入 IMP 命令和您的用户名/口令, 导入
操作将提示您输入参数: 
     例如: IMP SCOTT/TIGER
或者, 可以通过输入 IMP 命令和各种参数来控制导入的运行方式。要指定参数, 您可以使用关键字: 
     格式:  IMP KEYWORD=value 或 KEYWORD=(value1,value2,...,valueN)
     例如: IMP SCOTT/TIGER IGNORE=Y TABLES=(EMP,DEPT) FULL=N
               或 TABLES=(T1:P1,T1:P2), 如果 T1 是分区表
USERID 必须是命令行中的第一个参数
```

| 关键字（高亮为常用） | 说明 (默认值) |
|----|----|
| USERID | `USER/PASSWORD[@IP:PORT/SERVICE_NAME as ROLE]`，用户名、口令等登录信息 |
| FULL | 导入整个文件 的内容 (N) |
| BUFFER | 数据缓冲区大小，BUFFER=10000000，单位字节 |
| ==FROMUSER== | 所有者用户名列表 |
| ==FILE== | 输入文件 (EXPDAT.DMP)，FILE=/PATH/TO/SOME.DMP |
| ==TOUSER== | 用户名列表 |
| SHOW | 只列出文件内容 (N) |
| ==TABLES== | 表名列表，TABLES=(table1，table2，...) |
| ==IGNORE== | 忽略创建错误 (N)，IGNORE=N，导入时表已存在会导入失败，此时可用 |
| RECORDLENGTH | IO 记录的长度 |
| GRANTS | 导入权限 (Y) |
| ==INCTYPE== | 增量导入类型 |
| INDEXES | 导入索引 (Y) |
| COMMIT | 提交数组插入 (N) |
| ROWS | 导入数据行 (Y)，ROWS=Y |
| PARFILE | 参数文件名 |
| ==LOG== | 屏幕输出的日志文件 |
| CONSTRAINTS | 导入限制 (Y) |
| ==DESTROY== | 覆盖表空间数据文件 (N) |
| INDEXFILE | 将表/索引信息写入指定的文件 |
| SKIP_UNUSABLE_INDEXES | 跳过不可用索引的维护 (N) |
| FEEDBACK | 每 x 行显示进度 (0) |
| TOID_NOVALIDATE | 跳过指定类型 ID 的验证 |
| FILESIZE | 每个转储文件的最大大小 |
| STATISTICS | 始终导入预计算的统计信息 |
| RESUMABLE | 在遇到有关空间的错误时挂起 (N) |
| RESUMABLE_NAME | 用来标识可恢复语句的文本字符串 |
| RESUMABLE_TIMEOUT | RESUMABLE 的等待时间 |
| COMPILE | 编译过程, 程序包和函数 (Y) |
| STREAMS_CONFIGURATION | 导入流的一般元数据 (Y） |
| STREAMS_INSTANTIATION | 导入流实例化元数据 (N) |
| DATA_ONLY | 仅导入数据 (N) |
| VOLSIZE | 磁带的每个文件卷上的文件的字节数 |

下列关键字仅用于可传输的表空间

| 关键字               | 说明 (默认值)                  |
|----------------------|--------------------------------|
| TRANSPORT_TABLESPACE | 导入可传输的表空间元数据 (N)   |
| TABLESPACES          | 将要传输到数据库的表空间       |
| DATAFILES            | 将要传输到数据库的数据文件     |
| TTS_OWNERS           | 拥有可传输表空间集中数据的用户 |

示例：

``` bash
imp system/manager@TEST file=/tmp/test.dmp full=y ignore=y;
```

==注意==：

1.  表导入的过程：创建表，导入数据，创建序列
2.  导入时，不包含存储信息导入的时候就会默认导入到导入用户的默认表空间
3.  含存储信息导入的时候，如==用户不存在==、==表空间不存在==等也会导致部分表导入失败

==解决方案==：

方案一：

1.  会话窗口 1 用 system 用户登录，查找导入的目标用户的默认表空间

`select username, default_tablespace from dba_users where username='USERNAME';`

2.  执行修改表空间语句（假设目标数据库的表空间名是：xxx_tablespace）

`alter tablespace xxx_tablespace rename to xxx;`

3.  会话窗口 2 执行 imp 语句
4.  导入成功后，会话窗口 1 执行改回原来表空间的名称

`alter tablespace xxx rename to xxx_tablespace;`

方案二：

根据 log 信息重新创建对应的表，然后再执行 imp 语句（注意：要加上 ignore=y）

## 10.4 EXPDP 数据导出

------------------------------------------------------------------------

> EXPDP 是数据泵导出的工具，它可以把数据库中的对象选择性的导出到操作系统中。比如：表、用户、表空间、数据库等。

> 使用 EXPDP 工具与 EXP 不同的是，在使用 EXPDP 时要先创建目录对象，通过这个对象就可以找到要备份数据的数据库服务器，并且使 EXPDP 工具备份出来的数据必须存放在目录对象对应的操作系统的目录中。

``` bash
[oracle@localhost ~]$ expdp help=y
Export: Release 11.2.0.4.0 - Production on 星期四 2月 24 10:25:07 2022
Copyright (c) 1982, 2011, Oracle and/or its affiliates.  All rights reserved.

数据泵导出实用程序提供了一种用于在 Oracle 数据库之间传输
数据对象的机制。该实用程序可以使用以下命令进行调用:
   示例: expdp scott/tiger DIRECTORY=dmpdir DUMPFILE=scott.dmp
您可以控制导出的运行方式。具体方法是: 在 'expdp' 命令后输入 
各种参数。要指定各参数, 请使用关键字:
   格式:  expdp KEYWORD=value 或 KEYWORD=(value1,value2,...,valueN)
   示例: expdp scott/tiger DUMPFILE=scott.dmp DIRECTORY=dmpdir SCHEMAS=scott
               或 TABLES=(T1:P1,T1:P2), 如果 T1 是分区表
USERID 必须是命令行中的第一个参数。
```

| 关键字（高亮为常用） | 说明（默认值） |
|----|----|
| USERID | `USER/PASSWORD[@IP:PORT/SERVICE_NAME as ROLE]`，用户名、口令等登录信息 |
| ATTACH | 连接到现有作业，`ATTACH [=[schema_name.]job_name]`，schema_name 表示用户名，job_name 表示导出的作业名。==注意：如果使用 ATTACH 选项，在命令行除了连接字符串和 ATTACH 选项外，不能指定任何其他选项==。可以通过查询 DBA_DATAPUMP_JOBS 获得系统中现有的作业信息 |
| ==COMPRESSION== | 是否压缩数据库对象数据，以减少转储文件大小，有效的关键字值为: `COMPRESSION={ALL | DATA_ONLY | METADATA_ONLY | NONE}`，默认为 METADATA_ONLY |
| ==CONTENT== | `CONTENT={ALL | DATA_ONLY | METADATA_ONLY}`，当设置 CONTENT 为 ALL 时，会导出对象元数据及对象数据；当设置为 DATA_ONLY 时，只导出对象数据；当设置为 METADATA_ONLY 时，只导出对象元数据 |
| ==DIRECTORY== | 指定转储文件和日志文件所在的目录，给定的参数是一个 DIRECTORY 数据库对象（一个逻辑对象，并不会在系统上创建物理路径），是通过 CREATE DIRECTORY 语句建立的。`DIRECTORY=directory_object` |
| ==DUMPFILE== | 用于指定转储文件的名称，默认名称为 expdat.dmp；`DUMPFILE=[directory_object:]file_name [, …]`；directory_object 用于指定目录对象名，file_name 用于指定转储文件名。如果不给定 directory_object，导出工具会自动使用 DIRECTORY 选项指定的目录对象。这个参数==可以结合 FILESIZE 参数==一起使用，达到生成多个转储文件的目的。==注意：如果指定路径下已经存在待生成的导出文件，导出过程中将会报错退出==。 |
| ENCRYPTION_PASSWORD | 该参数需要和 Oracle 的透明数据加密特性（TDE）一同使用，因为 expdp 本身是不支持加解密的。`ENCRYPTION_PASSWORD = password` |
| ==EXCLUDE== | 用于控制在导出过程中哪些数据库对象不被导出。`EXCLUDE=[object_type]:[name_clause] [, …]`，object_type 用于指定要排除的对象类型，如 table、sequence、view、procedure、package 等；name_clause 用于指定要排除的具体对象名称，如。==注意：EXCLUDE 选项和 INCLUDE 选项不能同时使用==。该选项支持模糊匹配，非常好用的功能。另外，被指定不被导出的表上的约束、索引、触发器等均不会被导出。 |
| ==FILESIZE== | 限定单个转储文件的最大容量，默认值是 0，表示没有文件尺寸的限制。该选项与 DUMPFILE 选项一同使用。`FILESIZE=NUMBER [B | K | M | G]` |
| ==FULL== | 是否以全库模式导出数据库。默认为 N。`FULL={y | n}`，为 Y 时，表示执行数据库的全库导出。 |
| ==INCLUDE== | 指定导出哪些数据库对象类型或数据库对象。与 EXCLUDE 选项用法相同，功能相反。==注意：INCLUDE 选项和 EXCLUDE 选项不能同时使用==。`INCLUDE = [object_type]:[name_clause] [, …]` |
| JOB_NAME | 指定要导出作业的名称。默认为 SYS_EXPORT\_\[mode\]\_\[nn\]，`JOB_NAME=jobname_string`，对应的作业信息可以通过 DBA_DATAPUMP_JOBS 视图获得。 |
| ==LOGFILE== | 指定导出过程中日志文件的名称，默认值为 export.log。`LOGFILE=[directory_object:]file_name`，directory_object 指定目录对象的名称，file_name 用于指定导出日志文件的名称。如果不指定 directory_object，会自动使用 DIRECTORY 选项的值。 |
| NOLOGFILE | 控制是否禁止生成导出日志文件，默认值为 N。如果设置为 Y，表示不输出日志。`NOLOGFILE={y | n}` |
| PARALLEL | 指定执行导出操作的并行度，默认值为 1。`PARALLEL=INT_NUMBER`，==注意：这个参数给出的并行度是一个真正能启用进程数的最大值==。具体会启用多少个进程并行处理会受很多因素影响，例如生成转储文件的多少（不能多于文件数）、导出的数据量大小、CPU 资源还有系统 I/O 资源等因素影响。 |
| ==QUERY== | 用来指定类似 where 语句限定导出的记录。相比 exp 命令的 QUERY 选项，这里更加的灵活，可以同时针对每张表进行条件限制。`QUERY = [schema.\][table_name:] query_clause`，因为该参数目的是限制导出数据的多少，因此不能和 CONTENT=METADATA_ONLY、ESTIMATE_ONLY 还有 TRANSPORT_TABLESPACES 一起使用。 |
| SAMPLE | 给出导出表数据的百分比，参数值可以取.000001\~100（不包括 100）。不过导出过程不会和这里给出的百分比一样精确，是一个近似值。语法如下：`SAMPLE=[[schema_name.]table_name:]sample_percent`，示例：SAMPLE="HR"."EMPLOYEES":50 |
| ==SCHEMAS== | 按照 SCHEMA 模式导出，默认为当前用户，可简单理解为导出指定用户的相关数据，`SCHEMAS=schema_name [, …]` |
| ==TABLES== | 以表模式导出数据。可以同时导出多个表；支持通配符格式的导出；也支持只导出分区表中的某个分区。`TABLES=[schema_name.]table_name[:partition_name] [, …]`，schema_name 用于指定用户名，table_name 用于指定导出的表名，partition_name 用于指定要导出的分区名。 |
| ==TABLESPACES== | 指定需要导出哪个表空间中的表数据。==注意：只有表空间里的表数据会被导出==。`TABLESPACES=tablespace_name [, …]` |

==注意：执行导出操作时需要事先估算导出对象的大小，以避免磁盘空间不够而导出失败==

| 关键字（高亮为常用） | 说明（默认值） |
|----|----|
| ESTIMATE_ONLY | 是否仅作评估不会导出数据，`ESTIMATE_ONLY={ Y | N }` |
| ESTIMATE | 大小评估方式，`ESTIMATE={ blocks | statistics }`，设置为 estimate=bloks，Oracle 会按照目标对象所占用的数据块个数乘以数据块尺寸估算对象占用的空间；设置为 estimate=statistics，Oracle 根据最近统计值估算对象占用空间；限制：当数据库包含压缩表时，estimate=blocks 得出的结果会不准确，此时建议使用 estimate=statistics |

==注意：使用 expdp 工具时,其转储文件只能被存放在 DIRECTORY 对象对应的 OS 目录中，而不能直接指定转储文件所在的 OS 目录.因此，使用 expdp 工具时，必须首先建立 DIRECTORY 对象，并且需要为数据库用户授予使用 DIRECTORY 对象权限.==

==首先需要创建逻辑目录==，最好以 system 等管理员创建逻辑目录：

- 查看已有 DIRECTORY 对象及其 OS 上的路径

``` plsql
select * from dba_directories
```

- 创建或替换 DIRECTORY 对象，指定其 OS 上的路径

``` plsql
create or replace directory DIRECTORY_NAME as '/PATH/TO/SOMEWHERE';
```

- 授权数据库用户使用 DIRECTORY 对象权限

``` plsql
grant read,write on directory DIRECTORY_NAME to USER_NAME;
```

- 查看用户拥有的对象权限

``` plsql
SELECT privilege, directory_name, DIRECTORY_PATH
  FROM user_tab_privs t, all_directories d
 WHERE t.table_name(+) = d.directory_name
 ORDER BY 2, 1;
```

示例：假设 directory 对象为 DATA_BACKUP，其对应的 OS 目录为/tmp

- ==估算 test 用户的导出对象的大小，不执行导出操作==

``` bash
expdp test/123456@test schemas=scott estimate_only=y estimate=statistics
```

``` bash
expdp test/123456@test schemas=scott estimate_only=y estimate=blocks
```

- 导出指定表 test1

``` bash
expdp test/123456@test directory=DATA_BACKUP dumpfile=expdp.dmp tables=test1,test2 logfile=expdp.log;
```

- 导出 scott 用户及其对象数据

``` bash
expdp test/123456@test directory=DATA_BACKUP dumpfile=expdp.dmp schemas=scott logfile=expdp.log;
```

- 导出指定表 test1 数据（带条件）

``` bash
expdp test/123456@test directory=DATA_BACKUP dumpfile=expdp.dmp tables=test1 query='where deptno=20' logfile=expdp.log;
```

- 导出指定表空间数据

``` bash
expdp test/123456@test directory=DATA_BACKUP dumpfile=expdp.dmp tablespaces=temp,example logfile=expdp.log;
```

- 导出整个库的数据

``` bash
expdp test/123456@test directory=DATA_BACKUP dumpfile=expdp.dmp full=y logfile=expdp.log;
```

- 导出整个库的元数据信息

``` bash
expdp test/123456@test directory=DATA_BACKUP dumpfile=expdp.dmp content=metadata_only;
```

- 排除指定对象后，导出剩余数据

``` bash
expdp test/123456@test directory=DATA_BACKUP dumpfile=expdp.dmp EXCLUDE=VIEW;
```

- 指定转储文件大小，生成多个转储文件

``` bash
expdp test/123456@test directory=DATA_BACKUP dumpfile=expdp.dmp1,expdp.dmp2,expdp.dmp3 schemas=scott filesize=1K
```

==注意==：

- 使用 filesize 参数时，若 dumpfile 参数只给定了一个值，则导出数据大小为 filesize 参数限制的大小后就不会继续导出，所以在使用此参数时，要注意 dumpfile 参数指定的转储文件个数
- ==可在 dumpfile 参数值中使用 "%U" 变量，这个变量会从 01 开始增长，以此自动扩展==

``` bash
expdp test/123456@orcl directory=DATA_BACKUP dumpfile=expdp%U.dmp schemas=scott filesize=1K
```

- 上面语句会在 directory 对象下，生成 expdp01.dmp、expdp02.dmp、expdp03.dmp 形式的文件

==EXCLUDE 和 INCLUDE 的使用：==

- `EXCLUDE=[object_type]:[name_clause],[object_type]:[name_clause]` --\>排出特定对象
- `INCLUDE=[object_type]:[name_clause],[object_type]:[name_clause]` --\>包含特定对象
- `object_type` 子句用于指定对象的类型，如 table、sequence、view、procedure、package 等等，`name_clause` 子句可以为 SQL 表达式用于过滤特定的对象名字。它由 SQL 操作符以及对象名 (可使用通配符) 来过滤指定对象类型中的特定对象
- 当未指定 `name_clause` 而仅仅指定 `object_type` 则所有该类型的对象都将被过滤或筛选。多个 `[object_type]:[name_clause]` 中间以逗号分割。

常用过滤表达：

``` plsql
EXCLUDE=SEQUENCE,VIEW                               --过滤所有的SEQUENCE,VIEW
EXCLUDE=TABLE:"IN ('EMP','DEPT')"                   --过滤表对象EMP,DEPT
EXCLUDE=SEQUENCE,VIEW,TABLE:"IN ('EMP','DEPT')"     --过滤所有的SEQUENCE,VIEW以及表对象EMP,DEPT
EXCLUDE=INDEX:"= 'INDX_NAME'"                       --过滤指定的索引对象INDX_NAME
INCLUDE=PROCEDURE:"LIKE 'PROC_U%'"                  --包含以PROC_U开头的所有存储过程(_ 符号代表任意单个字符)
INCLUDE=TABLE:"> 'E' "                              --包含大于字符E的所有表对象

    其它常用操作符 NOT IN, NOT LIKE, <, != 等等
```

参考链接：<https://blog.csdn.net/liqfyiyi/article/details/7248911>

## 10.5 IMPDP 数据导入

------------------------------------------------------------------------

> IMPDP 是数据泵导入的工具，它可以把指定的 dump 文件中的对象选择性的导入到操作系统中。比如：表、用户、表空间、数据库等。

``` bash
[oracle@localhost ~]$ impdp help=y
Import: Release 11.2.0.4.0 - Production on 星期四 2月 24 16:22:38 2022
Copyright (c) 1982, 2011, Oracle and/or its affiliates.  All rights reserved.

数据泵导入实用程序提供了一种用于在 Oracle 数据库之间传输
数据对象的机制。该实用程序可以使用以下命令进行调用:
     示例: impdp scott/tiger DIRECTORY=dmpdir DUMPFILE=scott.dmp
您可以控制导入的运行方式。具体方法是: 在 'impdp' 命令后输入
各种参数。要指定各参数, 请使用关键字:
     格式:  impdp KEYWORD=value 或 KEYWORD=(value1,value2,...,valueN)
     示例: impdp scott/tiger DIRECTORY=dmpdir DUMPFILE=scott.dmp
USERID 必须是命令行中的第一个参数。
```

| 关键字（高亮为常用） | 说明（默认值） |
|----|----|
| ATTACH | 连接到现有作业，例如：ATTACH=job_name，具体可参考 EXPDP 中说明 |
| ==CONTENT== | 指定要加载的数据。`CONTENT={ALL |DATA_ONLY |METADATA_ONLY}`，当设置 CONTENT 为 ALL 时，会导入对象元数据及对象数据；当设置为 DATA_ONLY 时，只导入对象数据；当设置为 METADATA_ONLY 时，只导入对象元数据 |
| ==DIRECTORY== | 指定转储文件和日志文件所在的目录，给定的参数是一个 DIRECTORY 数据库对象（一个逻辑对象，并不会在系统上创建物理路径），是通过 CREATE DIRECTORY 语句建立的。`DIRECTORY=directory_object` |
| ==DUMPFILE== | 用于指定要导入的转储文件名称列表，默认名称为 expdat.dmp；`DUMPFILE=[directory_object:]file_name [, …]`；directory_object 用于指定目录对象名，file_name 用于指定转储文件名。如果不给定 directory_object，导出工具会自动使用 DIRECTORY 选项指定的目录对象 |
| ==EXCLUDE== | 排除特定对象类型。例如：`EXCLUDE=SCHEMA:"='HR'"` |
| ==FULL== | 导入源中的所有对象（Y），`FULL={ Y | N }` |
| ==INCLUDE== | 包括特定对象类型。例如：`INCLUDE=TABLE_DATA` |
| ==LOGFILE== | 日志文件名 （import.log），`LOGFILE=FILE_NAME` |
| NOLOGFILE | 不写入日志文件 （N）。`NOLOGFILE={ Y | N }` |
| PARALLEL | 指定执行导入操作的并行度，默认为 1，`PARALLEL=INT_NUMBER`，例如：PARALLEL=4 |
| ==QUERY== | 用来指定类似 where 语句限定导入的记录。可以同时针对每张表进行条件限制。`QUERY = [schema.][table_name:] query_clause`，因为该参数目的是限制导出数据的多少，因此不能和 CONTENT=METADATA_ONLY、ESTIMATE_ONLY 还有 TRANSPORT_TABLESPACES 一起使用。例如：QUERY=employees:"WHERE department_id \> 10" |
| ==REMAP_DATAFILE== | 将源数据文件名转变为目标数据文件名，在不同平台之间搬移表空间时可能需要该选项：`REMAP_DATAFIEL=SOURCE_DATAFIE:TARGET_DATAFILE` |
| ==REMAP_SCHEMA== | 将源方案的所有对象装载到目标方案中，简单理解就是导入时将归属于一个用户的数据的从属关系修改为另一个用户：`REMAP_SCHEMA=SOURCE_SCHEMA:TARGET_SCHEMA` |
| ==REMAP_TABLESPACE== | 将源表空间的所有对象导入到目标表空间中；`REMAP_TABLESPACE=SOURCE_TABLESPACE:TARGET:TABLESPACE` |
| REUSE_DATAFILES | 建立表空间时是否覆盖已存在的数据文件（N）：`REUSE_DATAFILES={ Y | N }` |
| SKIP_UNUSABLE_INDEXES | 导入时，是否跳过不可使用的索引（N）：`SKIP_UNUSABLE_INDEXES={ Y | N }` |
| STATUS | 监视作业状态的频率，单位 s，默认为 0 表示只要有新状态可用, 就立即显示新状态 |
| ==TABLES== | 以表模式导入数据。可以同时导入多个表；支持通配符格式的导入；也支持只导入分区表中的某个分区。`TABLES=[schema_name.]table_name[:partition_name][, …]`，schema_name 用于指定用户名，table_name 用于指定导出的表名，partition_name 用于指定要导出的分区名。 |
| ==TABLESPACES== | 指定需要导入哪个表空间中的表数据。==注意：只有表空间里的表数据会被导出入==。`TABLESPACES=tablespace_name [, …]` |
| ==TABLE_EXISTS_ACTION== | 当表已经存在时，导入作业要执行的操作（SKIP）：`TABBLE_EXISTS_ACTION={SKIP | APPEND | TRUNCATE | REPLACE }`；当设置该选项为 SKIP 时，导入作业会跳过已存在表处理下一个对象；当设置为 APPEND 时，会追加数据，为 TRUNCATE 时，导入作业会截断表，然后为其追加新数据；当设置为 REPLACE 时，导入作业会删除已存在表，重建表并追加数据；==注意：TRUNCATE 选项不适用与簇表和 NETWORK_LINK 选项== |
| TRANSPORT_DATAFILES | 搬移空间时要被导入到目标数据库的数据文件：`TRANSPORT_DATAFILE=DATAFILE_NAME`，DATAFILE_NAME 用于指定被复制到目标数据库的数据文件 |
| PARALLEL | 指定执行导入操作的并行度，默认值为 1。`PARALLEL=INT_NUMBER`，==注意：这个参数给出的并行度是一个真正能启用进程数的最大值==。具体会启用多少个进程并行处理会受很多因素影响，例如生成转储文件的多少（不能多于文件数）、导出的数据量大小、CPU 资源还有系统 I/O 资源等因素影响。 |

注意：

- 当导入目标库时，目标库中无对应的用户、表空间，则会导入失败

示例：

- 导入用户 test 的数据

``` bash
impdp test/123456@test directory=DATA_BACKUP dumpfile=expdp.dmp schemas=test logfile=impdp.log
```

- 从 scott 用户的表 dept、emp 导入到 system 用户中，表已存在则替换

``` bash
impdp test/123456@test directory=DATA_BACKUP dumpfile=expdp.dmp tables=scott.dept,scott.emp remap_schema=scott:test logfile=impdp.log table_exists_action=replace 
```

- 导入指定表空间中的对象数据

``` bash
impdp test/123456@test directory=DATA_BACKUP dumpfile=expdp.dmp tablespaces=example logfile=impdp.log
```

- 导入整个库

``` bash
impdp test/123456@test directory=DATA_BACKUP dumpfile=expdp.dmp full=y logfile=impdp.log
```

- 追加数据

``` bash
impdp test/123456@test directory=DATA_BACKUP dumpfile=expdp.dmp schemas=system table_exists_action=append logfile=impdp.log
```

# 11 数据备份

------------------------------------------------------------------------

## 11.1 单表备份

------------------------------------------------------------------------

### 11.1.1 单表本地库备份

1.  备份表结构和数据

``` plsql
create table 备份表 as select * from 被备份表;
```

==注意==：

- 包含表的结构 \[列名和数据类型\] ，但所有列都为可空列；
- 包含表中的数据

2.  仅备份表结构

``` plsql
create table 备份表 as select * from 被备份表 where 1=2
```

### 11.1.2 单表异地库备份

## 11.2 多表备份

------------------------------------------------------------------------

## 11.3 全库备份

------------------------------------------------------------------------

### 11.3.1 全量备份

### 11.3.2 差异备份

### 11.3.3 增量备份

# 12 其他

------------------------------------------------------------------------

## 12.1 重复数据查询、删除

------------------------------------------------------------------------

1.  查找表中多余的重复记录

- 根据单个字段（Id）来判断

``` plsql
select *
  from 表
 where Id in (select Id from 表 group by Id having count(Id) > 1);
```

- 根据多个字段（Id、seq）来判断

``` plsql
select *
  from 表 a
 where (a.Id, a.seq) in
       (select Id, seq from 表 group by Id, seq having count(*) > 1);
```

2.  删除表中多余数据

- 根据单个字段（Id）来判断，只留有 rowid 最小的记录

``` plsql
delete from 表
 where (id) in (select id from 表 group by id having count(id) > 1)
   and rowid not in
       (select min(rowid) from 表 group by id having count(*) > 1);
```

- 根据多个字段（id、seq）来判断，只留有 rowid 最小的记录

``` plsql
delete from 表 a
 where (a.Id, a.seq) in
       (select Id, seq from 表 group by Id, seq having count(*) > 1)
   and rowid not in
       (select min(rowid) from 表 group by Id, seq having count(*) > 1);
```

参考：

<https://www.cnblogs.com/zll-wyf/p/14665395.html>


---

> 作者: [仰泳的鱼](http://localhost:1313)  
> URL: http://localhost:1313/documentation/2bc07a1c-250727-170600/  

