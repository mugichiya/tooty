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
$ make menuconfig
$ make
```
6時間くらいかかったが、今度は問題なくビルドできた。次に、カーネルモジュールのビルドとインストールを行う。インストール先はどこでもいい。
```
$ mkdir ../mod_temp
$ make modules
$ make modules_install INSTALL_MOD_PATH=../mod_temp  INSTALL_MOD_STRIP=--strip-unneeded
```

ここまでOK。

https://www.raspberrypi.com/documentation/computers/linux_kernel.html

カーネルのイメージをコピー＆リネームし、SDカード内のイメージと置き換える、とのことだがそもそも/boot以下にkernel.imgというファイルが無い。
勝手に追加すればいいのか？それとも他のものを入れ替えるのか？
```
$ cp ./linux/arch/arm/boot/zImage ./kernel.img
$ mv /media/system-root/boot/kernel.img /media/system-root/boot/kernel.img_org
$ mv ./kernel.img /media/system-root/boot/kernel.img
```

lib/modules/3.12.31+/ をRaspberry Pi の /lib/modules/ に置く。
../mod-3.12.31/lib/firmware/ 以下のファイルも Raspberry Pi の /lib/firmware/ 以下にコピーする。らしい。
