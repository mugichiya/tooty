# gcovカーネルのビルド

## TL;DR
- このマークダウンは、Raspberry Pi向けのGCOVカーネルビルドの備忘録
- Ubuntuでカーネルをビルドすることを想定

## Raspberry Pi向けのLinuxカーネルのビルド（準備編）
### ソースコードの取得
まず、linuxカーネルのソースコードをクローンするのだが、すごい重いので注意。
```
$ git clone https://github.com/raspberrypi/linux
```
fatal: early EOF/fatal: fetch-pack: invalid index-pack outputでクローンできない場合は、ZIPでダウンロードする。
自分の場合は、クローンできなかったためZIPでダウンロードしたが、ダウンロードに半日くらいかかった...

### GCOVフラグの有効化
```
$ unzip linux-rpi-6.6.y.zip
$ cd ./linux-rpi-6.6.y
$ sudo apt install -y  build-essential libncurses5-dev liblz4-tool
$ sudo apt install bison flex
$ export ARCH=arm
$ export CROSS_COMPILE=arm-linux-gnueabihf-
$ make menuconfig
```
menuconfigでCUIベースの設定画面が表示される。
CONFIG_GCOV_KERNELとCONFIG_GCOV_PROFILE_ALLを有効にしたいので、
設定箇所を探すわけだが、どこにあるか見つけにくいため"/GCOV"でサーチする。
すると、以下のようにたどれば設定箇所にたどり着ける。

```
General architecture-dependent options
 --> GCOV-based kernel profiling
 --> Enable gcov-based kernel profiling (GCOV_KERNEL [=n])
 --> Profile entire Kernel (GCOV_PROFILE_ALL [=n])
```

最終的に、以下のようにチェックが入っていれば問題ない。
```
[*] Enable gcov-based kernel profiling
[*] Profile entire Kernel
```

SaveしてExitすると、.configが生成される。
```
$ cat .config | grep GCOV
# GCOV-based kernel profiling
CONFIG_GCOV_KERNEL=y
CONFIG_ARCH_HAS_GCOV_PROFILE_ALL=y
CONFIG_GCOV_PROFILE_ALL=y
# end of GCOV-based kernel profiling
# CONFIG_GCOV_PRFILE_FTRACE is not set
```

### ビルド
```
$ make
$ make modules
$ make modules_install INSTALL_MOD_PATH=../mod-3.12.31/  INSTALL_MOD_STRIP=--strip-unneeded
```
