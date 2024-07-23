# Docker4

## 目次
+ Gitサーバ
+ Sambaサーバ
+ SSHサーバ
+ AFL

## Gitbucket

```
$ docker run -it -d -p 18080:8080 gitbucket/gitbucket
```


## Sambaの設定
### Ubuntu18.04 for Samba

```
$ sudo apt-get -y install samba
$ mkdir /home/user/share
$ chmod 777 /home/user/share
$ sudo emacs /etc/samba/smb.conf
```

### smb.confの編集

```
unix charset = UTF-8
dos charset = CP932
interfaces = 127.0.0.0/8 10.0.0.0/24
bind interfaces only = yes
map to guest = Bad User

[Share]
   path = /home/user/share
   writable = yes
   guest ok = yes
   guest only = yes
   create mode = 0777
   directory mode = 0777
```

## SSHサーバ／クライアントの設定
### SSHクライアント(macOS側)

```
$ ssh-keygen -t rsa
  --> /home/user/.ssh/id_rsa （秘密鍵）
  --> /home/user/.ssh/id_rsa.pub （公開鍵）
$ ssh user@192.168.179.7
>$ mkdir .ssh
>$ touch .ssh/authorized_keys
>$ chmod 700 .ssh
>$ chmod 600 .ssh/authorized_keys
>$ exit
$ cat ~/.ssh/id_rsa.pub | ssh user@192.168.179.7 'cat >> .ssh/authorized_keys'
```

### SSHサーバ(Ubuntu18.04 for Fuzzing側)

```
$ sudo emacs /etc/ssh/ssh_config
(rewrite as bellow...)
PasswordAuthentication no
ChallengeResponseAuthentication no
$ sudo /etc/init.d/ssh restart
```


### Docker上のSSHサーバに接続

```
FROM verifsec/ubuntu-afl:16.04

RUN apt-get update && apt-get install -y openssh-server
RUN mkdir -p /var/run/sshd /home/user/targets

RUN sed -i 's/PermitRootLogin prohibit-password/PermitRootLogin yes/' /etc/ssh/sshd_config
RUN sed -i 's/#PasswordAuthentication yes/PasswordAuthentication no/' /etc/ssh/sshd_config

RUN sed 's@session\s*required\s*pam_loginuid.so@session optional pam_loginuid.so@g' -i /etc/pam.d/sshd

ENV NOTVISIBLE "in users profile"
RUN echo "export VISIBLE=now" >> /etc/profile

COPY id_rsa.pub /root/authorized_keys
COPY monitor.py /home/user

EXPOSE 22

CMD mkdir ~/.ssh && \
    mv ~/authorized_keys ~/.ssh/authorized_keys && \
    chmod 0600 ~/.ssh/authorized_keys &&  \
    /usr/sbin/sshd -D
```

## AFLの設定

### Ubuntu18.04 for Fuzzing

```
$ sudo apt install make git build-essential automake libtool-bin libglib2.0-dev bison clang-3.9 llvm-3.9
$ sudo git clone https://github.com/google/AFL.git /opt/AFL
$ cd /opt/AFL/
$ sudo make
$ cd ./llvm_mode/
$ sudo make LLVM_CONFIG=llvm-config-3.9 CC=clang-3.9 CXX=clang++-3.9
$ cd ../qemu_mode
$ ./build_qemu_support.sh
$ cd ../
$ make install
```



## 今後の予定
+ <strike>Dockerの基本的な操作</strike>（済）
+ <strike>Ubuntuでファジングの環境構築</strike>（済）
+ <strike>ARM向けファジング環境構築</strike>（済）
+ クラウドプラットフォームの利用
+ <strike>OSS-fuzz</strike>（済）
+ コンテナ間通信
+ Docker-Compose
