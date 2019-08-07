### 搭建超级账本环境

环境 Ubuntu16.04

时间 2019年3月6日13:48:33

#### 1.安装curl,node,npm

`sudo apt-get install curl`

#### 2.安装node,np,m

直接安装并升级最新的node和npm

```
sudo apt-get install nodejs npm && \ 
sudo npm install n -g && \
sudo n stable && \
sudo npm install npm -g

sudo apt-get install nodejs npm && sudo npm install n -g && sudo n stable && sudo npm install npm -g
```

#### 3.安装python

Ubuntu16.04 默认的python版本 python3.5.1  python 2.7 . 满足需求。

`sudo apt-get install python`

#### 4.安装go

- 去[官网](https://golang.org/dl/)找最新版本的下载链接。（2019/7/9  1.12最新）   `

`wget https://dl.google.com/go/go1.12.7.linux-amd64.tar.gz`

- 解压到 `/user/local`目录下

`sudo tar -xzf go1.12.linux-amd64.tar.gz -C /usr/local `

- 配置全局变量

`sudo vi ~/.bashrc`

在最后两行加上

```bash
export GOROOT=/usr/local/go
export GOPATH=~/go
export PATH=$GOROOT/bin:$PATH  
```

生效： `source ~/.bashr   `

- 验证

```
justin@ubuntu:~$ go version
warning: GOPATH set to GOROOT (/usr/local/go) has no effect
go version go1.12 linux/amd64
```

出现了警告，配置PATH, 终端输入：

```bash
export GOROOT=/usr/local/go  
export GOPATH=$PATH:$GOROOT/bin  

```

#### 5.安装docker

- 简易安装

`sudo apt-get install docker docker.io`

`sudo apt-get install docker-engine`

- 官方推荐安装

```
sudo apt-get install curl -y
sudo curl -sSL https://get.docker.com/ | sh
```

- 启动docker

`sudo systemctl start docker`

- 验证docker

`sudo docker version` 出现连接失败的情况，请使用sudo

#### 6.安装Docker-Compose

 通过pip简易安装：`sudo apt-get install python-pip`

直接安装：`sudo pip install docker-compose `

验证 `docker-compose version`

#### 7.Fabric源码下载

创建目录 ``mkdir -p ~/go/src/github.com/hyperledger && cd ~/go/src/github.com/hyperledger  `  `

下载完整版的 `git clone https://github.com/hyperledger/fabric.git ` 

只是纯粹的搭建环境下载教程版代码： `git clone https://github.com/hyperledger/fabric-samples.git`

#### 8.下载Fabric Docker镜像

使用的教程版，进行环境搭建

版本选择：先根据`git tag`  找最新发布版， 再切换`git checkout v1.4.0`

`cd ~/path/to/fabric-samples/scripts`

运行 `sudo ./bootstrap.sh`

检查是否安装成功：运行 `sudo docker images`能看到刚下载的镜像。

```
justin@ubuntu:~/go/src/github.com/fabric-samples/bin$ sudo docker images
[sudo] password for justin:
REPOSITORY                     TAG                 IMAGE ID            CREATED             SIZE
hyperledger/fabric-tools       1.4.0               0a44f4261a55        7 weeks ago         1.56GB
hyperledger/fabric-tools       latest              0a44f4261a55        7 weeks ago         1.56GB
hyperledger/fabric-ccenv       1.4.0               5b31d55f5f3a        7 weeks ago         1.43GB
hyperledger/fabric-ccenv       latest              5b31d55f5f3a        7 weeks ago         1.43GB
hyperledger/fabric-orderer     1.4.0               54f372205580        7 weeks ago         150MB
hyperledger/fabric-orderer     latest              54f372205580        7 weeks ago         150MB
hyperledger/fabric-peer        1.4.0               304fac59b501        7 weeks ago         157MB
hyperledger/fabric-peer        latest              304fac59b501        7 weeks ago         157MB
hyperledger/fabric-ca          1.4.0               1a804ab74f58        7 weeks ago         244MB
hyperledger/fabric-ca          latest              1a804ab74f58        7 weeks ago         244MB
hyperledger/fabric-zookeeper   0.4.14              d36da0db87a4        4 months ago        1.43GB
hyperledger/fabric-zookeeper   latest              d36da0db87a4        4 months ago        1.43GB
hyperledger/fabric-kafka       0.4.14              a3b095201c66        4 months ago        1.44GB
hyperledger/fabric-kafka       latest              a3b095201c66        4 months ago        1.44GB
hyperledger/fabric-couchdb     0.4.14              f14f97292b4c        4 months ago        1.5GB
hyperledger/fabric-couchdb     latest              f14f97292b4c        4 months ago        1.5GB
```

在`fabric-samples/bin`目录下有生成的可执行文件 

```
justin@ubuntu:~/go/src/github.com/fabric-samples/bin$ ls
configtxgen  configtxlator  cryptogen  discover  fabric-ca-client  get-docker-images.sh  idemixgen  orderer  peer
```

