# curlでopenssl以外のライブラリも使いたい

edit: 2024/07/29

## 色々な暗号系のライブラリのインストール
```
sudo apt install libwolfssl-dev gnutls-bin libgnutls28-dev libmbedtls12 libmbedtls-dev
```

## openssl
```
git clone https://github.com/curl/curl 
cd ./curl
./buildconf
./configure --with-openssl
make -j $(nproc)
```

## gnutls
```
git clone https://github.com/curl/curl 
cd ./curl
./buildconf
./configure --with-gnutls
make -j $(nproc)
```

## mbedtls
```
git clone https://github.com/curl/curl 
cd ./curl
./buildconf
./configure --with-mbedtls
make -j $(nproc)
```

## wolftls
```
git clone https://github.com/curl/curl 
cd ./curl
./buildconf
./configure --with-wolfssl
make
```

## secure-transport
macOSだけ
```
git clone https://github.com/curl/curl 
cd ./curl
./buildconf
./configure --with-secure-channel
make
```

## bearssl
```
git clone https://www.bearssl.org/git/BearSSL
cd ./BearSSL
make
sudo cp ./build/*.so /usr/lib/
sudo cp ./build/*.a /usr/lib/
sudo cp ./inc/* /usr/include/

git clone https://github.com/curl/curl 
cd ./curl
./buildconf
./configure --with-bearssl
make
```

## makeしたcurlの使用例
まず、アクセス先のTLSサーバを適当に構築する。
```
openssl genrsa -out root-01.key
openssl genrsa -out server-01.key
openssl req -new -key root-01.key -subj "/CN=root.docker.internal" -out root-01.csr
openssl req -new -key server-01.key -subj "/CN=host.docker.internal" -out server-01.csr
openssl x509 -req -sha256 -in root-01.csr -signkey root-01.key -days 3650 -out root-01.crt
openssl x509 -req -sha256 -in server-01.csr -CA root-01.crt -CAkey root-01.key -CAcreateserial -days 3650 -out server-01.crt
echo 'This is a test.' > index.html
openssl s_server -accept 10443 -key server-01.key -cert server-01.crt -CAfile ./root-01.crt -WWW -tls1_2 
```

クローンしたリポジトリ以下の./src/にcurlの実行可能バイナリがあるので、それを使う。
```
/path/to/curl/src/curl -k --tlsv1.2 --tls-max 1.3 --resolve host.docker.internal:10443:127.0.0.1 https://host.docker.internal:10443/index.html
```
