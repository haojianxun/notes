# dockerfile构建



\# MageEdu

FROM busybox:latest

MAINTAINER mage

COPY index.html    /web/html/index.html

CMD httpd -f -h /web/



