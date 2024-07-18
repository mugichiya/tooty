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

v3で証明書を発行したい場合は、/etc/ssl/openssl.cnfをどこかにコピーして適宜ポリシーを変更（countryNameとかをmatchからoptionalにする）する。
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
```

## ECDSAベースの証明書
```
```
