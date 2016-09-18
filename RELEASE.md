# 自动化部署方案上线步骤

## 1. Pre Release （上线前的操作步骤）

### 1.1 云收银线上nginx用户加入到sudo组

```
ip：121.40.167.112
username: nginx
```

两件事情：

1. `nginx`用户在sudo组内
2. 保证`nginx`用户有读写`/opt/nginx/conf`目录的权限



## 2. Release （nginx配置、网络配置）

### 2.1 将线下nginx的流量转到线上nginx，而不是直接转给app服务器。

#### 2.1.1 线下nginx配置（研发）

```
TODO: 线下配置没权限看

```

tcp转发 待验证（@陈诚）
https转发 待验证 (@john)

#### 2.1.2 线下防火墙配置 （数据中心）
保证线下出口网络能到达

验证：

```
telnet showmoney.cn 80
telnet showmoney.cn 443
telnet showmoney.cn 6000
telnet showmoney.cn 6001
```

### 2.2 线上nginx upstream pool分离 （研发）
分离后的配置已经由github管理起来了。往后发布时候的app1, app2应用切分，全由自动部署脚本来完成。

1. 同步配置文件到服务器

```
git clone git@github.com:CardInfoLink/rainbow.git --branch auto --single-branch supermario

cd nginx_conf/sites
rsync -rcv  --progress ./ nginx@121.40.167.112:/opt/nginx/conf/sites

```
2. reload nginx并观察

```
cd /opt/nginx
sudo sbin/nginx -s reload

```

## 3. Post Release（发布脚本的生产验证）
发布脚本可以在例行变更中校验，包括：

* version.go的废除
* changeLog.md的废除
* 打包脚本验证
* 重启脚本验证（日志的目录保持不变）


**发布脚本的验证，每个应用的脚本负责人需要参与。**