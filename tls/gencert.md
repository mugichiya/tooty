# 証明書の作成
edit: 2024/07/18

## OpenSSLのインストール
```
$ sudo apt update & sudo apt install -y openssl
```

最新のOpenSSLでは、SSL3.0/TLS1/TLS1.1が使用できないため、古いバージョンもインストールしておく。
例えば、1.0.1aの場合、以下のようにインストールする。
```
$ sudo apt update & sudo apt install -y openssl
$ cd /opt
$ apt install wget
$ wget -P /tmp https://ftp.openssl.org/source/old/1.0.1/openssl-1.0.1a.tar.gz
$ sudo tar xvzf /tmp/openssl-1.0.1a.tar.gz -C /opt/
$ cd /opt/openssl-1.0.1a/
$ sudo ./config shared
$ sudo make
$ export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/opt/openssl-1.0.1a
```

## RSAベースの証明書
何も考えずにv1で良ければ、以下の通り。
```
$ openssl genrsa -out root-01.key
$ openssl genrsa -out server-01.key
$ openssl req -new -key root-01.key -subj "/CN=root.docker.internal" -out root-01.csr
$ openssl req -new -key server-01.key -subj "/CN=host.docker.internal" -out server-01.csr
$ openssl x509 -req -sha256 -in root-01.csr -signkey root-01.key -days 3650 -out root-01.crt
$ openssl x509 -req -sha256 -in server-01.csr -CA root-01.crt -CAkey root-01.key -CAcreateserial -days 3650 -out server-01.crt
$ /opt/openssl-1.0.1a/apps/openssl x509 -req -sha1 -in server-01.csr -CA root-01.crt -CAkey root-01.key -CAcreateserial -days 3650 -out sha1-01.crt
$ /opt/openssl-1.0.1a/apps/openssl x509 -req -md5 -in server-01.csr -CA root-01.crt -CAkey root-01.key -CAcreateserial -days 3650 -out md5-01.crt
```

v3で証明書を発行したい場合は、/etc/ssl/openssl.cnfをどこかにコピーして適宜ポリシーを変更（countryName等をoptionalにする）する。
```
$ openssl req -new -key root-01.key -subj "/CN=root.docker.internal" -out root-01.csr \
    -extensions /path/to/v3_ca \
    -config /path/to/openssl.cnf
$ openssl x509 -req -sha256 -in server-01.csr -CA root-01.crt -CAkey root-01.key -CAcreateserial -days 3650 -out server-01.crt \
    -extensions /path/to/v3_req \
    -config /path/to/openssl.cnf
```

v3_caやv3_reqのサンプルは以下の通り。
```
$ cat /path/to/v3_ca
authorityKeyIdentifier=keyid,issuer
subjectKeyIdentifier=hash
basicConstraints=critical,CA:true

$ cat /path/to/v3_req
authorityKeyIdentifier=keyid,issuer
basicConstraints=CA:false
keyUsage = digitalSignature, nonRepudiation, keyEncipherment, dataEncipherment
```

## DSAベースの証明書
```
$ openssl dsaparam -out root-01.param 2048
$ openssl gendsa -out root-01.key root-01.param
$ openssl dsaparam -out server-01.param 2048
$ openssl gendsa -out server-01.key server-01.param
```

## ECDSAベースの証明書
```
$ openssl ecparam -genkey -name secp521r1 -out root-01.key
$ openssl ecparam -genkey -name secp521r1 -out server-01.key
```

## 有効期限の指定方法
```
mkdir -p ./demoCA/private         
mkdir -p ./demoCA/newcerts        
cp /path/to/root-01.crt ./demoCA/cacert.pem 
cp /path/to/root-01.key ./demoCA/private/cakey.pem 
echo '00' > ./demoCA/serial             
touch ./demoCA/index.txt          
openssl ca -in /path/to/server-01.csr -startdate 20200101000000Z -enddate 20200102000000Z -out expired-01.crt
rm -rf demoCA
```

## TLSサーバの起動
### HTMLファイルの準備
```
$ cd /path/to/certs
$ cat ./index.html
This is a test.
```

### TLS1.3/TLS1.2/TLS1.1/TLS1/SSL3サーバ
```
$ openssl s_server -accept 10443 -key ./server-01.key -cert ./server-01.crt -CAfile ./root-01.crt -WWW -tls1_3
$ openssl s_server -accept 10443 -key ./server-01.key -cert ./server-01.crt -CAfile ./root-01.crt -WWW -tls1_2
$ /opt/openssl-1.0.1a/apps/openssl s_server -accept 10443 -key ./server-01.key -cert ./server-01.crt -CAfile ./root-01.crt -WWW -tls1_1
$ /opt/openssl-1.0.1a/apps/openssl s_server -accept 10443 -key ./server-01.key -cert ./server-01.crt -CAfile ./root-01.crt -WWW -tls1
$ /opt/openssl-1.0.1a/apps/openssl s_server -accept 10443 -key ./server-01.key -cert ./server-01.crt -CAfile ./root-01.crt -WWW -ssl3
```

### SHA-1/MD5の証明書を用いたサーバ
```
$ /opt/openssl-1.0.1a/apps/openssl s_server -accept 10443 -key ./server-01.key -cert ./sha1-01.crt -CAfile ./root-01.crt -WWW -tls1_2
$ /opt/openssl-1.0.1a/apps/openssl s_server -accept 10443 -key ./server-01.key -cert ./md5-01.crt -CAfile ./root-01.crt -WWW -tls1_2
```

### 自己署名証明書/期限切れのサーバ
```
$ openssl s_server -accept 10443 -key ./root-01.key -cert ./root-01.crt -WWW -tls1_2
$ openssl s_server -accept 10443 -key ./server-01.key -cert ./expired-01.crt -CAfile ./root-01.crt -WWW -tls1_2
```
