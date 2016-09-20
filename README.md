因为rainbow比较大，clone的时候只下载一个分支：

git clone git@github.com:CardInfoLink/rainbow.git --branch auto --single-branch supermario


准备工具

1. vagrant 用于虚拟机,已经准备了Vagrantfile
2. ansible 部署用 brew install ansible

启动虚拟机

```
vagrant up

```

部署机与应用机器网络互通

```
默认分配sudo账户: vagrant/vagrant

app1: 192.168.99.20
app2: 192.168.99.30

```


多环境打包

```
./package.sh

```

打包结果, ls -1 dist/
```
app20160908165417.develop.tar.gz
app20160908165417.product.tar.gz
app20160908165417.testing.tar.gz

```

在干净机器上构建magisett环境, 脚本中为后端服务创建了quick用户,密码默认为test

以magisett为例,其它纯后端服务类似.

```

ansible-playbook -i hosts.dev.setup app_setup.yml --extra-vars "@vars/magisett_dev.yml" --ask-sudo-pass

(passwd: vagrant)

ansible-playbook -i hosts.dev.deploy app_deploy.yml --extra-vars "@vars/magisett_dev.yml"

(单台验证有问题/全量部署后有问题,可以选择回退.程序本身会做选择性回退,通过比较服务器app版本与本地记录的之前版本,如果相同,就跳过回退操作)

ansible-playbook -i hosts.dev.deploy app_rollback.yml --extra-vars "@vars/magisett_dev.yml"

本地就能访问magisett服务了

```
http://192.168.99.20:6910/settle/run
http://192.168.99.30:6910/settle/run
```
