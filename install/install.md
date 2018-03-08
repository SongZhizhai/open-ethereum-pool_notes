# open-ethereum-pool以太坊矿池-环境安装

## 安装Geth

```shell
//安装parity
cd /tmp/
wget http://d1h4xl4cr1h0mo.cloudfront.net/v1.8.11/x86_64-unknown-linux-gnu/parity_1.8.11_ubuntu_amd64.deb
dpkg -i parity_1.8.11_ubuntu_amd64.deb

//安装screen
apt-get update
apt-get -y install screen

//启动parity
screen parity --base-path=/root/.ethereum --chain ropsten --jsonrpc-interface local --jsonrpc-hosts all --jsonrpc-apis all --no-ws --no-dapps --author 0xc1d0932D18a4Ec35728b7fF02288dF745D1f4F40
//暂时离开 按住Ctrl，依次再按a,d
//恢复screen会话
screen -ls
screen -r 17172

//安装Geth
apt-get -y install software-properties-common
add-apt-repository -y ppa:ethereum/ethereum
apt-get update
apt-get -y install ethereum

//启动Geth连接parity
geth attach http://127.0.0.1:8545

//区块同步进展
> eth.syncing

//查账户余额
> eth.getBalance("0xc1d0932D18a4Ec35728b7fF02288dF745D1f4F40")

//getBlock
> eth.getBlock('pending')

//getWork
> eth.getWork()
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
ln -s /usr/local/go/bin/go /usr/bin/go
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
cd /root/
git config --global http.https://gopkg.in.followRedirects true
git clone https://github.com/sammy007/open-ethereum-pool.git
cd open-ethereum-pool
make

//运行open-ethereum-pool
cd /root/open-ethereum-pool/
screen ./build/bin/open-ethereum-pool config.json

//运行unlocker
screen ./build/bin/open-ethereum-pool unlock.json
```

## 运行www

```shell
//修改www/config/environment.js
vim www/config/environment.js

ApiUrl: '//106.15.89.134/',
HttpHost: 'http://106.15.89.134',
StratumHost: '106.15.89.134',

//编译www
cd www
npm install -g ember-cli@2.9.1
//低版本使用：npm install -g ember-cli@2.4.3
npm install -g bower
sudo npm install
bower install --allow-root
./build.sh

//复制静态文件目录
mkdir -p /data/www/
cp -r dist /data/www/

//配置nginx
vim /etc/nginx/sites-available/default

upstream api {
	server 127.0.0.1:8080;
}
root /data/www/dist;
//放在location /:之后
location /api {
	proxy_pass http://api;
}

//重启nginx
service nginx restart

//访问地址：http://106.15.89.134/
```

## 使用ethminer测试open-ethereum-pool

```shell
//下载地址https://github.com/ethereum-mining/ethminer/releases/download/v0.14.0.dev2/ethminer-0.14.0.dev2-Windows.zip
//解压至C:\ethminer
//启动
cd c:\ethminer\bin
ethminer.exe -G -F http://106.15.89.134:8888/0xc1d0932D18a4Ec35728b7fF02288dF745D1f4F40
```

## 参考文档

* [Installing Go Ethereum](https://ethereum.github.io/go-ethereum/install/#install-on-ubuntu-via-ppas)
* [以太坊客户端Geth命令用法-参数详解](http://blog.csdn.net/xilibi2003/article/details/78662000)
* [JavaScript Runtime Environment](https://ethereum.gitbooks.io/frontier-guide/content/jsre.html)
* [Ropsten Testnet Ethereum BlockChain Explorer and Search](https://ropsten.etherscan.io/)
* [Installation Instructions for Ubuntu](https://github.com/ethereum/go-ethereum/wiki/Installation-Instructions-for-Ubuntu)
* [Open Source Ethereum Mining Pool](https://github.com/sammy007/open-ethereum-pool)
* [ethminer](https://github.com/ethereum-mining/ethminer)

