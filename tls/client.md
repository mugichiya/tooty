# TLSクライアント
edit: 2024/07/30
## OpenSSL-1.1.1kのインストール
```
apt update && apt install -y build-essential openssl zlib1g-dev python3 curl git autoconf libtool
git clone -b 'OpenSSL_1_1_1k' --single-branch --depth 1 https://github.com/openssl/openssl /opt/openssl
cd /opt/openssl
./config no-shared enable-ssl2 enable-ssl3 enable-ssl3-method enable-weak-ssl-ciphers
make -j $(nproc)
export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/opt/openssl
cd /opt/openssl/lib
cp ../*.a .
```

## curl-7.76.1のインストール
```
git clone -b 'curl-7_76_1' --single-branch --depth 1 https://github.com/curl/curl /opt/curl
cd /opt/curl
./buildconf
LDFLAGS="-static" ./configure --with-ssl=/opt/openssl --disable-shared  --enable-static
make -j $(nproc)
export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/opt/curl
```

## ルート証明書のインストール
```
cd /usr/share/ca-certificates/mylocal
sudo su
cp /path/to/root-01.crt .
echo "mylocal/root-01.crt" >> /etc/ca-certificates.conf
update-ca-certificates
exit
```

## TLSクライアントによる接続
### まず最初にTLSサーバを準備
```
openssl genrsa -out root-00.key
openssl genrsa -out server-00.key
openssl req -new -key root-00.key -subj "/CN=psc.root" -out root-00.csr
openssl req -new -key server-00.key -subj "/CN=psc.local" -out server-00.csr
openssl x509 -req -sha256 -in root-00.csr -signkey root-00.key -days 3650 -out root-00.crt -extfile v3_ca
openssl x509 -req -sha256 -in server-00.csr -CA root-00.crt -CAkey root-00.key -CAcreateserial -days 3650 -out server-00.crt -extfile v3_req
openssl s_server -accept 443 -key server-00.key -cert server-00.crt -CAfile root-00.crt -tls1_3
```

### TLSサーバの起動
```
openssl s_server -accept 10443 -key ./server-00.key -cert ./server-01.crt -CAfile ./root-00.crt -WWW -tls1_3
```

### TLS1.2/TLS1.3をサポートしているTLSクライアント
```
curl --tlsv1.2 --tls-max 1.3 https://127.0.0.1:10443/index.html
```

### TLS1/TLS1.1/TLS1.2をサポートしているTLSクライアント
```
/opt/curl/src/curl -k --tlsv1 --tls-max 1.2 https://127.0.0.1:10443/index.html
```

