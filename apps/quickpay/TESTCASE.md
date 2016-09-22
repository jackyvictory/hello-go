# quickpay自动化部署方案Test Case

## 1. 本地内网虚拟机完整执行一次`quickpay`的自动部署测试
##### 1.1 在第一次`[nginx-exclude : pause]`的时候，验证服务可以正常使用，并且只有`app2`提供服务。
* 查看conf/sites/quickpay_tcp_server.upstream和conf/sites/quickpay_http_server.upstream是否只有app2
* curl http://192.168.99.10/scanpay/		验证http，192.168.99.10是nginx机器
* tail -f -n100 logs/scanpay.log 			如果只有如下192.168.99.30的日志，说明nginx切http成功
* 2016-09-20T06:22:38+00:00 192.168.99.10 GET /scanpay/ HTTP/1.1 192.168.99.10 404 0.001 222 19 192.168.99.30:6800 404 0.001 "curl/7.35.0"
* telnet 192.168.99.10 6000
* tail -f -n100 logs/error.log 				如果只有如下192.168.99.30的日志，说明nginx切tcp成功
* 2016/09/20 06:23:05 [error] 30862#30862: *95 recv() failed (104: Connection reset by peer) while proxying connection, client: 192.168.99.1, server: 0.0.0.0:6000, upstream: "192.168.99.30:6600", bytes from/to client:5/0, bytes from/to upstream:0/5

##### 1.2 回车继续，在接着的`[deploy-app : pause]`的时候，验证`app1`的服务可以正常使用。
* curl http://192.168.99.20:6800/scanpay/		返回404 page not found说明http服务正常启动
* telnet 192.168.99.20 6600						随便输入几个英文字母，不要出现数字
* tail -f -n100 logs/quickpay.log 				出现如下error log说明tcp服务正常启动
* 2016-09-20 06:39:38 error scanpay/tcpHandler.go:96 strconv.ParseInt: parsing "aaa\r": invalid syntax

##### 1.3 回车继续，在`[nginx-full : pause]`的时候，验证服务可以正常使用，并且`app1`和`app2`都提供服务。
* 类似1.1，不同的是需要观察日志中是否同时出现192.168.99.20和192.168.99.30

##### 1.4 回车继续，在第二次`[nginx-exclude : pause]`的时候，验证服务可以正常使用，并且只有`app1`提供服务。
* 类似1.1，不同的是需要观察日志中是否只出现192.168.99.20

##### 1.5 回车继续，在接着的`[deploy-app : pause]`的时候，验证`app2`的服务可以正常使用。
* 类似1.2，不同的是需要观察192.168.99.30

##### 1.6 回车继续，在`[nginx-full : pause]`的时候，验证服务可以正常使用，并且`app1`和`app2`都提供服务。
* 同1.3

##### 1.7 部署测试完成。