# 3 Vagrant Box

## 3-1 Vagrant Box 的基本操作

1. boxes 位置: 用户/.vagrant.d/boxes

   ```bash
   cd ~
   cd ./.vagrant
   cd boxes/
   ls
   tree /F
   ```

2. `vagrant box`命令

   ```bash
   vagrant box list # 查看下载好的vagrant box
   # 下载generic/ubuntu2004版本的
   # 输入数字选择对应的版本(hyperv virtualbox ....)
   vagrant box add generic/ubuntu2004 
   vagrant box list
   # 下载hyperv的 3.1.22的generic/ubuntu2004
   vagrant box add generic/ubuntu2004 --provider=hyperv --box-version=3.1.22 
   
   # 删除相应版本和provider的box
   vagrant box remove generic/ubuntu2004 --provider=hyperv --box-version=3.1.22
   # 删除相应provider的所有box
   vagrant box remove generic/ubuntu2004 --provider=hyperv --all
   ```

   

## 3-2 创建一个 VirtualBox 版的 Base Box

1. 创建/开启并进入相应的虚拟机

   ```bash
   cd E:/linux/vagrant-virtualbox
   vagrant status
   vagrant up
   vagrant ssh
   
   # 虚拟机里下载一些软件
   sodu yum install -y vim
   # 打包这个box
   # 条件1 : 安装vagrant-vbguest 
   exit
   vagrant status
   vagrant plugin list
   vagrant plugin install vagrant-vbguest --plugin-version 0.21
   vagrant ssh
   cd .ssh/
   # 条件2:  .ssh文件里要包含vagrant insecure public key
   ls
   more authorized_keys
   # vim 编辑authorized_keys文件,只保留 insecure public key
   vim authorized_keys
   # 条件3: virtualbox软件了对应的虚拟机设置里 Network 第一个网卡的 NAT
   # 开始打包: 关机
   vagrant halt
   # 打包命令, 会在当前目录打包一个 package.box的文件
   vagrant package --base 虚拟机的名字(virtualbox软件里复制)
   # 把这个box添加到 boxes 文件夹里
   vagrant box add --name centos-vim --provider=virtualbox ./package.box
   vagrant box list
   # 测试
   vagrant destroy -f 
   ls
   Remove-Item ./Vagrantfile
   Remove-Item ./.vagrant/
   # 初始化、安装、进入
   vagrant init centos-vim
   vagrant up
   vagrant ssh
   # 删掉这个box
   vagrant destroy =f
   vagrant box list
   vagrant box remove centos-vim
   ```

## 3-3 把 Box 上传到 Vagrant Cloud

1. 登录 Vagrant Cloud 官网

2. Dashboard 菜单下 Create a new Vagrant Box

3. 一步步操作

   ```
   # 生成SHA2256的哈希值
   Get-FileHas ./package.box
   ```

4. 发布: Release versin

5. 使用

   ```bash
   Remove-Item ./Vagrantfile
   Remove-Item ./.vagrant/
   Remove-Item ./package.box
   
   vagrant init lulukee/centos-vim
   vagrant up
   vagrant ssh
   ```

   

## 3-4 创建一个 Hyper-V 版的 Base Box 并上传

1. 创建一个hyperv的box

   ```bash
   cd E:/linux/vagran-hyperv
   vagrant init centos/7
   vagrant up --provider=hyperv
   vagran ssh
   cd .ssh/
   ls
   more authorized_keys
   sudo yum install -y vim
   # 1 vim 编辑authorized_keys文件,只保留 insecure public key
   vim authorized_keys
   vagant halt
   # 2 hyperv 只能通过软件里导出 -> Export选型 -> 选择路径 -> 进入导入的文件下 
   # -> 只保留Virtual Hard Disks和Virtual Machines -> 创建一个metadata.json文件 -> vscode打开metadata.json文件
   {"provider":"hyperv"}
   # 3 命令行里打包
   cd hyperv导出的虚拟机的目录下
   tar zcvf ../demo.box ./*
   cd ..
   # 给 demo.box 生成hash值
   Get-FileHash ./demo.box
   
   ```

2. 在 Vagrant Cloud 网站里添加 Add a provider

3. 测试

   ```bash
   cd E:/linux/vagran-hyperv/
   vagrant destroy -f
   ls
   Remove-Item *
   ls
   vagrant init lulukee\centos-vim 
   vagrant up --provider==hyperv
   vagrant status
    vagrant ssh # 可能会失败 权限问题 
    vagrant ssh-config
    ssh -i 'C:\Users\Administrator\.vagrant.d\insecure_private_key' vagrant@HostName
    exit
   #  vagrant ssh 权限问题 解决方法
   vagrant ssh-config
   # 把IdentityFile文件用用户目录下的.vagrant.d里的insecure_private_key覆盖
   ```

   