# acme.sh-docker
通过docker部署acme.sh 实现多域名（多dns服务）更新

如果只有1个dns服务，则只需要启动一个docker，命名为acme1。如果是多个，则每个dns跑服务一个容器，方便隔离存储的认证信息。

```
    volumes:
        - /data/nginx/acme1:/acme.sh
        - /data/nginx/ssl:/etc/nginx/ssl
```
挂载两个目录进容器。

`/data/nginx/acme1`是证书生成、认证信息保存的目录

`/data/nginx/ssl`证书install后的目录。如果不需要install，不挂这个也可以，只做证书颁发（--issue）

## 启动容器
首先复制env.example到根目录，命名为.env，把dns服务对应的变量存进去。
然后执行docker-compose up -d acme1

## 发布证书

```bash
docker-compose exec acme1 acme.sh --issue --dns dns_ali -d '*.xxxxxxx.com' --standalone
```

## 安装证书

```bash
# 因为acme和nginx分属两个container，所以--reloadcmd "service nginx force-reload"是无效的
docker-compose exec acme1 acme.sh --installcert -d '*.xxxxxxx.com' --key-file /etc/nginx/ssl/*.xxxxxxx.com.key  --fullchain-file /etc/nginx/ssl/*.xxxxxxx.com.fullchain.cer  --reloadcmd "echo success"
```

安装后`/data/nginx`目录的内容如下：

```
├── acme1 // 容器工作目录
│   ├── *.xxxxxxx.com // 生成的域名证书目录
│   │   ├── *.xxxxxxx.com.cer
│   │   ├── *.xxxxxxx.com.conf
│   │   ├── *.xxxxxxx.com.csr
│   │   ├── *.xxxxxxx.com.csr.conf
│   │   ├── *.xxxxxxx.com.key
│   │   ├── backup
│   │   │   ├── fullchain.bak
│   │   │   └── key.bak
│   │   ├── ca.cer
│   │   └── fullchain.cer
│   ├── account.conf
│   ├── ca
│   │   └── acme-v02.api.letsencrypt.org
│   │       ├── account.json
│   │       ├── account.key
│   │       └── ca.conf
│   └── http.header
├── log
│   ├── access.log
│   └── error.log
├── nginx.conf
├── sites-enabled
│   ├── site1
├── ssl // 安装的证书目录，可直接挂载到nginx容器使用
│   ├── *.xxxxxxx.com.fullchain.cer
│   └── *.xxxxxxx.com.key
└── ssl_params
```
