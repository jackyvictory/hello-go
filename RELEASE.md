# 自动化部署方案上线步骤

## 1. Pre Release （上线准备工作）
### 1.1 线下防火墙配置 （数据中心协助）
保证线下出口网络能到达。打开到showmoney.cn（121.40.167.112）的以下端口：
```
80
443
6000
6001
```

验证：

```
telnet showmoney.cn 80
telnet showmoney.cn 443
telnet showmoney.cn 6000
telnet showmoney.cn 6001
```

###  1.2 hosts更新（数据中心协助）
在/etc/hosts添加记录：

```
121.40.167.112 showmoney.cn

```

## 2. Release （nginx配置）

### 2.1 线下nginx的流量转到线上nginx，而不是直接转给app服务器。

```
0. 线下nginx: 10.99.1.67

1. 上传替换以下文件到/opt/nginx/conf/sites目录：

nginx_conf_offline/sites/quickpay.http.conf
nginx_conf_offline/sites/quickpay.stream.conf

2. reload nginx （TODO：是否是管理员权限）

cd /opt/nginx
sudo sbin/nginx -s reload

```

### 2.2 线上nginx用户权限控制

```
0. 服务器地址：121.40.167.112

1. 先停nginx（root执行）
cd /opt/nginx
sbin/nginx -s stop

2. 改/opt/nginx的owner（root执行）
chown nginx:nginx -R /opt/nginx

3. 使得nginx在`nginx`用户下具有绑定1024以下端口的权限（root执行）
setcap cap_net_bind_service=ep /opt/nginx/sbin/nginx

4. nginx用户下执行
cd /opt/nginx
sbin/nginx

```

### 2.3 线上nginx upstream pool分离
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
sbin/nginx -s reload

```

## 3. Post Release（发布脚本的生产验证）
发布脚本可以在例行变更中校验，包括：

* version.go的废除
* changeLog.md的废除
* 打包脚本验证
* 重启脚本验证（日志的目录保持不变）


**发布脚本的验证，每个应用的脚本负责人需要参与。**