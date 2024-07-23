# curlでopenssl以外のライブラリも使いたい

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
```
/path/to/curl/src/curl -k --tlsv1.2 --tls-max 1.3 --resolve host.docker.internal:10443:127.0.0.1 https://host.docker.internal:10443/index.html
```
