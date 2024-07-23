# Dockerの基本的な操作

## 目次
+ インストール
+ Nginxの起動
+ Tomcatの起動
+ Dockerイメージ作成

## インストール for macOS
+ DockerにSign in
+ [Docker Desktop for Mac](https://hub.docker.com/editions/community/docker-ce-desktop-mac)からdmgをダウンロード
+ インストール後にもう一度，DockerにSign in

## インストール for Ubuntu18.04

```
# sudo apt update
# sudo apt install -y apt-transport-https ca-certificates curl software-properties-common
# curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
# sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"
# sudo apt update
# sudo apt install -y docker-ce
# sudo chmod 666 /var/run/docker.sock
```


## Dockerとは
+ コンテナ型の仮想環境
+ ホストマシンのカーネルでプロセスやユーザなどを隔離し、あたかも別のマシンが動いているかのよう
+ 軽量で高速に起動、停止などが可能

## Nginxの起動
+ ホストマシンのtcp/8080から仮想環境のtcp/80のNginxにアクセス
+ 起動時にイメージを持っていないため，自動でDockerHub
からダウンロードされる

```
$ docker run --name some-nginx -d -p 8080:80 nginx
Unable to find image 'nginx:latest' locally
latest: Pulling from library/nginx
1ab2bdfe9778: Pull complete 
a17e64cfe253: Pull complete 
e1288088c7a8: Pull complete 
Digest: sha256:53ddb41e46de3d63376579acf46f9a41a8d7de33645db47a486de9769201fec9
Status: Downloaded newer image for nginx:latest
06946fb7e31ce4d2192cc756b80a9c24fcfaeb8d02964a4aa6fa77bc6370cc83
```

+ localhost:8080にアクセス

```
$ curl localhost:8080
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
    body {
        width: 35em;
        margin: 0 auto;
        font-family: Tahoma, Verdana, Arial, sans-serif;
    }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
```

## コンテナのプロセス管理と削除
+ コンテナのプロセスを取得

```
$ docker ps -a
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                  NAMES
06946fb7e31c        nginx               "nginx -g 'daemon of…"   11 minutes ago      Up 11 minutes       0.0.0.0:8080->80/tcp   some-nginx
```

+ プロセスを停止

```
$ docker stop some-nginx
some-nginx
```

+ プロセスを削除

```
$ docker rm some-nginx
some-nginx
```

## Tomcatの起動
+ CentOS7を起動

```
$ docker run -it -d -p 18080:8080 -v /Users/user/Desktop/tomcat/:/share/logs/ --name tomcat centos:7
b2845822da3a24119435340a2583d170d53dc17b974f489ddea37113af91a1e3
```

+ CentOS7のBashを起動

```
$ docker exec -it tomcat bash
[root@b2845822da3a /]# whoami
root
[root@b2845822da3a /]# exit
exit
$ whoami
user
```


+ ホストマシンに[Tomcat 9.0.24](http://www-eu.apache.org/dist/tomcat/tomcat-9/v9.0.24/bin/apache-tomcat-9.0.24.tar.gz)をダウンロード

```
$ wget http://www-eu.apache.org/dist/tomcat/tomcat-9/v9.0.24/bin/apache-tomcat-9.0.24.tar.gz
```

+ CentOSにTomcatを共有

```
$ export SHARE_PAT=/Users/user/Desktop/tomcat
$ mv apache-tomcat-9.0.24.tar.gz $SHARE_PATH
$ docker exec -it tomcat bash
[root@b2845822da3a /]# ls /share/logs/
apache-tomcat-9.0.24.tar.gz
```

+ CentOSにTomcatを共有2

```
$ docker cp <host machine's file> <containerID>:<guest machine's directory>
$ docker cp <containerID>:<guest machine's directory> <host machine's file>
```

+ Tomcatのインストール＆起動

```
[root@b2845822da3a /]# yum install -y java
[root@b2845822da3a /]# cd /share/logs/
[root@b2845822da3a /]# tar xvzf apache-tomcat-9.0.24.tar.gz
[root@b2845822da3a /]# cd ./apache-tomcat-9.0.24/
[root@b2845822da3a apache-tomcat-9.0.24]# ./bin/startup.sh  
¥Using CATALINA_BASE:   /share/logs/apache-tomcat-9.0.24
Using CATALINA_HOME:   /share/logs/apache-tomcat-9.0.24
Using CATALINA_TMPDIR: /share/logs/apache-tomcat-9.0.24/temp
Using JRE_HOME:        /usr
Using CLASSPATH:       /share/logs/apache-tomcat-9.0.24/bin/bootstrap.jar:/share/logs/apache-tomcat-9.0.24/bin/tomcat-juli.jar
Tomcat started.
```

+ Tomcatの終了

```
[root@b2845822da3a apache-tomcat-9.0.24]# ./bin/shutdown.sh 
Using CATALINA_BASE:   /share/logs/apache-tomcat-9.0.24
Using CATALINA_HOME:   /share/logs/apache-tomcat-9.0.24
Using CATALINA_TMPDIR: /share/logs/apache-tomcat-9.0.24/temp
Using JRE_HOME:        /usr
Using CLASSPATH:       /share/logs/apache-tomcat-9.0.24/bin/bootstrap.jar:/share/logs/apache-tomcat-9.0.24/bin/tomcat-juli.jar
```


+ ログの共有＆再起動

```
[root@b2845822da3a apache-tomcat-9.0.24]# cd /opt/
[root@b2845822da3a opt]# mv /share/logs/apache-tomcat-9.0.2 /opt/
[root@b2845822da3a opt]# cd ./apache-tomcat-9.0.24/
[root@b2845822da3a apache-tomcat-9.0.24]# sed -i -e "s/\${catalina.base}\/logs/\/share\/logs/g" ./conf/logging.properties
[root@b2845822da3a omcat-9.0.24/
[root@b2845822da3a ]# ./bin/startup.sh
[root@b2845822da3a ]# exit
exit
$ ls
apache-tomcat-9.0.24.tar.gz	localhost.2019-09-06.log
catalina.2019-09-06.log		manager.2019-09-06.log
host-manager.2019-09-06.log
```

+ CentOSの終了
 
```
$ docker stop tomcat 
tomcat
$ docker rm tomcat 
tomcat
```


## Dockerfile
+ DockerfileによってDockerコンテナ起動の起動や各コマンド実行を自動化できる

+ Dockerfileを書く

```
$ cat Dockerfile 
FROM centos:7
RUN yum install -y java wget
RUN wget http://www-eu.apache.org/dist/tomcat/tomcat-9/v9.0.24/bin/apache-tomcat-9.0.24.tar.gz -O /opt/apache-tomcat-9.0.24.tar.gz
RUN tar xvzf /opt/apache-tomcat-9.0.24.tar.gz -C /opt
CMD [ "/opt/apache-tomcat-9.0.24/bin/catalina.sh", "run" ]
```

+ Dockerビルドを実行してDockerイメージを作成

```
$ docker build -t tomcat:1 .
 ...[snip]..
$ docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
tomcat              1                   79f8dab123f5        46 seconds ago      529MB
centos              7                   67fa590cfc1c        2 weeks ago         202MB
nginx               latest              5a3221f0137b        3 weeks ago         126MB
```

+ 起動と終了，イメージの削除

```
$ docker run -it -d --name tomcat-1 -p 18080:8080 tomcat:1
f9f9840a835f1501df7b185fef9b1ee549d9b225b3d301e244210c6acc915217
$ docker stop tomcat-1
$ docker rm tomcat-1
$ docker image rm -f 79f8dab123f5
Deleted: sha256:79f8dab123f5827ee1f3fd6a5e3675ab277a16003bd01e6b40134a74c2e6b103
Deleted: sha256:4a6d6a2591d977fdece9d39dde922c4d25927f2691af46bfec03797678849986
Deleted: sha256:e6f78b341cb485c80713f85824eb42641fd31c14740a361706c38f4850eedc3b
Deleted: sha256:9afb05c390669662f32832e4ef1f3b5eba9e104d916b2b7eecfb7a020b88d847
Deleted: sha256:970d3c855bb4061d39a65f7e8f725af778c35e91587d1393f3913b0668f0d00a
Deleted: sha256:8c5ea3c4d6e17735e5379ebd77109d7867dde3f274b1de5f99e364bf5fb0ad62
Deleted: sha256:7395d66607abddff23a4c22cc716ca4e51ed8b040e49ac70e78825cbbcc70af0
```
 

## コンテナからDockerイメージの作成
+ Dockerfileから起動したTomcatサーバでnmapなどをインストール

```
$ docker build -t tomcat:1 .
$ docker run -it -d --name tomcat-1 -p 18080:8080 tomcat:1
$ docker exec -it tomcat-1 bash
[root@f2333746e809 /]# yum install -y nmap net-tools
[root@f2333746e809 /]# exit
exit
```

+ Dockerイメージを作成

```
$ docker stop tomcat-1 
tomcat-1
$ docker commit tomcat-1 tomcat-image
sha256:f8f050544948f00138b35a02f12e5f97697022ae3ecfe946f745e17df8e3b326
$ docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
tomcat-image        latest              f8f050544948        13 seconds ago      658MB
tomcat              1                   d6c4107ae0a1        6 minutes ago       529MB
centos              7                   67fa590cfc1c        2 weeks ago         202MB
nginx               latest              5a3221f0137b        3 weeks ago         126MB
```

+ Dockerイメージを起動

```
$ docker run -it -d -p 18082:8080 --name tomcat2 tomcat-image
c21547fddb898ca41d10bc52ce253ebe14c0e806c51f427dc50c268a0cd18b71
user:~/Desktop/tomcat/tom
$ docker exec -it tomcat2 bash
[root@c21547fddb89 /]# ls /usr/bin/nmap 
/usr/bin/nmap
[root@c21547fddb89 /]# ifconfig         
eth0: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 172.17.0.2  netmask 255.255.0.0  broadcast 172.17.255.255
        ether 02:42:ac:11:00:02  txqueuelen 0  (Ethernet)
        RX packets 12  bytes 968 (968.0 B)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 0  bytes 0 (0.0 B)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

lo: flags=73<UP,LOOPBACK,RUNNING>  mtu 65536
        inet 127.0.0.1  netmask 255.0.0.0
        loop  txqueuelen 1  (Local Loopback)
        RX packets 0  bytes 0 (0.0 B)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 0  bytes 0 (0.0 B)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
```

+ CentOSの終了

```
$ docker stop tomcat2 
tomcat2
$ docker rm tomcat-1 
tomcat-1
$ docker rm tomcat2 
tomcat2
```

+ イメージの削除

```
$ docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
tomcat-image        latest              f8f050544948        3 minutes ago       658MB
tomcat              1                   d6c4107ae0a1        10 minutes ago      529MB
centos              7                   67fa590cfc1c        2 weeks ago         202MB
nginx               latest              5a3221f0137b        3 weeks ago         126MB
```

+ 子イメージよりも先に親イメージを削除しようとするとエラーが発生

```
$ docker image rm -f d6c4107ae0a1
Error response from daemon: conflict: unable to delete d6c4107ae0a1 (cannot be forced) - image has dependent child images
```

+ 子イメージ，親イメージの順番で削除

```
$ docker image rm f8f050544948
Untagged: tomcat-image:latest
Deleted: sha256:f8f050544948f00138b35a02f12e5f97697022ae3ecfe946f745e17df8e3b326
Deleted: sha256:9c60736e116e54adcbe184eae755a4d93b31b9ec877f8a8023299e17e14401e2
$ docker image rm d6c4107ae0a1
Untagged: tomcat:1
Deleted: sha256:d6c4107ae0a129a97e5537f0fecf5722a79b9506c18de3f3d5a6ed0f22778636
Deleted: sha256:1bf5b648a76e70382047ce208e5e017b27f249c46c395bbc223d32e214c832b2
Deleted: sha256:67633c78f9ddfba216928520606f6a2cd7fde3377fe9dfbd44f6aba7bb64da12
Deleted: sha256:6b881d9e05953b504042bd08894995b3e26f102b21524b97fa4f809d303b87fa
Deleted: sha256:b0a1089c06ca3f8a7462864a07ac5fa75f32426cbd071c999dc13518a38daf62
Deleted: sha256:9b170d5191b48aa0995c3fad5d67d6fd67b672bea081d4e175758cee480edae2
Deleted: sha256:7f2798e602b0e8124950d6b3c79f830468d2119b1740038772899ebf8f4d3e71
```

## 疑問と答え
+ イメージの親子関係は独立できないのか?
+ イメージの配布方法は？　A. savesでtarとして出力

## 付録

+ Dockerイメージのエクスポート/インポート

```
$ docker save ubuntu -o ubuntu.tar
$ docker rmi ubuntu
$ docker load -i ubuntu.tar 
```

## 今後の予定
+ <strike>Dockerの基本的な操作</strike>（済）
+ Ubuntuでファジングの環境構築
+ クラウドプラットフォームの利用
+ OSS-fuzz
