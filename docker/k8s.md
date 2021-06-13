## Ubuntu 20.04 k8s 1.18.3集群
搭建环境及所用版本：
docker：19.03.11；
Kubernetes：1.18.3

+ 1、机器配置镜像源 
 ubuntu对应镜像源：
https://mirrors.tuna.tsinghua.edu.cn/help/ubuntu/

```
# 备份
sudo cp /etc/apt/sources.list /etc/apt/sources.list_backup

#修改为指定镜像源
sudo vim /etc/apt/sources.list

#更新
sudo apt-get update
```
