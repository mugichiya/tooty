# VNCサーバ環境の構築

## TL;DR
- このマークダウンは、UbuntuにおけるVNCサーバ構築の備忘録
- Ubuntu mateでは下記手順で環境構築可能
- Ubuntu gnomeはデフォルトの共有があるため、競合するかも
- 最近のUbuntu gnomeはRDPなので、競合しないかも

## TigerVNCのインストール
```
$ sudo apt install tigervnc-standalone-server
$ sudo apt install tigervnc-scraping-server
$ sudo apt-get install tigervnc-common tigervnc-xorg-extension
```

## TigerVNCのパスワード設定
```
$ sudo tigervncpasswd
```

## VNCサーバの起動設定
xstarupとconfigの設定

### xstartup
```
$ touch ~/.vnc/xstartup
$ pluma  ~/.vnc/xstartup
$ cat  ~/.vnc/xstartup
#!/bin/bash
/etc/X11/Xtigervnc-session
gnome-session
```

### config
```
$ touch ~/.vnc/config
$ pluma ~/.vnc/config
$ cat ~/.vnc/config
session=ubuntu
geometry=1200x720
alwaysshared
```

### ユーザ設定
```
$ sudo pluma /etc/tigervnc/vncserver.users
$ cat /etc/tigervnc/vncserver.users
:2=tester
```

### 環境変数の設定
```
$ sudo pluma /etc/X11/Xsession.d/80mate-environment
$ cat /etc/X11/Xsession.d/80mate-environment
unset DBUS_SESSION_BUS_ADDRESS　←これを先頭に追記
```

## VNCサーバの起動
```
$ sudo x0vncserver -passwordfile ~/.vnc/passwd -localhost no -display :0
```

## VNCサーバのサービス化
```
$ sudo pluma /etc/systemd/system/vncserver.service
$ cat /etc/systemd/system/vncserver.service
[Unit]
Description=VNC Server
Documentation=

[Service]
Type=forking
User=tester
Group=tester
TimeoutStartSec=0
Restart=on-failure
RestartSec=30
ExecStart=/usr/bin/x0vncserver -passwordfile /home/tester/.vnc/passwd -localhost no -display :0
SyslogIdentifier=Deskutilization

[Install]
WantedBy=multi-user.target

$ sudo systemctl daemon-reload
$ sudo systemctl enable vncserver.service
$ sudo systemctl start vncserver.service
```


## VNCクライアント
例えば、virt-viewerとかRemminaとか
```
$ sudo apt install -y virt-viewer
$ sudo apt install -y remmina
```
