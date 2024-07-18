# TLSサーバ

## 
```
$ cd /path/to/certs
$ cat ./index.html
This is a test.
$ openssl s_server -accept 20543 -key ./server-01.key -cert ./server-01.crt -CAfile ./root-01.crt -WWW -tls1_3
$ openssl s_server -accept 20543 -key ./server-01.key -cert ./server-01.crt -CAfile ./root-01.crt -WWW -tls1_2
$ /opt/openssl-1.0.1a/apps/openssl s_server -accept 20543 -key ./server-01.key -cert ./server-01.crt -CAfile ./root-01.crt -WWW -tls1_1
$ /opt/openssl-1.0.1a/apps/openssl s_server -accept 20543 -key ./server-01.key -cert ./server-01.crt -CAfile ./root-01.crt -WWW -tls1
$ /opt/openssl-1.0.1a/apps/openssl s_server -accept 20543 -key ./server-01.key -cert ./server-01.crt -CAfile ./root-01.crt -WWW -ssl3
```
