# ARM向けファジング環境構築

edit: 2019/xx/yy （過去にまとめたものなので、各種のバージョンは古い。）

## 目次
+ イメージのダウンロード
+ Docker用のイメージ作成
+ QEMU on Docker(Ubuntu16.04)
+ AFL（LLVMモード）の実行

## イメージのダウンロード

[Download](https://ubuntu-pi-flavour-maker.org/download)からubuntu-16.04-preinstalled-server-armhf+raspi3.img.xzをダウンロードして解凍

```
$ xz -dv ubuntu-16.04-preinstalled-server-armhf+raspi3.img.xz
ubuntu-16.04-preinstalled-server-armhf+raspi3.img.xz (1/1)
  100 %      251.1 MiB / 2252.0 MiB = 0.111   114 MiB/s       0:19
$ ls ubuntu-16.04-preinstalled-server-armhf+raspi3.img
ubuntu-16.04-preinstalled-server-armhf+raspi3.img
```

## qemu-arm-staticのインストール

+ 以下より，セクタサイズが512バイトでパーティション2の開始位置が270336セクタであることが分かる．

```
# fdisk -l ubuntu-16.04-preinstalled-server-armhf+raspi3.img
Disk ubuntu-16.04-preinstalled-server-armhf+raspi3.img: 2.2 GiB, 2361393152 bytes, 4612096 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: dos
Disk identifier: 0x124affd9

Device                                             Boot  Start     End Sectors  Size Id Type
ubuntu-16.04-preinstalled-server-armhf+raspi3.img1 *      8192  270335  262144  128M  c W95 FAT32 (LBA)
ubuntu-16.04-preinstalled-server-armhf+raspi3.img2      270336 4610047 4339712  2.1G 83 Linux
```

+ パーティション2のマウント

```
# mkdir ./images
# mount -o loop,offset=$((512*270336)) ubuntu-16.04-preinstalled-server-armhf+raspi3.img ./image/
# ls ./image
bin boot dev etc home lib ...[snip]...
```

+ qemu-arm-staticをインストールし，Docker用のイメージを作成

```
# cp /usr/bin/qemu-arm-static ./image/usr/bin
# cd ./image
# tar cf ../ubuntu-16.04-armhf.tar .
```

+ パーティション2のアンマウント

```
# cd ../
# umount ./image
```



## QEMU on Docker

+ イメージのインポート

```
# docker import ubuntu-16.04-armhf.tar ubuntu:pi
sha256:6b291e5a4f510e4b3e9f0d52ee4d36d483342dc633ea8a6caaf7c887a0898df9
# docker run --privileged -it --name pi ubuntu:pi /bin/bash
root@d5f5550cf02c:/# exit
exit
```

+ 中断した後はスタート＆アタッチで再開

```
# docker start pi
# docker attach pi
# docker stop pi
```

## AFL（LLVMモード）
### インストール

```
root@d5f5550cf02c:/# apt install -y gcc make git clang llvm gnuplot
root@d5f5550cf02c:/# git clone https://github.com/AFL.git /opt/AFL
root@d5f5550cf02c:/# cd /opt/AFL/
root@d5f5550cf02c:/opt/AFL# make AFL_NO_X86=1
root@d5f5550cf02c:/opt/AFL# make -C ./llvm-mode AFL_NO_X86=1
root@d5f5550cf02c:/opt/AFL# make install AFL_NO_X86=1
```


### サンプルの実行

```
root@d5f5550cf02c:/opt/AFL# mkdir ./sample
root@d5f5550cf02c:/opt/AFL/sample# mkdir in out
root@d5f5550cf02c:/opt/AFL/sample# echo aaaa > ./in/seed
root@d5f5550cf02c:/opt/AFL/sample# afl-clang-fast -o test-instr ../test-instr.c
afl-cc 2.54b by <lcamtuf@google.com>
afl-as 2.54b by <lcamtuf@google.com>
root@d5f5550cf02c:/opt/AFL/sample# afl-fuzz -m none -i ./in -o ./out -- ./test-instr
```

### Docker上での実行結果(ubuntu16.04)
+ 一旦停止等のハングは発生しなかった．

|Normal mode|QEMU mode|LLVM mode|
|:---:|:---:|:---:|
| - | - |~400/sec|

### bmp2tiffの実行結果
+ 一旦停止等のハングが発生...
+ （perfを使って原因調査せよ）

|Normal mode|QEMU mode|LLVM mode|
|:---:|:---:|:---:|
| - | - |~0/sec|

### カスタムイメージの作成

```
$ docker commit pi ubuntu-armhf:fuzz
$ docker save ubuntu-armhf:fuzz -o ubuntu-armhf-fuzz.tar
```

## RPI3動作確認済みのイメージをDockerで動かす

+ SDカードからcloudimg-rootfsのコピー
+ qemu-arm-staticをインストールし，Docker用のイメージを作成

```
# cp /usr/bin/qemu-arm-static ./cloudimg-rootsf/usr/bin
# cd ./cloudimg-rootsf
# tar cf ../ubuntu-16.04-rpi3.tar .
# rm ./usr/bin/qemu-arm-static
```


## 疑問
+ インストールの度に以下のエラーが出力され，apt upgradeができない． 
	+ A. docker run時に--privilegedを付けると，apt upgradeできない問題は解決するが同じエラーが出力される．

```
Label cloudimg-rootfs not found in /dev/disk/by-label
Warning: root device LABEL=cloudimg-rootfs does not exist

Press Ctrl-C to abort build, or Enter to continue

W: mdadm: /etc/mdadm/mdadm.conf defines no arrays.
Unsupported platform.
run-parts: /etc/initramfs/post-update.d//flash-kernel exited with return code 1
dpkg: error processing package initramfs-tools (--configure):
 subprocess installed post-installation script returned error exit status 1
```

+ 存在しないコマンドを入力した場合，以下のエラーが出力される．

```
qemu: Unsupported syscall: 384
```

## まとめ
+ ubuntu-16.04-rpi3.tar（エラーが出る・ファジング用）
+ ubuntu-armhf-fuzz.tar（エラーが出る・ファジング用）
+ ubuntu-16.04-armhf.tar（エラーが出る・ファジング用でない）


## 今後の予定
+ <strike>Dockerの基本的な操作</strike>（済）
+ <strike>Ubuntuでファジングの環境構築</strike>（済）
+ <strike>ARM向けファジング環境構築</strike>（済）
+ クラウドプラットフォームの利用
+ OSS-fuzz
