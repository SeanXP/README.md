配置Samba & 支持Windows访问
====

Samba是一个能让Linux系统应用Microsoft网络通讯协议的软件，而SMB是Server Message Block的缩写，即为服务器消息块 ，SMB主要是作为Microsoft的网络通讯协议，后来Samba将SMB通信协议应用到了Linux系统上，就形成了现在的Samba软件。后来微软又把 SMB 改名为 CIFS（Common Internet File System），即公共 Internet 文件系统，并且加入了许多新的功能，这样一来，使得Samba具有了更强大的功能。

Samba最大的功能就是可以用于Linux与windows系统直接的文件共享和打印共享，Samba既可以用于windows与Linux之间的文件共享，也可以用于Linux与Linux之间的资源共享，由于NFS(网络文件系统）可以很好的完成Linux与Linux之间的数据共享，因而 Samba较多的用在了Linux与windows之间的数据共享上面。

Samba 是 SMB/CIFS 网络协议的重新实现, 它作为 NFS 的补充使得在 Linux 和 Windows 系统中进行文件共享、打印机共享更容易实现。

[http://wiki.ubuntu.org.cn/Samba]

[https://wiki.archlinux.org/index.php/Samba]

[http://www.cnblogs.com/mchina/archive/2012/12/18/2816717.html]

##1. 安装Samba服务器

Ubuntu: 

	$ sudo apt-get install samba
	
Arch Linux:

	$ sudo pacman -S samba
	
##2. 创建共享目录
假设共享目录为: /home/Share, 设置该文件夹的权限777使其让所有用户可读可写可运行。

	$ sudo mkdir -p /home/Share
	$ sudo chmod 777 /home/Share
##3. 修改配置文件
	
备份并编辑smb.conf允许网络用户访问

	$ sudo cp /etc/samba/smb.conf /etc/samba/smb.conf_backup
	$ sudo vim /etc/samba/smb.conf

### 公共共享文件夹 (支持游客登录）	
/etc/samba/smb.conf文件末尾添加:

	[Public-Share]
		comment = shared folder for public
		path = /home/Share
		# 是否游客访问 
		# guest ok = yes # 等价 'public = yes'
		public = yes
		# 是否可写
		writable = yes
		# 是否可浏览
		browseable = yes
		# 是否可用
		available = yes
### 用户共享文件夹 (samba帐号登录）

	[Private-Share]
		comment = Shared Folder with username and password
		path = /home/Share
		# 指定能够使用该共享资源的用户和组
		valid users = user1
		# 是否可写
		writable = yes
		# 是否可浏览
		browseable = yes
		# 是否可用
		available = yes

##4. 创建samba用户(公共共享文件夹则无需创建帐号)

注意，创建samba用户之前，必须先确保有一个同名的Linux用户，否则samba用户会创建失败。

	$ sudo useradd user1
上面只是增加了user1这个用户，却没有给用户赋予本机登录密码。所以这个用户将只能从远程访问，不能从本机登录(当然，root可以修改user1用户的密码，这样user1就可以登录本机了)。

而且samba的登录密码可以和本机登录密码不一样。**smb用户密码与unix帐号密码可以一样或不同，它们之间不存在连接关系。**

现在要新增网络使用者的帐号，设置user1用户的samba密码，这个密码不是开机登录，而是用于samba登录：

	$ sudo smbpasswd -a user1

删除samba密码:
	
	$ sudo smbpasswd -x user1
	
同样，也可以使用pdbedit设置samba密码;

	$ sudo pdbedit -a -u user1
查看:

	$ sudo pdbedit -L
	
删除:

	$ sudo pdbedit -x user1

----

##5. 重启samba服务

	$ sudo service smbd restart
	$ sudo /etc/init.d/samba restart

##6. 客户端访问

###Linux - smbclient
	
	$ sudo apt-get install smbclient
	
匿名(-N) 查看(-L):
	
	$ smbclient -L 10.42.1.100 -N
实名(-U) 查看(-L):

	$ smbclient -L 10.42.1.100 -U user%password
	$ smbclient -L 10.42.1.100 -U user1 (推荐，避免保存明文密码）
这里的password是smbpasswd, 与linux登录密码不等价。

进入smbclient命令行进行交互:

	$ smbclient //10.42.1.100/Share -N
	$ smbclient //10.42.1.100/Share -U user1

###Linux - mount.cifs挂载

1. 安装cifs-utils组件

		$ sudo apt-get install cifs-utils

2. 挂载samba目录
		
		$ sudo mount -t cifs //10.42.1.100/Share /mnt -o guest
		$ sudo mount -t cifs //10.42.1.100/Share /mnt -o username=user1,password=xxxxxx
		$ sudo mount -t cifs //10.42.1.100/Share /mnt -o username=user1
		$ sudo mount.cifs //10.42.1.100/Share /mnt -o username=user1

----


## 配置文件详细配置

###[global]
	# 定义工作组, 建议修改为”WORKGROUP”（windows默认的工作组名字）
	workgroup = WORKGROUP
	# samba server名称
	server string = %h server (Samba, Ubuntu)
	dns proxy = no
	#定义samba的日志，这里的%m是上面的netbios name
	log file = /var/log/samba/log.%m
	max log size = 1000
	panic action = /usr/share/samba/panic-action %d
	# samba的安全等级。关于安全等级有四种：
	# 		share：用户不需要账户及密码即可登录samba服务器
	#		user：由提供服务的samba服务器负责检查账户及密码（默认）
	#		server：检查账户及密码的工作由另一台windows或samba服务器负责
	#		domain：指定windows域控制服务器来验证用户的账户及密码。
	security = user
	# passdb backend （用户后台），samba有三种用户后台：smbpasswd, tdbsam和ldapsam;
	#		smbpasswd：该方式是使用smb工具smbpasswd给系统用户（真实用户或者虚拟用户）设置一个Samba 密码；
	#				客户端就用此密码访问Samba资源。smbpasswd在/etc/samba中，有时需要手工创建该文件。
	#		tdbsam：使用数据库文件创建用户数据库。数据库文件叫passdb.tdb，在/etc/samba中。
	#				passdb.tdb用户数据库可使用smbpasswd -a创建Samba用户，要创建的Samba用户必须先是系统用户。
	#				也可使用pdbedit创建Samba账户。
	#		ldapsam：基于LDAP账户管理方式验证用户。首先要建立LDAP服务。
	passdb backend = tdbsam 
	 
	# 用来设置允许的主机，如果在前面加”;”则为注释，表示允许所有主机
	hosts allow = 127.  192.168.12.  192.168.13.

###其他配置

	
		# 指明新建立的文件的属性，一般是0755
		create mask = 0755
		# 指明新建立的目录的属性，一般是0755
		directory mask = 0755
		comment = smb share test # 该共享的备注
        path = /home/share # 共享路径
        allow hosts = host(subnet) # 设置该Samba服务器允许的工作组或者域
        deny hosts = host(subnet) # 设置该Samba服务器拒绝的工作组或者域
        available = yes|no # 设置该共享目录是否可用
        browseable = yes|no # 设置该共享目录是否可显示
        writable = yes|no # 指定了这个目录缺省是否可写，也可以用readonly = no来设置可写
        public = yes|no # 指明该共享资源是否能给游客帐号访问，guest ok = yes其实和public = yes是一样的
        user = user, @group # user设置所有可能使用该共享资源的用户，也可以用@group代表group这个组的所有成员，不同的项目之间用空格或者逗号隔开
        valid users = user, @group # 指定能够使用该共享资源的用户和组
        invalid users = user, @group # 指定不能够使用该共享资源的用户和组
        read list = user, @group # 指定只能读取该共享资源的用户和组
        write list = user, @group # 指定能读取和写该共享资源的用户和组
        admin list = user, @group # 指定能管理该共享资源（包括读写和权限赋予等）的用户和组
        hide dot files = yes|no # 指明是否像UNIX那样隐藏以“.”号开头的文件
        create mode = 0755 # 指明新建立的文件的属性，一般是0755
        directory mode = 0755 # 指明新建立的目录的属性，一般是0755
        sync always = yes|no # 指明对该共享资源进行写操作后是否进行同步操作
        short preserve case = yes|no # 指明是否区分文件名大小写
        preserve case = yes|no # 指明是否保持大小写
        case sensitive = yes|no # 指明是否对大小写敏感，一般选no，不然可能引起错误
        mangle case = yes|no # 指明混合大小写
        default case = upper|lower # 指明缺省的文件名是全部大写还是小写
        force user = testuser # 强制把建立文件的属主是谁。如果我有一个目录，让guest可以写，那么guest就可以删除，如果我用force user= testuser强制建立文件的属主是testuser，同时限制create mask = 0755，这样guest就不能删除了
        wide links = yes|no # 指明是否允许共享外符号连接，比如共享资源里面有个连接指向非共享资源里面的文件或者目录，如果设置wide links = no将使该连接不可用
        max connections = 100 # 设定最大同时连接数
        delete readonly = yes|no # 指明能否删除共享资源里面已经被定义为只读的文件


###防止出现中文目录乱码

`$ export LC_ALL=zh_CN.UTF-8`

在 /etc/samba/smb.conf 中的 [global] 段加上：:

		display charset = UTF-8
		unix charset = UTF-8
		dos charset = cp936
		
		
###Tip: 运行 testparm 检查 samba 的配置文件是否合法。

	$ sudo testparm
		
