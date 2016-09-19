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

## 4. 完整执行一次自动部署
#### 4.1 在第一次`[nginx-exclude : pause]`的时候，验证服务可以正常使用，并且只有`app2`提供服务。
#### 4.2 回车继续，在接着的`[deploy-app : pause]`的时候，验证`app1`的服务可以正常使用。
#### 4.3 回车继续，在`[nginx-full : pause]`的时候，验证服务可以正常使用，并且`app1`和`app2`都提供服务。
#### 4.4 回车继续，在第二次`[nginx-exclude : pause]`的时候，验证服务可以正常使用，并且只有`app1`提供服务。
#### 4.5 回车继续，在接着的`[deploy-app : pause]`的时候，验证`app2`的服务可以正常使用。
#### 4.6 回车继续，在`[nginx-full : pause]`的时候，验证服务可以正常使用，并且`app1`和`app2`都提供服务。
#### 4.7 部署完成。