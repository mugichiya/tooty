# Dockerに関する備忘録
edit: 2024/07/29
## TL;DR
- このマークダウンは、Dockerの環境設定時に知ったことの備忘録

## Dockerのインストール
ちょっと前のものだから、最新のは違うかも。
```
# sudo apt update
# sudo apt install -y apt-transport-https ca-certificates curl software-properties-common
# curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
# sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"
# sudo apt update
# sudo apt install -y docker-ce
# sudo chmod 666 /var/run/docker.sock
```

## 既成のイメージを使う
ubuntuの最新版のイメージを使用してコンテナ環境に入る方法は以下の通り。
```
$ docker run -it -d --name ubuntu ubuntu:latest
$ docker exec -it ubuntu bash
```

わざわざexecしなくても以下のように入ることもできる。
```
$ docker run --rm -it ubuntu:latest bash
```

展開したコンテナを消す方法は以下の通り。
```
$ docker ps -a
$ docker stop ubuntu
$ docker rm ubuntu
```

## コンテナのイメージ化
```
$ docker stop ubuntu
$ docker commit ubuntu ubuntu:fuzz
$ docker save ubuntu:fuzz -o ubuntu-fuzz.tar
$ ls -l ubuntu-fuzz.tar 
$ docker restart ubuntu
```

## コンテナとUSBアダプタ
Ubuntuで認識されるBluetoothアダプタ(USB)を挿入し、lsusbでバス番号とデバイス番号を確認する。
```
$ lsusb
Bus 002 Device 001: ID 1d6b:0003 Linux Foundation 3.0 root hub
Bus 001 Device 006: ID 8087:0a2b Intel Corp. Bluetooth wireless interface
Bus 001 Device 005: ID 5986:054c Acer, Inc USB HD Webcam
Bus 001 Device 004: ID 0a12:0001 Cambridge Silicon Radio, Ltd Bluetooth Dongle (HCI mode)
Bus 001 Device 003: ID 13fe:6700 Kingston Technology Company Inc. USB DISK
Bus 001 Device 002: ID 046d:c52b Logitech, Inc. Unifying Receiver
Bus 001 Device 001: ID 1d6b:0002 Linux Foundation 2.0 root hub
```
→`/dev/bus/usb/001/004`がBluetoothアダプタに該当、バス番号は001とデバイス番号は004。

それ用のイメージをビルドする。（別にしなくてもいいけど）
```
$ cat /path/to/Dockerfile
FROM ubuntu:latest
WORKDIR opt
RUN ln -sf /usr/share/zoneinfo/Asia/Tokyo /etc/localtime
RUN apt update
RUN apt install -y wget
RUN apt install -y python3 python3-scapy python3-terminaltables
RUN apt install -y bluetooth bluez libbluetooth-dev

$ docker build /path/to/ -t ubuntu:blue
$ docker images
```

以下のようにhciconfigを実行。
```
$ docker run --rm -it --device /dev/bus/usb/001/004 --net host --privileged \
      -v /var/run/dbus/system_bus_socket:/var/run/dbus/system_bus_socket:ro \
      ubuntu:blue hciconfig
```

または、bluetoothctlを実行。
```
$ docker run --rm -it --device /dev/bus/usb/001/004 --net host --privileged \
      -v /var/run/dbus/system_bus_socket:/var/run/dbus/system_bus_socket:ro \
      ubuntu:blue bluetoothctl
```

イメージの削除は以下の通り。
```
$ docker rmi ble_rw:latest
$ docker system prune
```
