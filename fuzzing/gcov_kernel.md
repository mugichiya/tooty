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

### GCOVカーネルのビルド
次にビルドを行う。このビルドもめちゃくちゃ時間がかかる...
```
$ make
```

数時間後、なんかエラーが出てた。
```
  UPD     include/generated/utsversion.h
  CC      init/version-timestamp.o
  LD      .tmp_vmlinux.kallsyms1
arm-linux-gnueabihf-ld: drivers/i2c/busses/i2c-designware-master.o: in function `i2c_dw_probe_master':
i2c-designware-master.c:(.text+0x32c4): undefined reference to `__aeabi_uldivmod'
arm-linux-gnueabihf-ld: i2c-designware-master.c:(.text+0x32e0): undefined reference to `__aeabi_uldivmod'
arm-linux-gnueabihf-ld: i2c-designware-master.c:(.text+0x3310): undefined reference to `__aeabi_uldivmod'
arm-linux-gnueabihf-ld: i2c-designware-master.c:(.text+0x332c): undefined reference to `__aeabi_uldivmod'
make[2]: *** [scripts/Makefile.vmlinux:37: vmlinux] エラー 1
make[1]: *** [/home/tester/Desktop/workspace2/linux-rpi-6.6.y/Makefile:1164: vmlinux] エラー 2
make: *** [Makefile:234: __sub-make] エラー 2
```

とりあえず深入りせずにクロスコンパイルをあきらめて、Raspbian上でビルドできるか試してみる。
```
$ sudo apt update
$ sudo apt install bison flex libssl-dev build-essential libncurses5-dev liblz4-tool
$ unzip linux-rpi-6.6.y.zip
$ cd ./linux-rpi-6.6.y
$ make bcm2711_defconfig
$ export KERNEL=kernel8
$ make menuconfig
```
Linux raspberrypi 6.6.x-v8-MY_CUSTOM_KERNELとなる想定。
```
$ uname -a
Linux raspberrypi 6.6.x-v8
$ emacs .config
$ cat .config | grep LOCALVERSION 
CONFIG_LOCALVERSION="-v8-MY_CUSTOM_KERNEL"
```

ビルドを実行する。3時間くらいかかる。
```
make -j6 Image.gz modules dtbs
sudo make -j6 modules_install
sudo cp /boot/firmware/$KERNEL.img /boot/firmware/$KERNEL-backup.img
sudo cp arch/arm64/boot/Image.gz /boot/firmware/$KERNEL.img
sudo cp arch/arm64/boot/dts/broadcom/*.dtb /boot/firmware/
sudo cp arch/arm64/boot/dts/overlays/*.dtb* /boot/firmware/overlays/
sudo cp arch/arm64/boot/dts/overlays/README /boot/firmware/overlays/
sudo reboot
```

一応、/sys/kernel/debug/gcov/ 以下にカバレッジデータがあることは確認できたが、
*.gcdaを見ても0バイト作成日1970/1/1のまま。
書き込まれるタイミングがあるのか？

https://www.raspberrypi.com/documentation/computers/linux_kernel.html
