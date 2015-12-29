Transmission
====

Transmission 是个开源的小型BT下载客户端，能用命令行，桌面GUI和Web对任务进行管理。

首先安装Transmission 的后台服务，
	
	# apt-get install transmission-daemon

(注意不是transmission要加-daemon)    
默认情况下，守护进程会以 transmission 的用户身份运行。

注意：先关闭transmission服务（/etc/init.d/transmission-daemon stop）再改参数，不然可能会被程序保存在内存里的上次配置覆盖
	
	# /etc/init.d/transmission-daemon stop

然后去/etc/transmission-daemon/目录下修改settings。打开这个json文件，找到那一堆rpc-开头的配置参数。

	# vi /etc/transmission-daemon/settings.json
	
需要修改的参数：   

* rpc-password     
  登录密码，自己随便改，直接填写明文，启动程序后它会帮你加密，下次打开就是密文了
* rpc-username       
  用户名，自己填
* rpc-whitelist    
  白名单，如果下面那个rpc-whitelist-enabled是true的话，那么就只有符合这项参数的IP才能访问，可以用星号'\*' 进行通配，比如192.168.1.*



改好后,  重新启动 transmission-daemon
	
	# /etc/ini.d/transmission-daemon start
在浏览器里输入IP: 9091就能访问Web管理界面了。

-----

###更详细的配置

编辑settings.json配置文件（//后为说明，0为关闭，1为启用）

	{
	"blocklist-enabled": 0,                  //阻挡列表：关闭
	"download-dir": "/var/lib/transmission-daemon/downloads",//下载目录 
	"download-limit": 100,                   //下载限速：100  
	"download-limit-enabled": 0,          //下载限速：关闭  
	"encryption": 1,                            //加密：启用  
	"max-peers-global": 200,              //全局最大：启用  
	"peer-port": 6620,                       //incoming端口：6620  
	"pex-enabled": 1,                         //种子交换：启用  
	"port-forwarding-enabled": 0,        //端口转发：关闭  
	"rpc-authentication-required": 0,    //web登陆密码验证：关闭  
	"rpc-password": "",                       //web登陆密码   
	"rpc-port": 9091,                          //web登陆端口：9091  
	"rpc-username": "",                       //web登陆用户名  
	"rpc-whitelist": "127.0.0.1,192.168.0.*",   //web登陆验证IP  
	"upload-limit": 100,                                 //上传限速：100
	"upload-limit-enabled": 0                         //上传限速：关闭  
	}

如果你更改了下载目录, 请确保 transmission 用户对新的下载目录有读写权限。
建议最好将下载目录的用户和用户组都设置为transmission的用户组;

Ubuntu下一般默认为debian-transmission:debian-transmission.


Sean Guo
2014/04/22