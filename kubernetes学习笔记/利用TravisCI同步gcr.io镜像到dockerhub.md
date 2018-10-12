# 利用TravisCI同步gcr.io镜像到dockerhub

利用google cloud sdk去查看谷歌 镜像站的镜像 , 获取到之后就同步到自己的dockerhub上, 在github上的项目构建2个分支 , 一个master , 用来存放项目 , 一个develop分支, 用来放构建出来的镜像



## 安装travis

我们用rvm和gem来安装travis

### 安装rvm

官方网站:www.rvm.io  具体的安装步骤都在官网

```
curl -sSL https://get.rvm.io | bash -s stable
```

执行这个脚本 , 等待一段时间执行完成

```
rvm version       //在线脚本安装完成之后 查看rvm是否安装成功
```

### 安装ruby

不能用yum安装或者apt-get安装ruby 会报错 , 我们用rvm来安装

```
rvm install ruby
```

等待一段时间 , 国内网络的关系 , 所以安装较慢 

```
ruby --version     //查看ruby是否安装成功
```

### 修改ruby的镜像源

我们用国内的镜像源 , 这样比较快

ruby国内镜像源官网: https://gems.ruby-china.com/

官网介绍:

> [RubyGems](http://rubygems.org/) 一直以来在国内都非常难访问到，在本地你或许可以翻墙，当你要发布上线的时候，你就很难搞了！
>
> 这是一个完整 [RubyGems](https://rubygems.org/) 镜像，你可以用此代替官方版本，我们是完全基于 CDN 技术来实现，能确保几乎无延迟的同步。

##### 如何使用？

请尽可能用比较新的 RubyGems 版本，建议 2.6.x 以上。

```
gem update --system # 这里请翻墙一下
gem -v
```

```
gem sources --add https://gems.ruby-china.com/ --remove https://rubygems.org/
gem sources -l   # 确保只有 gems.ruby-china.com
```

### 安装travis

```
gem install travis
```

这样travis就算完成了  , 之后我们会用到



## 添加私钥到代码仓库

```
 ssh-keygen -t -b 2048 rsa -P '' -C 'haojinaxun@gmail.com'
```

- 上述命令中`-t`是表示要用什么加密算法 我们用rsa算法
- `-P` 表示的是密码(password)  `''` 表示我们的密码为空
- `-C` 是注释信息
- `-b 2048` 表示我们用多少位加密 我们用2048位  一般这个数字是1024的倍数
- 默认的私钥生成文件生成到了`~/.ssh/`下 `.ssh`文件夹中有2个文件 , 一个是`id_rsa`  , 这个是用户谷刚刚生成的私钥 , 一个是`id_rsa.pub`  , 这个文件是生成的公钥

### 生成的公钥`~/.ssh/id_ras.pub` 的内容放在github上的`SSH and GPG keys` 

```
cat ~/.ssh/id_ras.pub   //将这个公钥的内容复制下来
```



之后登陆github , 找到`settings`一项  点击 进入设置 , 找到`SSH and GPG keys`  , 点击`New SSH key` 一项 , 把刚刚复制的公钥内容贴进去 



## 配置google cloud

用google cloud的作用是 用它来进到google的镜像站 , 去获取镜像的tag , 我们先配置出一个用来访问镜像tag的服务账号

先进入到google cloud网站 , 网站地址: https://console.cloud.google.com/home/dashboard   这个过程需要梯子

之后就是创建服务账号

![](https://md-image-1257428480.cos.ap-shanghai.myqcloud.com/pic-user-blog/google%20cloud%E6%9C%8D%E5%8A%A1%E8%B4%A6%E5%8F%B7.png?q-sign-algorithm=sha1&q-ak=AKIDNTfgVCZGqUAnTDhocmyBQoCjJIMEZTu8&q-sign-time=1539336954;1539337854&q-key-time=1539336954;1539337854&q-header-list=&q-url-param-list=&q-signature=19998737aace69317516e3c658897775a792338e)

进入之后创建服务账号

![](https://md-image-1257428480.cos.ap-shanghai.myqcloud.com/pic-user-blog/%E5%88%9B%E5%BB%BA%E6%9C%8D%E5%8A%A1%E8%B4%A6%E5%8F%B7.png?q-sign-algorithm=sha1&q-ak=AKIDNTfgVCZGqUAnTDhocmyBQoCjJIMEZTu8&q-sign-time=1539336977;1539337877&q-key-time=1539336977;1539337877&q-header-list=&q-url-param-list=&q-signature=efd1e23d1467371caf14d2bf916a5e0408b61330)





创建完成之后就创建密钥 , 之后下载下来保存 , 保存为gcloud.config.json

![](https://md-image-1257428480.cos.ap-shanghai.myqcloud.com/pic-user-blog/%E5%88%9B%E5%BB%BA%E5%AF%86%E9%92%A5.png?q-sign-algorithm=sha1&q-ak=AKIDNTfgVCZGqUAnTDhocmyBQoCjJIMEZTu8&q-sign-time=1539337000;1539337900&q-key-time=1539337000;1539337900&q-header-list=&q-url-param-list=&q-signature=b0101ac87075ecf0b8a9dd2521ab72bc1ee7d963)



## 登陆dockerhub

没有账号的话 先去dockerhub上注册

之后在主机上登陆dockerhub

```
docker login
```

登陆完成之后 , 我们的用户登陆信息报错在`~/.docker/config.json`



## 打包用户授权文件给travisCI用

首先把google cloud的服务账号的密钥拷贝一份到用户家目录 , 可以用`rz` 命令来操作

```
rz
```



之后 , 我们在自己家目录下执行下面命令:

```
tar zcf conf.tar.gz  .docker/config.json  gcloud.config.json  .ssh/id_rsa
```



## 创建github项目目录和编写`.travis.yml` 

之后我们在github上面创建一个github项目 , 命名为mirror-grc.io  (github网站:github.com   没有账号的话自己注册一个)  

###创建项目

创建项目完成之后 ,  回到我们自己主机上 , 也创建个相同名称的目录



不过在这之前 ,  先安装软件git

```
yum install -y git
```

创建目录

```
mkdir mirror-grc.io && cd mirror-grc.io
```



之后进入目录下 , 进行git初始化

```
git init

git remote add origin git@github.com:haojianxun/mirror-grc.io.git

git config --global user.name "haojianxun"
git config --global user.email haojinaxun@gmail.com
```



在目录下 , 创建travis用的文件 , 创建一个`.travis.yml`的文件

```
touch .travis.yml
```

我们把用到的密钥文件进行加密处理

```
travis encrypt-file ~/conf.tar.gz --add  //就是把刚刚在用户家目录下打包的conf.tar.gz,进行加密
```

查看`.travis.yml`文件 , 而后对其修改

```
cat .travis.yml
```

加密内容会生成在文件里, 去掉`~\/conf.tar.gz`里的转义斜线`\`  变成类似于这样的信息`-in conf.tar.gz.enc -out ~/conf.tar.gz -d`   , 把生成内容中的`~\/conf.tar.gz` 转义斜线去掉即可



###编写`.travis.yml`

之后编写.`travis.yml`  , 我编写完是这样的 , 大家也可以进行copy

`cat .travis.yml`

```
sudo: required
language: python
python:
- '2.7'
addons:
  apt:
    packages:
    - docker-ce
branches:
  only:
  - master
install:
- git remote -v
script:
- "./get_image.sh"
before_install:
- export start_time=$(date +%s)
- mkdir -p ~/.docker
- mkdir -p ~/.ssh
- openssl aes-256-cbc -K $encrypted_9d3b19838c65_key -iv $encrypted_9d3b19838c65_iv
  -in .travis/conf.tar.gz.enc -out ~/conf.tar.gz -d
- tar xf ~/conf.tar.gz -C ~
- mv ~/id_rsa ~/.ssh/id_rsa
- mv ~/config.json ~/.docker/config.json
- chmod 600 ~/.ssh/id_rsa
- chmod 600 ~/.docker/config.json
- eval $(ssh-agent)
- ssh-add ~/.ssh/id_rsa
```

最后整理下目录

将之前生成的`conf.tar.gz.enc` 放到一个单独目录下`.travis`

```
cd mirror-grc.io
mkdir .travis
cd .travis
cp /root/mirror-grc.io/conf.tar.gz.enc .
```



## 编写同步google镜像脚本

进入项目目录下 , 编写一个脚本 , 我的脚本如下:

cat get_image.sh

```bash
#!/bin/bash

GCR_NAMESPACE=gcr.io/google-containers
DOCKERHUB_NAMESPACE=haojianxun

today(){
   date +%F
}

git_init(){
    git config --global user.name "haojianxun"
    git config --global user.email haojinaxun@gmail.com
    git remote rm origin
    git remote add origin git@github.com:haojianxun/mirror-grc.io.git
    git pull
    if git branch -a |grep 'origin/develop' &> /dev/null ;then
        git checkout develop
        git pull origin develop
        git branch --set-upstream-to=origin/develop develop
    else
        git checkout -b develop
        git pull origin develop
    fi
}

git_commit(){
     local COMMIT_FILES_COUNT=$(git status -s|wc -l)
     local TODAY=$(today)
     if [ $COMMIT_FILES_COUNT -ne 0 ];then
        git add -A
        git commit -m "Synchronizing completion at $TODAY"
        git push -u origin develop
     fi
}

add_yum_repo() {
cat > /etc/yum.repos.d/google-cloud-sdk.repo <<EOF
[google-cloud-sdk]
name=Google Cloud SDK
baseurl=https://packages.cloud.google.com/yum/repos/cloud-sdk-el7-x86_64
enabled=1
gpgcheck=1
repo_gpgcheck=1
gpgkey=https://packages.cloud.google.com/yum/doc/yum-key.gpg
       https://packages.cloud.google.com/yum/doc/rpm-package-key.gpg
EOF
}

add_apt_source(){
export CLOUD_SDK_REPO="cloud-sdk-$(lsb_release -c -s)"
echo "deb http://packages.cloud.google.com/apt $CLOUD_SDK_REPO main" | sudo tee -a /etc/apt/sources.list.d/google-cloud-sdk.list
curl https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -
}

install_sdk() {
    local OS_VERSION=$(grep -Po '(?<=^ID=")\w+' /etc/os-release)
    local OS_VERSION=${OS_VERSION:-ubuntu}
    if [[ $OS_VERSION =~ "centos" ]];then
        if ! [ -f /etc/yum.repos.d/google-cloud-sdk.repo ];then
            add_yum_repo
            yum -y install google-cloud-sdk
        else
            echo "gcloud is installed"
        fi
    elif [[ $OS_VERSION =~ "ubuntu" ]];then
        if ! [ -f /etc/apt/sources.list.d/google-cloud-sdk.list ];then
            add_apt_source
            sudo apt-get -y update && sudo apt-get -y install google-cloud-sdk
        else
             echo "gcloud is installed"
        fi
    fi
}

auth_sdk(){
    local AUTH_COUNT=$(gcloud auth list --format="get(account)"|wc -l)
    if [ $AUTH_COUNT -eq 0 ];then
        gcloud auth activate-service-account --key-file=$HOME/gcloud.config.json
    else
        echo "gcloud service account is exsits"
    fi
}

repository_list() {
    if ! [ -f repo_list.txt ];then
        gcloud container images list --repository=${GCR_NAMESPACE} --format="value(NAME)" > repo_list.txt && \
        echo "get repository list done"
    else
        /bin/mv  -f repo_list.txt old_repo_list.txt
        gcloud container images list --repository=${GCR_NAMESPACE} --format="value(NAME)" > repo_list.txt && \
        echo "get repository list done"
        DEL_REPO=($(diff  -B -c  old_repo_list.txt repo_list.txt |grep -Po '(?<=^\- ).+|xargs')) && \
        rm -f old_repo_list.txt
        if [ ${#DEL_REPO} -ne 0 ];then
            for i in ${DEL_REPO[@]};do
                rm -rf ${i##*/}
            done
        fi
    fi
}

generate_changelog(){
    if  ! [ -f CHANGELOG.md ];then
        echo  >> CHANGELOG.md
    fi

}

push_image(){
    GCR_IMAGE=$1
    DOCKERHUB_IMAGE=$2
    docker pull ${GCR_IMAGE}
    docker tag ${GCR_IMAGE} ${DOCKERHUB_IMAGE}
    docker push ${DOCKERHUB_IMAGE}
    echo "$IMAGE_TAG_SHA" > ${IMAGE_NAME}/${i}
    sed -i  "1i\- ${DOCKERHUB_IMAGE}"  CHANGELOG.md
}

clean_images(){
     IMAGES_COUNT=$(docker image ls|wc -l)
     if [ $IMAGES_COUNT -gt 1 ];then
         docker image prune -a -f
     fi
}

clean_disk(){
    DODCKER_ROOT_DIR=$(docker info --format '{{json .}}'|jq  -r '.DockerRootDir')
    USAGE=$(df $DODCKER_ROOT_DIR|awk -F '[ %]+' 'NR>1{print $5}')
    if [ $USAGE -eq 80 ];then
        wait
        clean_images
    fi
}

main() {
    git_init
    install_sdk
    auth_sdk
    repository_list
    generate_changelog
    TODAY=$(today)
    PROGRESS_COUNT=0
    LINE_NUM=0
    LAST_REPOSITORY=$(tail -n 1 repo_list.txt)
    while read GCR_IMAGE_NAME;do
        let LINE_NUM++
        IMAGE_INFO_JSON=$(gcloud container images list-tags $GCR_IMAGE_NAME  --filter="tags:*" --format=json)
        TAG_INFO_JSON=$(echo "$IMAGE_INFO_JSON"|jq '.[]|{ tag: .tags[] ,digest: .digest }')
        TAG_LIST=($(echo "$TAG_INFO_JSON"|jq -r .tag))
        IMAGE_NAME=${GCR_IMAGE_NAME##*/}
        if [ -f  breakpoint.txt ];then
           SAVE_DAY=$(head -n 1 breakpoint.txt)
           if [[ $SAVE_DAY != $TODAY ]];then
             :> breakpoint.txt
           else
               BREAK_LINE=$(tail -n 1 breakpoint.txt)
               if [ $LINE_NUM -lt $BREAK_LINE ];then
                   continue
               fi
           fi
        fi
        for i in ${TAG_LIST[@]};do
            GCR_IMAGE=${GCR_IMAGE_NAME}:${i}
            DOCKERHUB_IMAGE=${DOCKERHUB_NAMESPACE}/${IMAGE_NAME}:${i}
            IMAGE_TAG_SHA=$(echo "${TAG_INFO_JSON}"|jq -r "select(.tag == \"$i\")|.digest")
            if [[ $GCR_IMAGE_NAME == $LAST_REPOSITORY ]];then
                LAST_TAG=${TAG_LIST[-1]}
                LAST_IMAGE=${LAST_REPOSITORY}:${LAST_TAG}
                if [[ $GCR_IMAGE  == $LAST_IMAGE ]];then
                    wait
                    clean_images
                fi
            fi
            if [ -f $IMAGE_NAME/$i ];then
                echo "$IMAGE_TAG_SHA"  > /tmp/diff.txt
                if ! diff /tmp/diff.txt $IMAGE_NAME/$i &> /dev/null ;then
                     clean_disk
                     push_image $GCR_IMAGE $DOCKERHUB_IMAGE &
                     let PROGRESS_COUNT++
                fi
            else
                mkdir -p $IMAGE_NAME
                clean_disk
                push_image $GCR_IMAGE $DOCKERHUB_IMAGE &
                let PROGRESS_COUNT++
            fi
            COUNT_WAIT=$[$PROGRESS_COUNT%50]
            if [ $COUNT_WAIT -eq 0 ];then
               wait
               clean_images
               git_commit
            fi
        done
        if [ $COUNT_WAIT -eq 0 ];then
            wait
            clean_images
            git_commit
        fi

        echo "sync image $MY_REPO/$IMAGE_NAME done."
        echo -e "$TODAY\n$LINE_NUM" > breakpoint.txt
    done < repo_list.txt 
    sed -i "1i-------------------------------at $(date +'%F %T') sync image repositorys-------------------------------"  CHANGELOG.md
    git_commit
}

main
```

里面有些个人信息 , 把他替换成自己的信息即可 , 脚本的名称如果想改名称 , 改完之后要把`.travis,yml`中的脚本 名称也改了

## 设置travis

进入travis官网  官网地址: https://travis-ci.org

用自己的github账号登陆, 登陆上去之后同步自己的仓库 , 打开要同步的仓库即可

![](https://md-image-1257428480.cos.ap-shanghai.myqcloud.com/pic-user-blog/%E5%90%8C%E6%AD%A5travis%E4%BB%93%E5%BA%93.png?q-sign-algorithm=sha1&q-ak=AKIDNTfgVCZGqUAnTDhocmyBQoCjJIMEZTu8&q-sign-time=1539337028;1539337928&q-key-time=1539337028;1539337928&q-header-list=&q-url-param-list=&q-signature=8826298aa74509521264ef87e17b88589fae27b3)

## 推送仓库到github上

进入到仓库目录中

```
git add .
git commit -m "v1"
git push -u origin master

git checkout -b develop
git add -A
git commit -m 'develop'
git remote add origin git@github.com:haojianxun/mirror-grc.io.git
git push origin develop -f
```

一旦master分支上有变动 , travis就会进行构建 , 每次构建时常50分钟 ,  当然时间到了 你可以进行重新构建

也可以在自己机器上执行 , 重新构建 (写个定时任务也好 , 这样就能同步更新了)

```
travis restart -r haojianxun/mirror-grc.io
```









