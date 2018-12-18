# SpringBoot Dubbo Docker Jenkins

## 开发工具

- 基于 IntelliJ idea 进行开发，下载的是旗舰版试用版，试用期为 1 月。
- 下载 docker, 直接在官网下载相关安装包就 ok

## docker 初始化

```bash
# 拉取 tomcat
docker pull chaimm/tomcat:1.1
# 运行各个服务的 tomcat 容器，这里以 gaoxi-user 作为示例
docker run --name gaoxi-user-1 -p 8082:8080 -v /usr/web/gaoxi-log:/opt/tomcat/gaoxi-log chaimm/tomcat:1.1

# 拉取 zookeeper
docker pull chaimm/zookeeper-dubbo:1.0
# 结果发现是无法通过 http://192.168.99.101:10000/dubbo-admin-2.8.4/ 访问的，原因是网络不通，网上说需要对 VirtualBox 进行相关设置才行，结果直接整坏了。
# 重启 Kitematic (Alpha) 说只能删除 VM 再重来，于是遵循指导，进入漫长的等待阶段。实际上并不是该问题
# 后来去 Issue 中找，有人发现是 dubbo-admin 没有默认启动，参见:https://github.com/bz51/SpringBoot-Dubbo-Docker-Jenkins/issues/13 
docker run --name zookeeper-debug -p 2182:2181 -p 10000:8080 chaimm/zookeeper-dubbo:1.0
# 在 bash 中运行容器
docker exec -i -t  zookeeper-debug /bin/bash
# 启动 tomacat，发现 10000 端口可以正式访问，只是提示输入密码
cd /zookeeper-3.4.10/tomcat/bin
./startup.sh
# 进入 tomcat 配置目录
cd /zookeeper-3.4.10/tomcat/apache-tomcat-8.5.23/webapps/dubbo-admin-2.8.4/WEB-INF
# 显示密码
cat dubbo.properties


# 拉取 jenkins
docker pull docker.io/jenkins/jenkins
docker run --name jenkins -p 10080:8080 docker.io/jenkins/jenkins
```