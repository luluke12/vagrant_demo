# 4 Networking

## 4-1 SSH Network 

1. virtual box里的虚拟机设置里 Network选项 NAT; 地址转发选项

2. 进入虚拟机

   ```bash
   vagrant ssh
   # 查看虚拟机的网络接口
   # 这里查的地址是内部地址, 在外部不能 ping
   ip -c a
   ```

3. hyperv的

   ```bash
   # 查看hyperv的hostname
   vagrant ssh-config
   # 进入虚拟机查看网络
   vagrant ssh
   ip -c a
   ```

4. 打开hyperv; 选择虚拟机; 右键 设置选项; Network Adapter选项是 Default Switch(网络连接里的); 在这个网段里面。

   ![1646138729838](../img/1646138729838.png)

## 4-2 Forwarder Port 端口转发

- 访问 virtual box 里的虚拟机安装的 nginx

  1. 安装 nginx

     ```bash
     vagrant ssh
     sudo yum install epel-release
     sudo yum install nginx
     sudo systemctl restart nginx
     sudo systemctl status nginx
     curl 127.0.0.1
     ip a
     ```

  2. 设置virtual box里虚拟机的网络配置: 添加一条端口转发规则

     | Name  | Protocol | Host IP   | Host Port | Guest IP | Guest Port |
     | ----- | -------- | --------- | --------- | -------- | ---------- |
     | ssh   | TCP      | 127.0.0.1 | 2222      |          | 22         |
     | nginx | TCP      | 127.0.0.1 | 8000      |          | 80         |

  3. 在外部访问 127.0.0.1:8000 即可

- 通过Vagrantfile配置自动的 端口转发规则 (只对virtual box 生效)

  1. 删除virtuai box配置好的端口转发规则

  2. 打开 Vagrantfile

     ```ruby
     config.vm.network "forwarded_port", guest: 80, host: 8080
     ```

  3. 重启虚拟机

     ```bash
     vagrant reload
     vagrant ssh
     sudo systemctl start nginx
     ```

  4. 访问 127.0.0.1:8080 即可访问nginx

## 4-3 Private Network和 静态 IP

1. Vagrantfile配置文件如下

   ```ruby
   host_list = [
       {
           :name => "host1"
           },
       {
           :name => "host2"
           }
       ]
   
   Vagrant.configure("2") do |config|
       config.vm.box = "centos/7"
       host_list.each do |item| 
           config.vm.define item[:name] do |host|
               host.vm.hostname = item[:name]
           end 
       end    
   end
   ```

2. 通过Vagrantfile分别创建virtual 和 hyperv的各自的两个虚拟机

   ```bash
   # 1 vagrant-hyperv 里的
   vagrant ssh-config
   # Host host1
   #	HostName 172.28.70.51
   #	.......
   # Host host2
   #	HostName 172.28.68.87
   #	......
   # 进入host1里面可以ping host2
   vagrant ssh host1
   ping 172.28.68.87
   
   # 2 vagrant-virtualbox 里的
   vagrant ssh-config
   # Host host1
   #	HostName 172.0.0.1
   #	.......
   # Host host2
   #	HostName 172.0.0.1
   #	......
   # 进入host1里面不可以ping host2
   vagrant ssh host1
   # 两台的etho的地址都是一样的
   ip -c a
   exit
   vagrant ssh host2
   ip -c a
   # 通过Vagrantfile的配置可以相互通信
   ```

3. 配置 Vagrantfile使virtualbox里的虚拟机可以相互通信

   ```Ruby
   host_list = [
       {
           :name => "host1",
           :eth1 => "192.169.50.10"
           },
       {
           :name => "host2",
           :eth1 => "192.169.50.11"
           }
       ]
   
   Vagrant.configure("2") do |config|
       config.vm.box = "centos/7"
       host_list.each do |item| 
           config.vm.define item[:name] do |host|
               host.vm.hostname = item[:name]
               # 这里的配置只能适用于virtualbox, 不能用于hyperv
               host.vm.network "private_network", ip: item[:eth1]
           end 
       end    
   end
   ```

4. 重启

   ```bash
   vagrant reload
   vagrant ssh host1
   ip -c a
   exit
   vagrant ssh host2
   ping 192.169.50.10
   ping 192.169.50.1
   ```

    

## 4-4 Private Network和DHCP

1. 上节使用了静态的ip, 本机使用动态的: 一般来说使用固定的更稳定些。

   ```ruby
   host_list = [
       {
           :name => "host1",
           :eth1 => "192.169.50.10"
           },
       {
           :name => "host2",
           :eth1 => "192.169.50.11"
           }
       ]
   
   Vagrant.configure("2") do |config|
       config.vm.box = "centos/7"
       host_list.each do |item| 
           config.vm.define item[:name] do |host|
               host.vm.hostname = item[:name]
               # 这里的配置只能适用于virtualbox, 不能用于hyperv
               host.vm.network "private_network", type: "dhcp"
           end 
       end    
   end
   ```

2. 重启会配置一个 172.28.128.1 的网段; 下次再重新启动, 会继续使用这个网段(vagrant源码里配置的)

   ```bush
   vagrant reload
   vagrant ssh host1
   # 172.28.128.2
   ip -c a
   vagrant ssh host2
   # 172.28.128.3
   ip -c a
   ping 172.28.128.2
   ping 172.28.128.1
   ```

   ![1646141995297](../img/1646141995297.png)

## 4-5 Public Network 详解

1. private_network 是只有本机能访问

2. public_network 可以远端访问

   ```ruby
   Vagrant.configure("2") do |config|
       config.vm.network "public_network"
   end
   ```

3. 配置Vagrantfile

   ```ruby
   host_list = [
       {
           :name => "host1"
           },
       {
           :name => "host2"
           }
       ]
   
   Vagrant.configure("2") do |config|
       config.vm.box = "centos/7"
       host_list.each do |item| 
           config.vm.define item[:name] do |host|
               host.vm.hostname = item[:name]
               host.vm.network "private_network", type: "dhcp"
               host.vm.network "public_network"
           end 
       end    
   end
   ```

4. 重启

   ```bash
   vagrant reload
   # 选择1
   # 选择1
   # bridged # (public_network)
   vagrant ssh host1
   up -c add
   # eth0: NAT 用于对外访问
   # eth1:  HostOnly (private_network)
   # eth2:  bridged (public_network)
   ```

5. eth0、eth1、eth2 和 Adapter 的链接关系

   ![1646143057403](../img/1646143057403.png)

   - eth0: 通过网络地址转换 NAT 使用windos主机的wifi地址来访问internet
   - eth1: 通过 HostOnly; 访问VB网卡, 但是不能访问wifi->internet
   - eth2: 通过bridged; 和windows、maxOs主机都是直接使用wifi分配的ip地址；可以通过wifi访问internet,以及同一wifi下的maxOs主机

6. 通过eth2来同一wifi下的网络设备 (我的wifi是192.168.0.1)

   ```bash
   ping 192.168.128.1
   ping 192.168.128.178
   ```

   

