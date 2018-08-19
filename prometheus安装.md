# prometheus安装

1.prometheus下载安装包并解压

```
wget https://github.com/prometheus/prometheus/releases/download/v2.3.2/prometheus-2.3.2.linux-amd64.tar.gz

tar xvzf prometheus-2.3.2.linux-amd64.tar.gz

mv prometheus-2.3.2.linux-amd64.tar.gz  prometheus

cd prometheus
```

1.2.启动

```
./prometheus

查看帮助
./prometheus -h

指定配置文件运行
./prometheus --config.file=prometheus.yml
```





2.安装node_exporter

```
wget https://github.com/prometheus/node_exporter/releases/download/v0.16.0/node_exporter-0.16.0.linux-amd64.tar.gz

tar xvzf node_exporter-0.16.0.linux-amd64.tar.gz

mv node_exporter-0.16.0.linux-amd64 node_exporter
cd node_exporter

启动运行
./node_exporter
```



3.安装alertmanager

```
wget https://github.com/prometheus/alertmanager/releases/download/v0.15.1/alertmanager-0.15.1.linux-amd64.tar.gz

tar xvzf alertmanager-0.15.1.linux-amd64.tar.gz
mv alertmanager-0.15.1.linux-amd64.tar.gz alertmanager
cd alertmanager

启动运行
./alertmanager
```

