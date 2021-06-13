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

+ 2 docker安装参照：https://developer.aliyun.com/article/763005?spm=a2c6h.14164896.0.0.74ef55aeCe1aRt
```
# step 1: 安装必要的一些系统工具
sudo apt-get update
sudo apt-get -y install apt-transport-https ca-certificates curl software-properties-common
# step 2: 安装GPG证书
curl -fsSL https://mirrors.aliyun.com/docker-ce/linux/ubuntu/gpg | sudo apt-key add -

# Step 3: 写入软件源信息--
sudo add-apt-repository "deb [arch=amd64] https://mirrors.aliyun.com/docker-ce/linux/ubuntu bionic stable" 

#查找可安装Docker-CE的版本:
apt-cache madison docker-ce

# Step 2: 安装指定版本的Docker-CE:
sudo apt-get -y install docker-ce=5:19.03.11~3-0~ubuntu-bionic

# 锁定版本
sudo apt-mark hold docker-ce=5:19.03.11~3-0~ubuntu-bionic 

# 相关命令
# 启动docker,设置开机自启动
sudo systemctl enable docker && sudo systemctl start docker

# 配置docker 镜像源
vi /etc/docker/daemon.json
{
    "registry-mirrors": ["https://g2djyyu3.mirror.aliyuncs.com"],
    "exec-opts": [ "native.cgroupdriver=systemd" ]
}  

# 重启docker
sudo systemctl restart docker

# 查看docker状态
systemctl status docker

# 详细日志
journalctl -u docker.server


# 将当前登录用户user_name加入docker用户组中，便于用户使用
$ sudo usermod -aG docker user_name
```

