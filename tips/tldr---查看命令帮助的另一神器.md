# tldr---查看命令帮助的另一神器

地址:https://github.com/tldr-pages/tldr

## 在bash中安装

## Installation

```
mkdir -p ~/bin
curl -o ~/bin/tldr https://raw.githubusercontent.com/raylee/tldr/master/tldr
chmod +x ~/bin/tldr
```

Then try using the command! If you get an error such as *-bash: tldr: command not found*, you may need to add `~/bin` to your `$PATH`. On OSX edit `~/.bash_profile` (`~/.bashrc` on Linux), and add the following line to the bottom of the file:

```
export PATH=~/bin:$PATH
```

If you'd like to enable shell completion (eg. `tldr w<tab><tab>` to get a list of all commands which start with w) then add the following to the same startup script:

```
complete -W "$(tldr 2>/dev/null --list)" tldr
```



## 使用docker来安装

Docker images:

- [tldr-docker](https://github.com/nutellinoit/tldr-docker)- Run the `tldr` command via a docker container: `alias tldr='docker run --rm -it -v ~/.tldr/:/root/.tldr/ nutellinoit/tldr'`



其余更多的方式请查看github项目