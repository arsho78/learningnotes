# Debian Diary #

误按`ctrl-s`导致vim没有反应，按下`ctrl-q`解决此问题

## make uefi boot usb key ##
download all debian dvd images
use the rufus and the 1st dvd image to make a uefi bootable usb key, choose the "gpt for uefi" option when repartition the usb key
mount the rest images and copy all contents in the `pool` directory from each image into the `pool` directory in the usb key
make sure you set your motherboard to uefi only mode, which means not using csm
now can use the usb key to start up the computer and install debian in uefi mode

## Install VirtualBox additions ##

		apt install build-essential module-assistant
		m-a prepare
		cd /media/cdrom && sh ./VBoxLinuxAdditions.run

## Install sudo ##
    apt install sudo
edit /etc/sudoers and adding the following line

		root ALL=(ALL:ALL) ALL
		yourusername ALL=(ALL:ALL) ALL #this is the new line
	

## Install Chinese input method

1. add soteware source 

		sudo vim /etc/apt/sources.list.d/sogoupinyin.list
		deb http://archive.ubuntukylin.com:10006/ubuntukylin xenial main

2. import GPG public key for xenial software source

		sudo apt install dirmngr
		sudo apt-key adv --keyserver keyserver.ubuntu.com --recv-keys D259B7555E1D3C58

3. install sogou Chinese Input

		sudo apt update
		sudo apt install sogoupinyin

4. install lilydjwg/fcitx.vim for vim

## Install brothers printer ##

1. Download the tool.(linux-brprinter-installer-*.*.*-*.gz)
The tool will be downloaded into the default "Download" directory.
(The directory location varies depending on your Linux distribution.)
e.g. `/home/(LoginName)/Download`

2. Open a terminal window.

3. Go to the directory you downloaded the file to in the last step. By using the cd command.
e.g. `cd Downloads`

4. Enter this command to extract the downloaded file:
Command: gunzip linux-brprinter-installer-*.*.*-*.gz
e.g. `gunzip linux-brprinter-installer-2.1.1-1.gz`

5. Get superuser authorization with the "su" command or "sudo su" command.

6. Run the tool:
Command: bash linux-brprinter-installer-*.*.*-* Brother machine name 
e.g. `bash linux-brprinter-installer-2.1.1-1 MFC-J880DW`

7. The driver installation will start. Follow the installation screen directions.
 
 When you see the message `"Will you specify the DeviceURI ?"`,
 For USB Users: Choose N(No)
 For Network Users: Choose Y(Yes) and DeviceURI number.
The install process may take some time. Please wait until it is complete.

8. refer to `http://localhost:631` in a web browser to manage the printer

## Install universal-ctags ##

		git clone https://github.com/universal-ctags/ctags
		cd ctags
		./autogen.sh
		./configure --prefix=/usr/local
		make && sudo make install

## Install git ##

		apt install git

### 安装node.js和npm ###

		//安装nvm（nodejs version manager） 
		curl -o- https://raw.githubusercontent.com/creationix/nvm/v0.34.0/install.sh | bash
		//验证安装
		command -v nvm
		//安装nodejs，"node"直接引用最新的nodejs版本
		nvm install node

### 安装git工具 ###

		//安装commitizen
		npm install -g commitizen
		//安装commitizen配件，具体配件列表参阅它的git网址
		npm install -g cz-conventional-changelog
		//配置使用commitizen
		echo '{ "path": "cz-conventional-changelog" }' > ~/.czrc
		//安装commitlint
		npm install -g @commitlint/cli @commitlint/config-conventional
		//配置commitlint
		echo "module.exports = {extends: ['@commitlint/config-conventional']}" > ~/commitlint.config.js
		//验证commitlint安装成功
		echo "should fail" | commitlint
		echo "fix(server): should succeed" | commitlint
		//配置git hooks
		mkdir -p ~/hooks
		git config --global core.hooksPath /path/to/my/hooks //此例是~/hooks
		//创建能够执行的提交消息的脚本
		touch /path/to/my/hooks/commit-message
		chmod a+x /path/to/my/hooks/commit-message
		//将如下内容拷贝至刚创建的脚本中

		#! /bin/bash

		# run any local commit-msg hook first
		if test -e ./.git/hooks/commit-msg
			then
				sh ./.git/hooks/commit-msg
		fi

		cat $1 | commitlint

		if [ $status != 0 ]
			then
			  exit 1
		fi

		exit 0

		//安装生成CHANGELOG.md的工具
		npm install -g conventional-changelog-cli
		cd my-project
		//生成augular格式的CHANGELOG
		conventional-changelog -p angular -i CHANGELOG.md -s



## 安装vim ##

在最新的debian9中已经将vim更新为8.0版本，但默认安装的vim有很多功能不支持，所以还是选择重新自己编译安装

首先，卸载debian自带的vim

		apt remove vim vim-runtime vim-tiny vim-common gvim vim-gui-common

接下来从github上下载最新的vim
		git clone https://github.com/vim/vim.git

然后安装vim必需的一些依赖

		apt install python python-dev python3 python3-dev \
					ruby ruby-dev \
					perl libperl-dev \
					tcl8.6 tcl8.6-dev libtcl8.6 \
					lua5.3 liblua5.3-dev luajit libluajit-5.1-dev 					
		apt install libx11-dev //clipboard support
		apt install libxt-dev  //clipboard support

并编写一个编译的批处理文件，放在vim目录下
	
		make clean
		git clean -fdx
		
		./configure \
		    --prefix=/usr/local/ \
		    --with-features=huge \
		    --enable-multibyte \
		    --enable-cscope=yes \
		    --enable-perlinterp=yes \
		    --enable-rubyinterp=yes \
		    --with-ruby-command=/usr/bin/ruby \
		    --enable-luainterp=yes \
		    --enable-pythoninterp=yes \
		    --with-python-config-dir=/usr/lib/python2.7/config-x86_64-linux-gnu \
		    --enable-python3interp=yes \
		    --with-python3-config-dir=/usr/lib/python3.5/config-3.5m-x86_64-linux-gnu \
		    --enable-tclinterp=yes \
		    --enable-gui=no \
		    --with-luajit \    
		    --enable-sniff \
		    --enable-xim \
		    --enable-fontset \
		    --with-x \
		    --with-compiledby=$USER \
		    --enable-fail-if-missing
	
		make && src/vim --version

检查一下显示的features中是否齐全，然后运行`make install`安装vim

安装vim的plug管理器vim-plug时，注意轻易不要使用/etc/vim目录下的vimrc配置文件，在用户的主目录下创建自己的.vimrc配置文件，另外不要设置系统变量$VIM指向/etc/vim，否则会出现错误：

		can't open file /etc/vim/syntax/syntax.vim

安装插件管理器vim-plug

		apt install curl
		curl -fLo ~/.vim/autoload/plug.vim --create-dirs \
    	https://raw.githubusercontent.com/junegunn/vim-plug/master/plug.vim
    
编辑`~/.vimrc`文件如下：

    
		set go=
		set nocompatible
		set ai
		set sw=4
		set ts=4
		set cursorline
		exec 'cd ' . fnameescape($HOME)
		set autochdir
		set history=200
		set diffexpr=MyDiff()
		function MyDiff()
		  let opt = '-a --binary '
		  if &diffopt =~ 'icase' | let opt = opt . '-i ' | endif
		  if &diffopt =~ 'iwhite' | let opt = opt . '-b ' | endif
		  let arg1 = v:fname_in
		  if arg1 =~ ' ' | let arg1 = '"' . arg1 . '"' | endif
		  let arg2 = v:fname_new
		  if arg2 =~ ' ' | let arg2 = '"' . arg2 . '"' | endif
		  let arg3 = v:fname_out
		  if arg3 =~ ' ' | let arg3 = '"' . arg3 . '"' | endif
		  if $VIMRUNTIME =~ ' '
		    if &sh =~ '\<cmd'
		      if empty(&shellxquote)
		        let l:shxq_sav = ''
		        set shellxquote&
		      endif
		      let cmd = '"' . $VIMRUNTIME . '\diff"'
		    else
		      let cmd = substitute($VIMRUNTIME, ' ', '" ', '') . '\diff"'
		    endif
		  else
		    let cmd = $VIMRUNTIME . '\diff'
		  endif
		  silent execute '!' . cmd . ' ' . opt . arg1 . ' ' . arg2 . ' > ' . arg3
		  if exists('l:shxq_sav')
		    let &shellxquote=l:shxq_sav
		  endif
		endfunction
		
		call plug#begin('~/.vim/plugged')
		Plug 'scrooloose/nerdtree', { 'on': 'NERDTreeToggle' }
		Plug 'scrooloose/nerdcommenter'
		Plug 'haya14busa/incsearch.vim'
		Plug 'haya14busa/incsearch-fuzzy.vim'
		Plug 'easymotion/vim-easymotion'
		Plug 'haya14busa/incsearch-easymotion.vimVG'
		Plug 'godlygeek/tabular'
		"Plug 'thaerkh/vim-workspace'
		Plug 'altercation/vim-colors-solarized'
		Plug 'ctrlpvim/ctrlp.vim'
		Plug 'vim-airline/vim-airline'
		Plug 'vim-syntastic/syntastic'
		Plug 'SirVer/ultisnips'
		Plug 'majutsushi/tagbar'
		Plug 'tpope/vim-surround'
		Plug 'xuhdev/SingleCompile'
		Plug 'tomasr/molokai'
		call plug#end()
		
		syntax enable
		set background=dark
		let g:solarized_visibility = "high"
		let g:solarized_contrast = "high"
		"let g:solarized_termcolors=16
		colorscheme solarized
		set guifont=source-code-pro:h12
		
		nmap <F9> :SCCompile<cr>
		nmap <F10> :SCCompileRun<cr>
		
		set number
		
		let g:airline_powerline_fonts = 1

进入vim后使用`:PlugUpdate`命令安装插件

change the default editor by adding the following lines into `/etc/profile`

		EDITOR=vim
		export EDITOR

使用`ctrl+w` 切换窗口，或者使用`ctrl+w`再加上`h/j/k/l`键切换至指定方向窗口

### 安装中文帮助 ###

1. 从`http://vimcdoc.sourceforge.net/`下载中文帮助文档压缩文件
2. 解压缩`tar -zxvf vimcdoc-<version>.tar.gz`
3. 进入解压缩后的目录执行`./vimcdoc.sh -i`安装
(ps: 以root身份安装可以使所有用户使用中文文档，否则只限该用户)
4. 可以使用命令`:set helplang`来切换帮助文档的语言，如`:set helplang=en`或者`:set helplang=cn`，也可以在`.vimrc`中指定，如`set helplang=en`
5. 使用`./vimcdoc.sh -u`卸载中文文档



### 使用NERDTree ###

设置如果打开目录则在vim中自动打开NERDTree

		autocmd StdinReadPre * let s:std_in=1
		autocmd VimEnter * if argc() == 1 && isdirectory(argv()[0]) && !exists("s:std_in") | exe 'NERDTree' argv()[0] | wincmd p | ene | endif

设置快捷键

		nmap <C-n> :NERDTreeToggle<cr>

### 使用surround ###

`ys`+范围+符号：给指点范围添加符号

`cs`+旧符号+新符号：使用新符号替换旧符号，如果旧符号是tag，则用t代表

`ds`+（符号）：删除指定符号，缺省是最近符号

use `S` for visual mode 

### 设置easymotion ###


		"easymotion settings
		"<Leader>f{char} to move to {char}
		map <Leader>f <Plug>(easymotion-bd-f)
		nmap <Leader>f <Plug>(easymotion-overwin-f)
		"s{char}{char} to move to {char}{char}
		nmap s <Plug>(easymotion-overwin-f2)
		"move to line
		map <Leader>L <Plug>(easymotion-bd-jk)
		nmap <Leader>L <Plug>(easymotion-overwin-line)
		"move to word
		map <Leader>w <Plug>(easymotion-bd-w)
		nmap <Leader>w <Plug>(easymotion-overwin-w)
		" You can use other keymappings like <C-l> instead of <CR> if you want to
		" use these mappings as default search and somtimes want to move cursor with
		" EasyMotion.
		function! s:incsearch_config(...) abort
		  return incsearch#util#deepextend(deepcopy({
		  \   'modules': [incsearch#config#easymotion#module({'overwin': 1})],
		  \   'keymap': {
		  \     "\<CR>": '<Over>(easymotion)'
		  \   },
		  \   'is_expr': 0
		  \ }), get(a:, 1, {}))
		endfunction
		
		noremap <silent><expr> /  incsearch#go(<SID>incsearch_config())
		noremap <silent><expr> ?  incsearch#go(<SID>incsearch_config({'command': '?'}))
		noremap <silent><expr> g/ incsearch#go(<SID>incsearch_config({'is_stay': 1}))
		" incsearch.vim x fuzzy x vim-easymotion
		
		function! s:config_easyfuzzymotion(...) abort
		  return extend(copy({
		  \   'converters': [incsearch#config#fuzzy#converter()],
		  \   'modules': [incsearch#config#easymotion#module()],
		  \   'keymap': {"\<CR>": '<Over>(easymotion)'},
		  \   'is_expr': 0,
		  \   'is_stay': 1
		  \ }), get(a:, 1, {}))
		endfunction
		
		noremap <silent><expr> <Space>/ incsearch#go(<SID>config_easyfuzzymotion())


### use tagbar ###

press `s` to switch sorting methods: by name/by order

### use ultisnips ###


put the following lines into `.vimrc`

		"UltiSnips settings
		set rtp+=~/projects/snipts/
		let g:UltiSnipsSnippetsDir="~/projects/snipts/UltiSnips/"
		let g:UltiSnipsSnippetDirectories=["UltiSnips"]
		"multiple path should be in order, seperated by comma

## install neovim related plugins ##
need to install python3-pip or python-pip first, 
then use `pip3 install neovim` or `pip install neovim`

## Install Powerline ##

		apt install powerline
		#install powerline fonts
		git clone https://github.com/powerline/fonts.git --depth=1
		# install
		cd fonts
		sudo ./install.sh
		sudo cp -r /root/.local/share/fonts /usr/local/share
		sudo fc-cache -vf /usr/local/share/fonts
		# clean-up a bit
		sudo chmod 777 uninstall.sh
		sudo ./uninstall.sh
		cd ..
		rm -rf fonts

Add the following line into `~/.bashrc` to activate powerline in bash

		. /usr/share/powerline/bindings/bash/powerline.sh

## Change default editor to vim ##

if you installed vim through `apt`, then execute 
`sudo update-alternatives --config editor`
and choose the vim from the given list.

if you manually installed vim like me, you need to register the vim first by executing 
`sudo update-alternatives --install /usr/bin/editor editor /yourVimPathLike(/usr/local/bin/vim) 100` 
then choose the vim from the given list.


## Install termite ##


		git clone --recursive https://github.com/thestinger/termite.git
		git clone https://github.com/thestinger/vte-ng.git
		sudo apt install -y \
			g++ \
			libgtk-3-dev \
			gtk-doc-tools \
			gnutls-bin \
			valac \
			intltool \
			libpcre2-dev \
			libglib3.0-cil-dev \
			libgnutls28-dev \
			libgirepository1.0-dev \
			libxml2-utils \
			gperf
		echo export LIBRARY_PATH="/usr/include/gtk-3.0:$LIBRARY_PATH"
		cd vte-ng && sudo ./autogen.sh && sudo make && sudo make install
		cd ../termite && sudo make && sudo make install
		sudo ldconfig
		sudo mkdir -p /lib/terminfo/x
		sudo ln -s \
		/usr/local/share/terminfo/x/xterm-termite \
		/lib/terminfo/x/xterm-termite
		
		sudo update-alternatives --install /usr/bin/x-terminal-emulator x-terminal-emulator /usr/local/bin/termite 60

## Install tmux ##

1. `sudo apt install tmux`
2. use customized .tmux.conf at [https://github.com/gpakosz/.tmux](https://github.com/gpakosz/.tmux "customized .tmux.conf")

		cd
		git clone https://github.com/gpakosz/.tmux.git
		ln -s -f .tmux/.tmux.conf
		cp .tmux/.tmux.conf.local .
	
3. modify the `~/.tmux.conf.local`
	1. adding `bind \ splitw -h` at the end.(personal preference)
	2. active the vi mode by uncomment the `#set -g mode-keys vi`

set no delay for esc key

		set -s escape-time 0

## Install Java ##

After downloading the java package from oracle website, run the following code:

		tar -zxvf jdk1.8.0_152.tar.gz
		sudo mkdir /usr/lib/java
		sudo mv jdk1.8.0_152 /usr/lib/java
		sudo update-alternatives --install "/usr/bin/java" "java" "/usr/lib/java/jdk1.8.0_152/bin/java" 1
		sudo update-alternatives --install "/usr/bin/javac" "javac" "/usr/lib/java/jdk1.8.0_152/bin/javac" 1
		sudo update-alternatives --install "/usr/bin/javaws" "javaws" "/usr/lib/java/jdk1.8.0_152/bin/javaws" 1
		
		sudo update-alternatives --set java /usr/lib/java/jdk1.8.0_152/bin/java
		sudo update-alternatives --set javac /usr/lib/java/jdk1.8.0_152/bin/javac
		sudo update-alternatives --set javaws /usr/lib/java/jdk1.8.0_152/bin/javaws
		

update `$JAVA_HOME` by addding the following lines at the end of `/etc/profile`

		JAVA_HOME=/usr/lib/java/jdk1.8.0_152
		export JAVA_HOME
		PATH=$PATH:$HOME/bin:$JAVA_HOME/bin
		export PATH
		
Remove an older version


		sudo update-alternatives --remove javac /usr/local/java/jdk1.8.0_144/bin/javac
		sudo update-alternatives --remove java /usr/local/java/jdk1.8.0_144/bin/java
		sudo update-alternatives --remove javaws /usr/local/java/jdk1.8.0_144/bin/javaws
		

## Install i3-gaps ##

install the dependencies:

		sudo apt install libxcb-keysyms1-dev libpango1.0-dev libxcb-util0-dev xcb libxcb1-dev libxcb-icccm4-dev libyajl-dev libev-dev libxcb-xkb-dev libxcb-cursor-dev libxkbcommon-dev libxcb-xinerama0-dev libxkbcommon-x11-dev libstartup-notification0-dev libxcb-randr0-dev libxcb-xrm0 libxcb-xrm-dev

install i3-gaps

		cd /path/where/you/want/the/repository
		
		# clone the repository
		git clone https://www.github.com/Airblader/i3 i3-gaps
		cd i3-gaps
		
		# compile & install
		autoreconf --force --install
		rm -rf build/
		mkdir -p build && cd build/
		
		# Disabling sanitizers is important for release versions!
		# The prefix and sysconfdir are, obviously, dependent on the distribution.
		../configure --prefix=/usr --sysconfdir=/etc --disable-sanitizers
		make
		sudo make install
		
once logged into i3, press `alt + enter` to open a terminal window and install i3status

		sudo apt install i3status

install necessary package to use dmenu 

		sudo apt install suckless-tools

change default terminal by adding the following line into `/etc/profile`

		TERMINAL=termite
		export TERMINAL



## touchcursor ##

1. install xcape
	$ sudo apt-get install git gcc make pkg-config libx11-dev libxtst-dev libxi-dev
	$ git clone https://github.com/alols/xcape.git
	$ cd xcape
	$ make
	$ sudo make install
2. install autokey
	sudo add-apt-repository ppa:sporkwitch/autokey
	sudo apt update
	sudo apt install autokey-gtk
	# Or alternatively, to install the Qt5 based GUI:
	sudo apt install autokey-qt

## maven ##

1. download and uncompress to /opt/apache/maven
2. `sudo ln -s /opt/apache/maven/apache-maven-x.x.x /usr/local/maven`

## nexus ##

1. download and uncompress to `/opt/nexus`
2. `sudo ln -s /opt/nexus/nexus-x.x.x-xx /usr/local/nexus`
3. 
		# 创建用户并指定用户目录
		$ sudo useradd -d /home/nexus -m nexus
		# 授权
		$ sudo chown -R nexus:nexus /home/nexus
		$ sudo chown -R nexus:nexus /opt/nexus-3.11.0-01
		$ sudo chown -R nexus:nexus /opt/sonatype-work/

4. 在作为服务运行之前，先更改一下 Nexus Repository Manager 运行的端口。找到 /opt/sonatype-work/nexus3/etc/ 目录下的 nexus.properties 文件进行修改：

		application-port=8090

5. add as service. Create a file called nexus.service. Add the following contents, then save the file in the  /etc/systemd/system/ directory:

		[Unit]
		Description=nexus service
		After=network.target
		  
		[Service]
		Type=forking
		ExecStart=/opt/nexus/bin/nexus start
		ExecStop=/opt/nexus/bin/nexus stop
		User=nexus
		Restart=on-abort
		  
		[Install]
		WantedBy=multi-user.target


## tomcat ##

1. 创建tomcat专有用户

		# groupadd tomcat
		# useradd -g tomcat -s /bin/false tomcat
		或
		# useradd -g tomcat -s /sbin/nologin tomcat

		注意：

		-g tomcat用户隶属于tomcat组
		-s /bin/false 禁用shell访问

2. Apache Tomcat 8.5下载安装与配置&设置用户组权限并创建软连接

		# cd /tmp
		# wget http://apache.fayea.com/tomcat/tomcat-8/v8.5.16/bin/apache-tomcat-8.5.16.tar.gz
		# tar zxvf apache-tomcat-8.5.16.tar.gz
		# mv apache-tomcat-8.5.16 /usr/local/
		# cd /usr/local/
		# chown -hR tomcat:tomcat apache-tomcat-8.5.16
		# ln -s apache-tomcat-8.5.16 tomcat

3. 添加tomcat自启动systemd服务单元文件

		# vim /lib/systemd/system/tomcat.service

		[Unit]
		Description=Apache Tomcat 8
		After=syslog.target network.target

		[Service]
		Type=forking
		User=tomcat
		Group=tomcat

		Environment=JAVA_HOME=/usr/local/jdk/jre
		Environment=CATALINA_PID=/usr/local/tomcat/temp/tomcat.pid
		Environment=CATALINA_HOME=/usr/local/tomcat
		Environment=CATALINA_BASE=/usr/local/tomcat
		Environment='CATALINA_OPTS=-Xms512M -Xmx4096M -server -XX:+UseParallelGC'
		Environment='CATALINA_OPTS=-Dfile.encoding=UTF-8 -server -Xms2048m -Xmx2048m -Xmn1024m -XX:SurvivorRatio=10 -XX:MaxTenuringThreshold=15 -XX:NewRatio=2 -XX:+DisableExplicitGC'
		Environment='JAVA_OPTS=-Djava.awt.headless=true -Djava.security.egd=file:/dev/./urandom'

		ExecStart=/usr/local/tomcat/bin/startup.sh
		ExecStop=/bin/kill -15 $MAINPID
		Restart=on-failure

		[Install]
		WantedBy=multi-user.target

4. 重载systemd服务单元，给予软连接目录权限，启动Apache Tomcat服务并设置Tomcat为开机自启动

		# systemctl daemon-reload
		# cd /usr/local/
		# chown -hR tomcat:tomcat tomcat
		# systemctl start tomcat
		# systemctl enable tomcat

5. 配置Apache Tomcat用户实现远程登录

在tomcat-users.xml文件<tomcat-users></tomcat-users>中间添加；

		# vim /usr/local/tomcat/conf/tomcat-users.xml
		<role rolename="manager-gui"/>
		<user username="tomcat" password="s3cret" roles="manager-gui"/>

tomcat8.5之后的版本，已经增强远程登录安全过滤规则，默认不支持远程登录，需要修改配置文件。

修改文件:

		/host-manager/META-INF/context.xml
		/manager/META-INF/context.xml

默认值:

		<Valve className="org.apache.catalina.valves.RemoteAddrValve"
		 allow="127\.\d+\.\d+\.\d+|::1|0:0:0:0:0:0:0:1" />

修改为:

    <Valve className="org.apache.catalina.valves.RemoteAddrValve"
     addConnectorPort="true"
     allow="127\.\d+\.\d+\.\d+;\d*|::1;\d*|0:0:0:0:0:0:0:1;\d*|.*;8080"/>

    详情可查看 Valve 文档

6. 配置Firewalld防火墙

    如果不放行8080端口，就无法在外部使用8080进行访问，现在将端口放行并重载firewall服务

		# firewall-cmd --zone=public --add-port=8080/tcp --permanent
		# firewall-cmd --reload
		# firewall-cmd --list-ports
		# firewall-cmd --list-services

改变服务端口
		
		# /usr/local/tomcat/conf/server.xml
		<Connector prot="8888" protocol="HTTP/1.1" connectionTimeout="20000" redirectPort="8443" />


列出Web应用根路径下的所有页面

		# /usr/local/tomcat/conf/web.xml
		<init-param>
			<param-name>listings</param-name>
			<param-name>true</param-name>
		</init-param>


#### Debian 9 安装 Chrome ####

1. 系统需求

Debian 9 系统必须是 64 位系统，Chrome 没有 32 位系统的软件包。

2. 下载并安装谷歌官方秘钥

		wget -q -O - https://dl.google.com/linux/linux_signing_key.pub | sudo apt-key add -

如上 wget 命令使用 -O - 选项，把文件的内容输出到标准输出，然后使用管道传递给 apt-key 命令作为标准输入，apt-key 命令使用 add - 选项从标准输入读取信息，并添加秘钥。

3. 在 Debian 9 系统中添加谷歌官网软件源

		echo "deb http://dl.google.com/linux/chrome/deb/ stable main" | sudo tee /etc/apt/sources.list.d/google-chrome.list

如上首先使用echo 命令把软件源信息打印在标准输出上，然后使用管道传递给 tee 命令作为标准输入，tee 命令从标准输入读取内容 deb http://dl.google.com/linux/chrome/deb/ stable main，然后把内容写入文件，具体查看 Linux tee 命令详解。

4. 更新 Debian 9 软件源

		sudo apt-get update

5. 安装 Chrome 浏览器稳定版

		sudo apt-get -y install google-chrome-stable




#### Intellij Idea ####

解决编辑窗口在使用autokey或切换窗口后失去焦点的方法：
Help->Edit customized properties, 在打开的properties设置文件中输入：suppress.focus.stealing=false

		


