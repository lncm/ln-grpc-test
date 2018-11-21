# Testing LND + gRPC

## set up a box, open TCP ports;
```
22 SSH
443 SSL
8333 BTCD
9735 LND
```

## install docker + nginx
```
sudo apt-get update
sudo apt-get install \
    nginx \
    apt-transport-https \
    ca-certificates \
    curl \
    software-properties-common -y
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
sudo add-apt-repository \
   "deb [arch=amd64] https://download.docker.com/linux/ubuntu \
   $(lsb_release -cs) \
   stable" -y
sudo apt-get update
sudo apt-get install docker-ce -y
```

## install node
```
curl -o- https://raw.githubusercontent.com/creationix/nvm/v0.33.11/install.sh | bash
. ~/.bashrc
nvm install --lts
```

## install this project + install deps
```
git clone http://github.com/lncm/ln-grpc-test
cd ln-grpc-test
npm i
```

## replace `/etc/nginx/sites-enabled/default`
```
server {
    server_name MY_DOMAIN_NAME;

    listen 443 ssl http2;

    ssl_certificate     /home/ubuntu/ln-grpc-test/tls.cert;
    ssl_certificate_key /home/ubuntu/ln-grpc-test/tls.key;

    location / {
        grpc_pass grpcs://localhost:10009;
    }
}
```

## start the docker container with config
clean up old version if requried & todo, remove --noencryptwallet
```
sudo docker stop lndt
sudo docker rm `sudo docker ps --no-trunc -aq`
sudo docker create \
--net host \
--name lndt lightninglabs/lnd \
--bitcoin.active \
--bitcoin.testnet \
--bitcoin.node=neutrino \
--neutrino.connect=faucet.lightning.community \
--noencryptwallet \
--tlsextradomain=MY_DOMAIN_NAME
sudo docker start lndt
```

## tail logs
wait for `[INF] RPCS: RPC server listening`
```
sudo docker logs lndt -f
```

## copy creds
todo, make this secure
```
rm tls.cert admin.macaroon
sudo docker cp lndt:/root/.lnd/tls.cert ./tls.cert ## used by nginx
sudo docker cp lndt:/root/.lnd/tls.key ./tls.key ## used by nginx
sudo docker cp lndt:/root/.lnd/admin.macaroon ./admin.macaroon
```

## restart nginx to pick up keys
```
sudo service nginx restart
```

## test with gRPC
from within this project's root
``` 
export LND_URI=MY_DOMAIN_NAME:443
node index.js
```
# Robert is indeed your mother's brother

---

# MISC

## To SCP the creds to a remote server
```
sudo chown ubuntu:ubuntu ./tls.cert ./admin.macaroon 
```

## SSL certificate - you don't need to do this with self-signed cert..
```
sudo apt-get update
sudo apt-get install software-properties-common -y
sudo add-apt-repository ppa:certbot/certbot -y
sudo apt-get update
sudo apt-get install python-certbot-nginx -y
sudo certbot --nginx
```