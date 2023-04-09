+++
title = "脚本分享"
description = "share of my shell script snippet"
date = 2017-04-22T00:00:00+08:00
lastmod = 2022-02-24T15:15:45+08:00
tags = ["shell", "linux", "command_line", "tool"]
categories = ["tool"]
draft = false
toc = true
+++

分享一下平时工作生活中编写的一些脚本片段(一直更新). 适用于 OS X 和 Linux


## <span class="section-num">1</span> 准备工作 {#准备工作}

因为我比较多的脚本都是基于 `percol` 这个神器，所以需要先安装 `percol`, 如果 不了解 `percol` 的话，可以翻看一下我之前的文章 [Linux/Unix Shell 二三事之神器percol](https://ramsayleung.github.io/post/2017/percol/) .

我一般将写好的函数 source 命令添加到 Shell. 例如脚本函数都在一个叫`tool_function.sh` 的文件里面，而我使用 Zsh, 则只需要在 `.zshrc` 添加一句语句：

```shell
source /path/to/tool_function.sh
```

如果使用 Bash, 添加到 `.bashrc` 即可。


## <span class="section-num">2</span> 有趣的脚本 {#有趣的脚本}


### <span class="section-num">2.1</span> SSH 免密码登录 {#ssh-免密码登录}

SSH 基本就是登录远程服务器的标配了，只是每次登录服务器都要输入密码，未免太麻烦了(好吧，我拥有懒惰这个美德)，所以我决定配置 SSH 的免密码登录。代码如下：

```shell
function config_ssh_login_key(){
    if [ $# -lt 3 ];then
       echo "Usage: $(basename $0) -u user -h hostname -p port"
       kill -INT $$
    fi
       #if public/private key doesn't exist ,generate public/private key
       if [ -f ~/.ssh/id_rsa ];then
	  echo "public/private key exists"
	  else
	      ssh-keygen -t rsa
       fi
	  while getopts :u:h:p: option
	  do
	      case "$option" in
		  u) user=$OPTARG;;
		  h) hostname=$OPTARG;;
		  p) port=$OPTARG;;
		  *) echo "Unknown option:$option";;
	      esac
	  done

	  if [ -z "$port" ];then
	     port=22
	  fi
	     #check whether it is the first time to run this script and whether authorized_keys exists
	     # ssh_host_and_user="$1@$2"
	     authorized_keys="$HOME/.ssh/authorized_keys"
	     read -r -s -p "$user@$hostname's password:" password
	     if sshpass -pv $password ssh -p "$port" "$user@$hostname" test -e "$authorized_keys";then
		echo "authorized key exists"
		kill -INT $$
		else
		    sshpass -p $password ssh  $user@$hostname -p $port "mkdir -p ~/.ssh;chmod 0700 .ssh"
		    sshpass -p $password scp -P $port  ~/.ssh/id_rsa.pub $user@$hostname:~/.ssh/authorized_keys
		    # ssh-copy-id "$user@$hostname -p $port"
	     fi
}
```

基本做法就是生成一对公私密钥，然后把公钥发送到服务器。而脚本其他的部分就是判断密钥是否存在，修改密钥权限等工作。用法也很简单，假如你把以上脚本保存到了一个叫 `config_ssh_login_key.sh` 的文件：

```shell
bash config_ssh_login_key.sh -h your-server-ip -u user -p 2222
```

当然，如果你按照我的前面提到的做法，用 source 命令引入脚本，你可以直接在命令行输入：

```shell
config_ssh_login_key -u root -h your-server-ip
```

如果端口未指定，默认端口为 22


### <span class="section-num">2.2</span> 生成若干位密钥 {#生成若干位密钥}

生成若干位的密钥是常见的需求，得益于 Linux/Unix 命令行强大的过滤器，所以只需把命令整理成脚本即可：

```shell
# generate key
function gkey(){
    if [ -n "$1" ];then
       local length="$1"
       else
	   local length=32
    fi
       OS_NAME=$(uname)
       if [ $OS_NAME = "Darwin" ]; then
	   LC_CTYPE=C cat /dev/urandom |tr -cd "[:alnum:]"|head -c "$length";echo
       else
	   cat /dev/urandom |tr -cd "[:alnum:]"|head -c "$length";echo
       fi
}
```

用法：

```shell
gkey 45
```

即生成一个45位字符的随机密钥，如果没有指定长度的话，默认是 32 位。因为 OS X和 Linux 的 `tr` 使用有差异，所以要处理一下


### <span class="section-num">2.3</span> 复制命令行输出 {#复制命令行输出}

有时可能需要复制某个命令的输出，一般的做法都是运行某个命令，用鼠标选中，然后复制。例如在生成密钥之后，需要复制到项目的配置文件。但是每次都要用鼠标，效率实在不高。这个功能其实可以脚本实现：

```shell
OS_NAME=$(uname)
function pclip() {
    if [ $OS_NAME = "CYGWIN" ]; then
	putclip "$@";
    elif [ $OS_NAME = "Darwin" ]; then
	pbcopy "$@";
    else
	if [ -x /usr/bin/xsel ]; then
	    xsel -ib "$@";
	else
	    if [ -x /usr/bin/xclip ]; then
		xclip -selection c "$@";
	    else
		echo "Neither xsel or xclip is installed!"
	    fi
	fi
    fi
}
```

备注：这个脚本不是我原创，取自 [陈斌](http://blog.binchen.org/archive.html) 博客。

在 Linux 运行这脚本需要先安装 xsel 或者是 xclip 命令。结合生成密钥的命令使用：

```shell
gkey -28|pclip
```

这样，生成的密钥就被复制到系统上了。


### <span class="section-num">2.4</span> 复制当前目录 {#复制当前目录}

有时候，我需要复制当前目录下某个文件的路径，但是无论是文件管理器，还是在Shell 中都要用鼠标选中然后复制指定文件的路径，效率不高且很不方便。所以我通过结合 percol 和上面提高的 pclip 函数改进了做法：

```shell
function pwdf()
{
    local current_dir=`pwd`
    local copied_file=`find $current_dir -type f -print |percol`
    echo -n $copied_file |pclip;
}
```

只需在 Shell 中输入 `pwdf`, 然后选择需要复制的路径即可。
运行截图：

{{< figure src="https://i.imgur.com/Ppkm2xV.gif" >}}

---

<span class="timestamp-wrapper"><span class="timestamp">&lt;2017-05-22 Mon&gt; </span></span> Update


### <span class="section-num">2.5</span> 判断 Unix 系统的版本 {#判断-unix-系统的版本}

因为我经常需要在不同的 Unix 机器之间切换，例如工作用的 Mac OS X, 另外一台笔记本上的 Fedora, 还有一台工作站上的 Arch Linux, 以及各种发行版本的 VPS 等，在不同的发行版本或者系统之间切换，我希望我常用的工具也可以很轻易地移植到不同的发行版本上。

但是不同的发行版本使用不同的包安装管理器，例如 OS X 上的 `brew`, Fedora 的 `dnf`, Centos 的 `yum`, Ubuntu 上的 `apt-get` 等等。如果可以通过使用脚本来实现根据不同的发行版本使用不同的包安装管理器安装软件，这样就省心很多。

```shell
# GetOSVersion
function GetOSVersion {

    # Figure out which vendor we are
    if [[ -x "`which sw_vers 2>/dev/null`" ]]; then
	# OS/X
	os_VENDOR=`sw_vers -productName`
    elif [[ -x $(which lsb_release 2>/dev/null) ]]; then
	os_VENDOR=$(lsb_release -i -s)
	if [[ "Debian,Ubuntu,LinuxMint" =~ $os_VENDOR ]]; then
	    os_PACKAGE="deb"
	elif [[ "SUSE LINUX" =~ $os_VENDOR ]]; then
	    lsb_release -d -s | grep -q openSUSE
	    if [[ $? -eq 0 ]]; then
		os_VENDOR="openSUSE"
	    fi
	elif [[ $os_VENDOR == "openSUSE project" ]]; then
	    os_VENDOR="openSUSE"
	elif [[ $os_VENDOR =~ Red.*Hat ]]; then
	    os_VENDOR="Red Hat"
	fi
	os_CODENAME=$(lsb_release -c -s)
    elif [[ -r /etc/redhat-release ]]; then
	# Red Hat Enterprise Linux Server release 5.5 (Tikanga)
	# Red Hat Enterprise Linux Server release 7.0 Beta (Maipo)
	# CentOS release 5.5 (Final)
	# CentOS Linux release 6.0 (Final)
	# Fedora release 16 (Verne)
	# XenServer release 6.2.0-70446c (xenenterprise)
	# Oracle Linux release 7
	os_CODENAME=""
	for r in "Red Hat" CentOS Fedora XenServer; do
	    os_VENDOR=$r
	done
	if [ "$os_VENDOR" = "Red Hat" ] && [[ -r /etc/oracle-release ]]; then
	    os_VENDOR=OracleLinux
	fi
    elif [[ -r /etc/SuSE-release ]]; then
	for r in openSUSE "SUSE Linux"; do
	    if [[ "$r" = "SUSE Linux" ]]; then
		os_VENDOR="SUSE LINUX"
	    else
		os_VENDOR=$r
	    fi
	    os_VENDOR=""
	done
	# If lsb_release is not installed, we should be able to detect Debian OS
    elif [[ -f /etc/debian_version ]] && [[ $(cat /proc/version) =~ "Debian" ]]; then
	os_VENDOR="Debian"
    fi
    export os_VENDOR
}

```


### <span class="section-num">2.6</span> 根据不同的发行版本安装软件 {#根据不同的发行版本安装软件}

刚刚上面的脚本是为了准确判断出所有的 \*nix 系统的，但是方便起见，也可以直接使用`uname` 命令

```shell
if [ "$(uname)" == "Darwin" ]; then
    # Do something under Mac OS X platform
    echo "This is mac os"
    # check if brew exists
    type brew>/dev/null 2>&1 || {
	echo >&2 " require brew but it's not installed.  Aborting.";
	exit 1; }
    echo "install htop"
    brew install htop

    echo "install ag"
    brew install ag

    echo "install httpie"
    brew install httpie

    echo "install fasd"
    brew install fasd

    echo "install tree"
    brew install tree

    echo "install shellcheck"
    brew install shellcheck

    echo "install guile"
    brew install guile

    echo "install proxychains-ng"
    brew install proxychains-ng

    echo "install pandoc"
    brew install pandoc

    echo "install markdown"
    brew install markdown

    echo "install cloc"
    brew install cloc
elif [ "$(expr substr $(uname -s) 1 5)" == "Linux" ]; then
    # Do something under GNU/Linux platform
    GetOSVersion
    if [ "$os_VENDOR" == "Ubuntu" ] || [[ "$os_VENDOR" == "Debian" ]] || [[ "$os_VENDOR" == "LinuxMint" ]]; then
	# install htop
	sudo apt-get install htop -y
	# install httpie
	sudo apt-get install httpie -y
	# install ag
	sudo apt-get install  silversearcher-ag -y
	# install zeal
	sudo apt-get install zeal -y
	# install ncdu
	sudo apt-get install ncdu -y
	# install i3
	sudo apt-get install i3 -y
	# install emacs (i could die without it)
	sudo apt-get install emacs -y
	# install vim
	sudo apt-get install vim -y
	# install tree
	sudo apt-get install tree -y
	# install shellcheck
	sudo apt-get install shellcheck -y
	# install guile (scheme compiler)
	sudo apt-get install guile -y
	# install source code pro font
	[ -d /usr/share/fonts/opentype ] || sudo mkdir /usr/share/fonts/opentype
	sudo git clone https://github.com/adobe-fonts/source-code-pro.git /usr/share/fonts/opentype/scp
	sudo fc-cache -f -v
	# install proxychains-ng
	sudo apt-get install proxychains-ng -y
	# install pandoc
	sudo apt-get install pandoc -y

	sudo apt-get install markdown -y

	sudo apt-get install cloc -y
    elif [  "$os_VENDOR" == "Fedora" ] || [[ "$os_VENDOR" == "CentOS" ]] || [[ "$os_VENDOR" == "Korora" ]]; then
	# install ag
	sudo yum install -y the_silver_searcher
	# install zeal
	sudo yum install -y zeal
	# install httpie
	sudo yum install -y httpie
	# install htop
	sudo yum install -y htop
	# install ncdu
	sudo yum install -y ncdu
	# install vim
	sudo yum install -y vim
	# install emacs
	sudo yum install -y emacs
	# install i3
	sudo yum install -y i3
	# install tree
	sudo yum install -y tree
	# install shellcheck
	sudo yum install ShellCheck -y
	# install guile
	sudo yum install guile -y
	# install source  code pro font
	sudo yum install adobe-source-code-pro-fonts -y
	# install proxychains-ng
	sudo yum install proxychains-ng -y

	sudo yum install pandoc -y

	sudo yum install markdown -y

	# count line and space in code
	sudo yum install cloc  -y
    elif [  "$os_VENDOR" == "Arch" ] ; then
	# install ag
	sudo pacman -S -y the_silver_searcher
	# install zeal
	sudo pacman -S -y zeal
	# install httpie
	sudo pacman -S -y httpie
	# install htop
	sudo pacman -S -y htop
	# install ncdu
	sudo pacman -S -y ncdu
	# install vim
	sudo pacman -S -y vim
	# install emacs
	sudo pacman -S -y emacs
	# install i3
	sudo pacman -S -y i3
	# install tree
	sudo pacman -S -y tree
	# install shellcheck
	sudo pacman -S ShellCheck -y
	# install guile
	sudo pacman -S guile -y
	# install source-code-pro font
	sudo pacman -S adobe-source-code-pro-fonts -y
	# install proxychains-ng
	sudo pacman -S proxychains-ng -y

	sudo pacman -S pandoc -y

	sudo pacman -S markdown -y

	sudo pacman -S ripgrep -y

	sudo pacman -S cloc  -y
    fi
elif [ "$(expr substr $(uname -s) 1 10)" == "MINGW32_NT" ]; then
    # Do something under 32 bits Windows NT platform
    echo "This is 32-bit windows"
elif [ "$(expr substr $(uname -s) 1 10)" == "MINGW64_NT" ]; then
    # Do something under 64 bits Windows NT platform
    echo "this is 64-bit windows"
fi
```


### <span class="section-num">2.7</span> 加密目录 {#加密目录}

每个人都会有需要只属于自己的东西，保护这些东西最好的办法就是对其进行加密：


#### <span class="section-num">2.7.1</span> 加密 {#加密}

使用 `tar` 和 `openssl` 对目录进行加密，先使用 `tar` 归档当前文件，然后使用
`aes256` 算法进行加密：

```sh
tar -czf - * | openssl enc -e -aes256 -out encrypted.tar.gz
```


#### <span class="section-num">2.7.2</span> 解密 {#解密}

把加密后的归档文件解密到当前命令：

```sh
openssl enc -d -aes256 -in encrypted.tar.gz| tar xz -C $(pwd)
```