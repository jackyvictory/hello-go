# 自动化部署方案Test Case

## 1. 模拟线下nginx`（tcp服务）`，直连线上nginx
#### 1.1 创建虚拟机，安装nginx，后面称之为虚拟线下nginx。
#### 1.2 将生产线下nginx的`conf/`中内容拷贝到虚拟线下nginx中。
#### 1.3 修改`conf/sites/quickpay.stream.conf`，将`upstream quickpay_tcp_server`中两个server去掉一个，并改为`test.quick.ipay.so:6000`
#### 1.4 `quickpay_tcp_server_gbk`同上。
#### 1.5 重新加载nginx配置。
#### 1.6 启动云收银桌面版，将请求IP和端口改为虚拟线下nginx的IP和tcp监听端口。
#### 1.7 执行一笔下单交易。
#### 1.8 用tcpdump -i eth0 -vnn dst host <IP> and dst port <PORT> 看nginx是否将请求转发到目的IP端口。
```
    10.30.1.38.58781 > 10.30.1.99.6000: Flags [P.], cksum 0xc0bf (correct), seq 0:269, ack 1, win 16425, length 269
    10.30.1.99.54742 > 121.40.86.222.6000: Flags [P.], cksum 0xdcba (incorrect -> 0x4808), seq 0:269, ack 1, win 229, options [nop,nop,TS val 1156372 ecr 2073314995], length 269
    10.30.1.99是虚拟线下nginx，10.30.1.38.58781是扫码客户端，121.40.86.222.6000是线上nginx。
```

#### 1.9 若1.8中显示nginx执行了转发，并且交易成功，说明测试通过。

## 2. 模拟线下nginx`（http服务）`，直连线上nginx
#### 2.1 修改`conf/sites/quickpay.http.conf`，将`proxy_pass`都改为`https://showmoney.cn`
#### 2.2 重新加载nginx配置。
#### 2.3 在浏览器中用虚拟nginx的IP访问云收银管理平台。能正常访问说明测试通过。
#### 2.4 在内部测试中执行预下单，能生成二维码说明测试通过。

## 3. 模拟线下nginx`（移动端http服务）`，直连线上nginx
#### 3.1 若配置文件在2中已经修改完成，则生成一个往线下走的apk包。
#### 3.2 用这个apk测试登录，如果能成功登录说明测试通过。

## 4. 本地内网虚拟机完整执行一次`quickpay`的自动部署测试
##### 4.1 在第一次`[nginx-exclude : pause]`的时候，验证服务可以正常使用，并且只有`app2`提供服务。
* 查看conf/sites/quickpay_tcp_server.upstream和conf/sites/quickpay_http_server.upstream是否只有app2
* curl http://192.168.99.10/scanpay/		验证http，192.168.99.10是nginx机器
* tail -f -n100 logs/scanpay.log 			如果只有如下192.168.99.30的日志，说明nginx切http成功
* 2016-09-20T06:22:38+00:00 192.168.99.10 GET /scanpay/ HTTP/1.1 192.168.99.10 404 0.001 222 19 192.168.99.30:6800 404 0.001 "curl/7.35.0"
* telnet 192.168.99.10 6000
* tail -f -n100 logs/error.log 				如果只有如下192.168.99.30的日志，说明nginx切tcp成功
* 2016/09/20 06:23:05 [error] 30862#30862: *95 recv() failed (104: Connection reset by peer) while proxying connection, client: 192.168.99.1, server: 0.0.0.0:6000, upstream: "192.168.99.30:6600", bytes from/to client:5/0, bytes from/to upstream:0/5

##### 4.2 回车继续，在接着的`[deploy-app : pause]`的时候，验证`app1`的服务可以正常使用。
* curl http://192.168.99.20:6800/scanpay/		返回404 page not found说明http服务正常启动
* telnet 192.168.99.20 6600						随便输入几个英文字母，不要出现数字
* tail -f -n100 logs/quickpay.log 				出现如下error log说明tcp服务正常启动
* 2016-09-20 06:39:38 error scanpay/tcpHandler.go:96 strconv.ParseInt: parsing "aaa\r": invalid syntax

##### 4.3 回车继续，在`[nginx-full : pause]`的时候，验证服务可以正常使用，并且`app1`和`app2`都提供服务。
* 类似4.1，不同的是需要观察日志中是否同时出现192.168.99.20和192.168.99.30

##### 4.4 回车继续，在第二次`[nginx-exclude : pause]`的时候，验证服务可以正常使用，并且只有`app1`提供服务。
* 类似4.1，不同的是需要观察日志中是否只出现192.168.99.20

##### 4.5 回车继续，在接着的`[deploy-app : pause]`的时候，验证`app2`的服务可以正常使用。
* 类似4.2，不同的是需要观察192.168.99.30

##### 4.6 回车继续，在`[nginx-full : pause]`的时候，验证服务可以正常使用，并且`app1`和`app2`都提供服务。
* 同4.3

##### 4.7 部署测试完成。