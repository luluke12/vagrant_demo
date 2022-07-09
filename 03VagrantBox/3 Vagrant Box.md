# 3 Vagrant Box

## 3-1 Vagrant Box 的基本操作

1. boxes 位置: 用户/.vagrant.d/boxes

   ```bash
   $ cd ~/.vagrang.d/boxes
   $ tree /F
   ```
   
2. `vagrant box`命令

   ```bash
   # vagrant box 的查看、下载、删除
   $ vagrant box list
   $ vagrant box add
   $ vagrant box remove
   
   # 下载generic/ubuntu2004版本的
   $ vagrant box add generic/ubuntu2004 
   # --provider 和 --box-version 参数
   $ vagrant box add generic/ubuntu2004 --provider=hyperv --box-version=3.1.22 
   
   # 删除相应版本和provider的box
   $ vagrant box remove generic/ubuntu2004 --provider=hyperv --box-version=3.1.22
   # 删除相应provider的所有box使用: --all参数
   $vagrant box remove generic/ubuntu2004 --provider=hyperv --all
   ```
   
   

## 3-2 创建一个 VirtualBox 版的 Base Box

1. 创建/开启并进入相应的虚拟机

   ```bash
   # 打包box 使用 vagrant package 命令
   # 条件1 : 安装vagrant-vbguest 
   $vagrant plugin install vagrant-vbguest --plugin-version 0.21
   # 条件2:  ~/.ssh/authorized_keys 里要包含vagrant insecure public key
   # vim 编辑authorized_keys文件,只保留 insecure public key(insecure_private_key在~/.vagrant.d/目录下)
   $ cd ~/.ssh/
   $ more authorized_keys
   $ vim authorized_keys
   # 条件3: virtualbox软件了对应的虚拟机设置里 Network 第一个网卡的 NAT
   
   
   # 打包命令, 会在当前目录生成一个 package.box的文件
   $ vagrant halt
   $ vagrant package --base <vb_name>(virtualbox软件里复制)
   
   # 把当前的package.box添加到 boxes 文件夹里
   $ vagrant box add --name centos-vim --provider=virtualbox ./package.box
   $ vagrant box list
   
   # 测试
   # 初始化、安装、进入
   $ vagrant init centos-vim
   $ vagrant up
   $ vagrant ssh
   # 删掉这个box
   $ vagrant destroy =f
   $ vagrant box list
   $ vagrant box remove centos-vim
   ```

## 3-3 把 Box 上传到 Vagrant Cloud

1. 登录 Vagrant Cloud 官网

2. Dashboard 菜单下 Create a new Vagrant Box

3. 一步步操作

   ```bash
   # 生成SHA2256的哈希值
   $ Get-FileHas ./package.box
   ```

4. 发布:  Release versin

5. 使用

   ```bash
   $ vagrant init lulukee/centos-vim
   $ vagrant up
   $ vagrant ssh
   ```
   

## 3-4 创建一个 Hyper-V 版的 Base Box 并上传

1. 创建一个hyperv的box

   ```bash
   # 1 vim 编辑authorized_keys文件,只保留 insecure public key
   # 2 hyperv 只能通过软件里导出 -> Export -> 选择路径 -> 进入导入的文件下 
   # -> 只保留Virtual Hard Disks和Virtual Machines -> 创建一个metadata.json文件 -> vscode打开metadata.json文件
   {"provider":"hyperv"}
   
   # 3 命令行里打包
   $ cd <hyperv导出的虚拟机的目录下>
   $ tar zcvf ../demo.box ./*
   cd ..
   
   # 给 demo.box 生成hash值
   $ Get-FileHash ./demo.box
   
   ```
   
2. 在 Vagrant Cloud 网站里添加 Add a provider 

3. 测试

   ```bash
   $ vagrant init lulukee\centos-vim 
   $ vagrant up --provider==hyperv
   $ vagrant ssh
   #  vagrant ssh 权限问题 解决方法
   $ vagrant ssh-config
   # 使用insecure_private_key覆盖private_key
   ```
   
   