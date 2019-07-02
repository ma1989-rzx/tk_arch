
Ubuntu系统
### 01，参看系统版本命令
	cat /etc/os-release	查看系统版本

### 02，安装*.deb包命令
	dpkg -i	xxx.deb		安装包
	dpkg -l	xxx		查看安装包状态
	dpkg --remove  xxx	只删除安装包，不删除配置文件
	dpkg --purge  xxx	只删除安装包及配置文件
	dpkg --force-all -i xxx	安装包(会忽略其全部依赖【对于循环依赖包安装有效】)

### 03，下载依赖*.deb包
	手动下载参考地址：https://pkgs.org/
	自动下载参考方法：
		1，准备一个相同版本的服务器(16.04)，(可通过阿里云、腾讯云、华为云等平台买一台服务器，轻量应用服务器)
		2，登录后服务器后
			cd /var/cache/apt/archives  #切换到下载依赖包目录
			apt-get clean	#清理原有依赖
				或者使用  apt-get remove xxx.deb   针对特定的包，使用其命令来卸载
			apt-get install xxx	#安装某服务，获取其安装包
				注意：通过日志记录其安装顺序，遇到某些循环依赖的包，通过强制忽略依赖包来安装
			ll  ./	#查看目录/var/cache/apt/archives下下载的依赖包 xx.deb
				注意：Ubuntu下安装redis依赖的ruby包时，需要有redis.gem 插件
