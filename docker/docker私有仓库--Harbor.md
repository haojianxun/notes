# docker私有仓库--Harbor

下载离线版

```
wget https://storage.googleapis.com/harbor-releases/harbor-offline-installer-v1.5.3.tgz

tar xvf /harbor-offline-installer-v1.5.3.tgz -C /usr/local


cd /usr/local/harbor
vim harbor.cfg

./install.sh
```

停止的话:

```
docker-compose stop
```

