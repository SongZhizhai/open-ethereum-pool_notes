# open-ethereum-pool以太坊矿池-环境安装

## 安装Geth

```shell
//安装parity
wget http://d1h4xl4cr1h0mo.cloudfront.net/v1.9.4/x86_64-unknown-linux-gnu/parity_1.9.4_ubuntu_amd64.deb
dpkg -i parity_1.9.4_ubuntu_amd64.deb
parity --chain=ropsten --geth --rpccorsdomain=all

//安装Geth
add-apt-repository -y ppa:ethereum/ethereum
apt-get update
apt-get -y install ethereum
//启动Geth连接parity
geth attach http://127.0.0.1:8545

//区块同步进展
> eth.syncing

//创建账户
> personal.newAccount()
Passphrase: 
Repeat passphrase: 
"0xa5230ee1a682bf6a65245d2be3ff1b7b56b62ec6"

//查账户余额
> eth.getBalance("0xa5230ee1a682bf6a65245d2be3ff1b7b56b62ec6")
```

## open-ethereum-pool矿池安装

```shell
//go >= 1.9
//geth or parity
//redis-server >= 2.8.0
//nodejs >= 4 LTS
//nginx

//安装go
wget https://dl.google.com/go/go1.9.4.linux-amd64.tar.gz
tar -zxvf go1.9.4.linux-amd64.tar.gz
mv go /usr/local/
mkdir -p /work/go
vim /etc/profile
	export GOPATH=/work/go
	export PATH=$PATH:/usr/local/go/bin:$GOPATH/bin
source /etc/profile
go version

//安装redis-server
apt-get -y install redis-server

//安装nodejs
curl -sL https://deb.nodesource.com/setup_8.x | sudo -E bash -
apt-get install -y nodejs
npm config set registry https://registry.npm.taobao.org

//安装nginx
apt-get -y install nginx

//安装open-ethereum-pool
git config --global http.https://gopkg.in.followRedirects true
git clone https://github.com/sammy007/open-ethereum-pool.git
cd open-ethereum-pool
make

//运行open-ethereum-pool
cd /work/open-ethereum-pool/
cp config.example.json config.json
./build/bin/open-ethereum-pool config.json
```

## 运行www

```shell
//修改www/config/environment.js
vim www/config/environment.js

ApiUrl: '//10.18.216.180/',

//编译www
cd www
npm install -g ember-cli@2.9.1
npm install -g bower
sudo npm install
bower install --allow-root
./build.sh

//配置nginx
vim /etc/nginx/sites-available/default

upstream api {
	server 127.0.0.1:8080;
}
root /work/open-ethereum-pool/www/dist;
location /api {
	proxy_pass http://api;
}

//重启nginx
service nginx restart

//访问地址：http://10.18.216.180/
//API地址
http://10.18.216.180/api/stats
http://10.18.216.180/api/miners
http://10.18.216.180/api/blocks
http://10.18.216.180/api/payments
http://10.18.216.180/api/accounts/{login:0x[0-9a-fA-F]{40}}
```

## 使用ethminer测试open-ethereum-pool

```shell
//下载地址https://github.com/ethereum-mining/ethminer/releases/download/v0.14.0.dev2/ethminer-0.14.0.dev2-Windows.zip
//解压至C:\ethminer
//启动
cd c:\ethminer\bin
ethminer.exe -G -F http://10.18.216.180:8888/0xc1d0932D18a4Ec35728b7fF02288dF745D1f4F40
```

## 参考文档

* [Installing Go Ethereum](https://ethereum.github.io/go-ethereum/install/#install-on-ubuntu-via-ppas)
* [以太坊客户端Geth命令用法-参数详解](http://blog.csdn.net/xilibi2003/article/details/78662000)
* [JavaScript Runtime Environment](https://ethereum.gitbooks.io/frontier-guide/content/jsre.html)
* [Ropsten Testnet Ethereum BlockChain Explorer and Search](https://ropsten.etherscan.io/)
* [Installation Instructions for Ubuntu](https://github.com/ethereum/go-ethereum/wiki/Installation-Instructions-for-Ubuntu)
* [Open Source Ethereum Mining Pool](https://github.com/sammy007/open-ethereum-pool)
* [ethminer](https://github.com/ethereum-mining/ethminer)

