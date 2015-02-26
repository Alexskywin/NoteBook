#Vagrant搭设ubuntu的LNMP环境

安装Vagrant和Virtualbox. 这个不多说, 安装完成两个软件后, 在一个空闲的硬盘分区中

    mkdir ubuntu_lnmp
    vagrant init ubuntu/trusty64

打开vagrantfile打开这两个配置
	
	config.vm.network "private_network", ip: “192.168.10.10"

	config.vm.provider "virtualbox" do |vb|
  	  # Display the VirtualBox GUI when booting the machine
  	  # vb.gui = true
  
      # Customize the amount of memory on the VM:
      vb.memory = "1024"
    end

vagrant up等待安装
vagrant ssh的连接用户名和密码都是vagrant

vagrant环境搞定

	sudo apt-get update

安装nginx
	
	sudo apt-get install nginx

安装Mysql请设置密码, 在远程连接时候, 可以使用SSH方式连接, ssh的用户名和密码都是vagrant
	
	sudo apt-get install mysql-server  mysql-client  libmysqlclient-dev


安装Git版本控制

	sudo apt-get git

安装PHP以及相关模块

	sudo apt-get install php5-fpm php5-mysql php5-cli php5-gd php5-memcache php5-memcached php5-json php5-mcrypt php5-curl php-pear build-essential php5-dev -y
	sudo pecl install xdebug -y
	sudo php5enmod json
	sudo php5enmod mcrypt

删除/usr/share/nginx/html
	
	sudo ln -s /vagrant /usr/share/nginx/html

修改/etc/php5/fpm/php.ini

	cgi.fix_pathinfo = 0
	display_errors = On
	date.timezone = PRC

/etc/nginx/sites-available/default
修改

	server {
    	...
    	//找到index
    	index index.php index.html index.htm

   		...
   		location ~ \.php {
        	#       fastcgi_split_path_info ^(.+\.php)(/.+)$;
        	#       # NOTE: You should have "cgi.fix_pathinfo = 0;" in php.ini
        	#
        	#       # With php5-cgi alone:
                	fastcgi_pass 127.0.0.1:9000;
        	#       # With php5-fpm:
        	#       fastcgi_pass unix:/var/run/php5-fpm.sock;
            	    fastcgi_index index.php;
            	    include fastcgi_params;
            	    set $pathinfo "";
                	set $real_script_name $fastcgi_script_name;
                	if ($fastcgi_script_name ~ "^(.+?\.php)(/.+)$") {
                        set $real_script_name $1;
                        set $path_info $2;
                	}
                	fastcgi_param SCRIPT_FILENAME $document_root$real_script_name;
                	fastcgi_param SCRIPT_NAME $real_script_name;
                	fastcgi_param PATH_INFO $path_info;
        }
	}

这样是为了支持PHP的PHPINFO. 其实不是很难理解, fastcgi_param 配置项名称会出现在$_SERVER[配置项名称]中

修改nginx的用户名称
	
	sudo vi /etc/nginx/nginx.conf

找到user www-data; 改为 user vagrant;

修改配置php-fpm配置

	sudo vi /etc/php5/fpm/pool.d/www.conf

找到以下配置项目修改

	user = vagrant
	group = vagrant

	;listen = /var/run/php5-fpm.sock 注释掉 改为
	listen = 127.0.0.1:9000

	listen.owner = vagrant
	listen.group = vagrant

我的pm.相关配置

	pm.max_children = 1000
	pm.start_servers = 25
	pm.min_spare_servers = 25
	pm.max_spare_servers = 50
	pm.max_requests = 1000

为了防止cli和fpm的php.ini不相同, 可以讲cli的php.ini文件删除, 然后ln过去一个

	sudo rm -rf /etc/php5/cli/php.ini
	ln -s /etc/php5/fpm/php.ini /etc/php5/cli/php.ini

这样修改一个配置文件, 就可以达到两边同步了.

