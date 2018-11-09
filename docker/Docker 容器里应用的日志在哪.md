# Docker 容器里应用的日志在哪

Docker 容器里应用的日志，默认会保存在宿主机的 /var/lib/docker/containers/{{. 容器 ID}}/{{. 容器 ID}}-json.log 文件里