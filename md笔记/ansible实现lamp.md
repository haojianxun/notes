# ansible实现lamp

> #创建role的步骤
> (1) 创建以roles命名的目录；
> (2) 在roles目录中分别创建以各角色名称命名的目录，如webservers等；
> (3) 在每个角色命名的目录中分别创建files、handlers、meta、tasks、templates和vars目录；用不
> 到的目录可以创建为空目录，也可以不创建；
> (4) 在playbook文件中，调用各角色；
> #role内各目录中可用的文件
> tasks目录：至少应该包含一个名为main.yml的文件，其定义了此角色的任务列表；此文件可以使用in
> clude包含其它的位于此目录中的task文件；
> files目录：存放由copy或script等模块调用的文件；
> templates目录：template模块会自动在此目录中寻找Jinja2模板文件；
> handlers目录：此目录中应当包含一个main.yml文件，用于定义此角色用到的各handler；在handler
> 中使用include包含的其它的handler文件也应该位于此目录中；
> vars目录：应当包含一个main.yml文件，用于定义此角色用到的变量；
> meta目录：应当包含一个main.yml文件，用于定义此角色的特殊设定及其依赖关系；
> default目录：为当前角色设定默认变量时使用此目录；应当包含一个main.yml文件；

```
[root@scholar ~]# vim /etc/ansible/hosts 
#定义被控主机
[webservers]
172.16.10.123 ansible_ssh_user=root ansible_ssh_pass=centos
172.16.10.124 ansible_ssh_user=root ansible_ssh_pass=centos
[dbservers]
172.16.10.125 ansible_ssh_user=root ansible_ssh_pass=centos


实现基于ssh密钥通信
[root@scholar ~]# ssh-keygen -t rsa -P ''
[root@scholar ~]# yum install sshpass -y  #请确保安装sshpass，不然无法通信

#此时可以将/etc/ansible/hosts改为
[webservers]
172.16.10.123 
172.16.10.124 
[dbservers]
172.16.10.125

创建各目录
[root@scholar ~]# mkdir lamp/role -pv
[root@scholar role]# mkdir web/{files,handlers,meta,tasks,templates,vars,default} db/{files,handlers,meta,tasks,templates,vars,default} php/{files,handlers,meta,tasks,templates,vars,default} -p

准备各服务配置文件
[root@scholar role]# cp /etc/httpd/conf/httpd.conf web/files/
[root@scholar role]# cp /etc/php.ini php/files/
[root@scholar role]# cp /etc/my.cnf db/files/

创建各剧本
[root@scholar role]# touch web.yml php.yml db.yml site.yml
[root@scholar role]# touch web/{handlers,tasks}/main.yml db/{handlers,tasks}/main.yml php
/tasks/main.yml
[root@scholar role]# vim web.yml
 
- name: web service
  remote_user: root
  hosts: webservers
  roles:
    - web
 
[root@scholar role]# vim php.yml    
 
- name: php service
  remote_user: root
  hosts: webservers
  roles:
    - php
     
[root@scholar role]# vim db.yml 
 
- name: mysql service
  remote_user: root
  hosts: dbservers
  roles:
    - db
 
[root@scholar role]# vim web/tasks/main.yml     
 
- name: install httpd
  yum: name=httpd state=present
- name: configuration httpd
  copy: src=httpd.conf dest=/etc/httpd/conf/httpd.conf
  notify:
    - restart httpd
- name: service httpd start
  service: name=httpd enabled=no state=started
   
[root@scholar role]# vim web/handlers/main.yml 
 
- name: restart httpd
  service: name=httpd state=restarted
   
[root@scholar role]# vim php/tasks/main.yml 
 
- name: install php
  yum: name=php state=present
- name: configuration php
  copy: src=php.ini dest=/etc/php.ini
   
[root@scholar role]# vim db/tasks/main.yml 
 
- name: install mysql
  yum: name=mysql state=present
- name: install mysql-server
  yum: name=mysql-server state=present
- name: configuration mysqld
  copy: src=my.cnf dest=/etc/my.cnf
  notify:
    - restart mysqld
- name: service mysqld start
  service: name=mysqld enabled=no state=started
   
[root@scholar role]# vim db/handlers/main.yml 
 
- name: restart mysqld
  service: name=mysqld state=restarted

```



