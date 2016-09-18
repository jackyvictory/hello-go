# 脚本使用指南

备注：在脚本未由jenkins服务器托管前，还是需要人手工触发脚本执行。

## 发布流程

备注：以bigcat为例，其余应用自行脑补。

开发机部署需要满足以下条件：

1. 安装git
2. 安装ansible（pip install ansible）
3. github：有应用仓库的读写权限、有脚本仓库的读权限
4. Go语言环境
5. 应用代码的工作目录在$GOPATH下
6. （建议）vendor目录下有所有的第三方依赖包（备注：使用工具godeps。应用安装依赖，通过go get的方式不稳定。）


```
1. checkout主干分支（备注：保持工作区干净）

git fetch CIL/develop
git checkout CIL/develop

2. 打包
./package.sh

3. 下载发布脚本
git clone git@github.com:CardInfoLink/rainbow.git --branch auto --single-branch supermario

4. 正式发布，按照提示进行（备注：ansible-playbook脚本是否需要用.sh封装起来）
cd supermario
ansible-playbook -i hosts.dev.deploy app_deploy.yml --extra-vars "@vars/bigcat_dev.yml" --ask-sudo-pass

5. 回退（可选）
ansible-playbook -i hosts.dev.deploy app_rollback.yml --extra-vars "@vars/bigcat_dev.yml" --ask-sudo-pass

6. 发布OK后，生成changelog、打tag并上传，在release内粘贴changelog
./get_changelog.sh
git tag v1.0.x  &&  git push CIL --tags

```

## 服务器配置

TODO：