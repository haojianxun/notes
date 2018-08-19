

# 安装zsh

1.查看当前系统有哪些shell

```
cat /etc/shells

如果没有zsh就进行安装
```

2.安装zsh

```
yum install -y zsh  //CentOS系统

apt-get install -y zsh //ubuntu系统

#如果没有安装curl或者wget或者git  建议安装一下
yum install -y curl wget git

查看是否安装成功
zsh --version
```

```
方法1:
使用脚本安装
sh -c "$(curl -fsSL https://raw.githubusercontent.com/robbyrussell/oh-my-zsh/master/tools/install.sh)"

或者使用wget获取脚本
sh -c "$(wget https://raw.githubusercontent.com/robbyrussell/oh-my-zsh/master/tools/install.sh -O -)"

安装完成之后可以修改主题:
在家目录下 vi ~/.zshrc
修改ZSH_THEME="agnoster"
#主题预览地址:https://github.com/robbyrussell/oh-my-zsh/wiki/Themes

方法2:
git clone https://github.com/robbyrussell/oh-my-zsh.git ~/.oh-my-zsh
cp ~/.oh-my-zsh/templates/zshrc.zsh-template ~/.zshrc
chsh -s /bin/zsh

vi .zshrc
ZSH_THEME="agnoster" # (this is one of the fancy ones)
# see https://github.com/robbyrussell/oh-my-zsh/wiki/Themes#agnoster


```

zsh搭配git  

![img](https://upload-images.jianshu.io/upload_images/3995745-542011a4e02aea1d.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/700) 



