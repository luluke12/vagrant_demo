# 5 Provisioning

## 5-1 什么需要 Provisioning

1. 创建一个box,需要自己安装很多东西

   ```bash
   vagrant status
   vagrant ssh
   sudo yum install -y epel-release
   sudo yum install -y nginx
   sudo systemctl start nginx
   ip -c a
   #...
   # 很麻烦
   ```

2. 可以通过vagrant 的 Provisioning 来自动安装及配置

## 5-2 Shell inline

1. Vagrantfile 配置

   ```ruby
   config.vm.provision "shell", inline: <<-SHELL
   	# apt-get update
   	# apt-get install -y apache2
   	sudo yum install -y epel-release
   	sudo yum install -y nginx
   	sudo systemctl start nginx
   	sudo yum install -y vim
   SHELL
   ```

2. Provisioners 的三种执行的情况

   - vagrant up
   - vagrant provision
   - vagrant reload --provision

3. 使用vagrant reload --provison

   ```bash
   vagrant reload
   vagrant ssh
   ip -c a
   ```

   

## 5-3 Shell Script

1. shell 脚本 快速的在centos系统里安装docker

   setup.sh 文件内容

   ```shell
   #/bin/sh
   
   # install some tolls
   sudo yum install -y git vim gcc glibc-static telnet psmisc
   
   # install docker
   curl -fsSL get.docker.com -o get-docker.sh
   sh get-docker.sh
   
   if [  ! $(getent group docker) ]; 
   then
   	sudo groupadd docker
   else
   	echo "docker user group already exists"
   fi
   
   sudo gpasswd -a $USER docker
   sudo systemctl start docker
   
   rm -rf get-docker.sh
   ```

2. 配置Vagrantfile

   `vagrant init centos/7`

   ```
   config.vm.provision "shell", path: "./setup.sh"
   ```

3. 创建

   ```bash
   vagrant up
   vagrant status
   vagrant ssh
   # docker 需要sudo, 可能是加入组的命令没有成功成功
   sudo docker version
   
   # bug 解决方法
   sudo gpasswd -a $USER docker
   sudo systemctl restart docker
   exit
   vagrant ssh
   docker version
   ```

   

## 5-4 Ansible Script

1. Vagrangfile

   ```ruby
   config.vm.provision "ansible", playbook: "playbook.yml"
   # ansible 基本在linux和maxos系统上, windows上比较麻烦
   ```

2. playbook.yml 文件

   ```yaml
   ---
   - name: test
     hosts: all
     gather_facts: true
     become: true
     
     tasks:
       - name : Add epel-release repo
         yum:
           name: epel-release
           state: present
           
         - name: Install nginx
           yum:
             name: nginx
             state: present
             
         - name: Install Index Page
           template:
             src: index.html
             dest: /usr/share/nginx/html/index.html
             
         - name: Start NGINX
           service:
             name: nginx
             state: started
   ```

3. 创建虚拟机

   ```bash
   vagrant up
   vagrant ssh
   ip -c a
   ```

   