
### 脚本使用

在干净机器上构建quickpay环境。

备注：脚本中为nginx服务器创建了nginx用户，默认密码nginx123456，为web服务器创建了quick用户，密码默认为test。


#### 以管理员权限配置服务器

```
ansible-playbook -i apps/quickpay/hosts/setup.dev app_setup.yml --extra-vars "@apps/quickpay/vars/dev.yml" --ask-sudo-pass

(passwd: vagrant)

```

#### 以应用权限发布应用
##### Dev

```
ansible-playbook -i apps/quickpay/hosts/deploy.dev app_deploy.yml --extra-vars "@apps/quickpay/vars/dev.yml"

（本地就能访问quickpay服务了 http://192.168.99.10/scanpay/）

(单台验证有问题/全量部署后有问题,可以选择回退.程序本身会做选择性回退,通过比较服务器app版本与本地记录的之前版本,如果相同,就跳过回退操作)

ansible-playbook -i apps/quickpay/hosts/deploy.dev app_rollback.yml --extra-vars "@apps/quickpay/vars/dev.yml"

```

##### QA

```
ansible-playbook -i apps/quickpay/hosts/deploy.qa app_deploy.yml --extra-vars "@apps/quickpay/vars/qa.yml"

ansible-playbook -i apps/quickpay/hosts/deploy.qa app_rollback.yml --extra-vars "@apps/quickpay/vars/qa.yml"

```

##### Product
```
ansible-playbook -i apps/quickpay/hosts/deploy.product app_deploy.yml --extra-vars "@apps/quickpay/vars/product.yml"

ansible-playbook -i apps/quickpay/hosts/deploy.product app_rollback.yml --extra-vars "@apps/quickpay/vars/product.yml"

```



### 本地开发环境模拟

准备工具

1. vagrant 用于虚拟机,已经准备了Vagrantfile
2. ansible 部署用 pip install ansible

启动虚拟机

```
vagrant up

```

部署机与应用机器网络互通

```
默认分配sudo账户: vagrant/vagrant

nginx: 192.168.99.10
app1: 192.168.99.20
app2: 192.168.99.30

```

### 应用打包

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

因为rainbow比较大，clone的时候只下载一个分支：

git clone git@github.com:CardInfoLink/rainbow.git --branch auto --single-branch supermario
