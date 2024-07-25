# GCOVカーネルのビルド

## TL;DR
- このマークダウンは、Raspberry Pi向けのGCOVカーネルビルドの備忘録
- Raspbianでカーネルをビルドすることを想定
- クロスコンパイル環境ではないので注意
-  [The Linux kernel](https://www.raspberrypi.com/documentation/computers/linux_kernel.html) を参考
    
## Raspberry Pi向けのLinuxカーネルのビルド（準備編）
### ソースコードの取得
まず、linuxカーネルのソースコードをクローンするのだが、すごい重いので注意。
```
sudo apt install git
git clone --depth=1 https://github.com/raspberrypi/linux
```
fatal: early EOF/fatal: fetch-pack: invalid index-pack outputでクローンできない場合は、ZIPでダウンロードする。
自分の場合は、クローンできなかったためZIPでダウンロードしたが、ダウンロードに半日くらいかかった...

### GCOVフラグの有効化

```
sudo apt update
sudo apt install bc bison flex libssl-dev make
sudo apt install build-essential libncurses5-dev liblz4-tool
cd /home/user/Desktop
unzip linux-rpi-6.6.y.zip
cd /linux-rpi-6.6.y
make bcm2711_defconfig
export KERNEL=kernel8
make menuconfig
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
CONFIG_GCOV_KERNEL=y
CONFIG_ARCH_HAS_GCOV_PROFILE_ALL=y
CONFIG_GCOV_PROFILE_ALL=y
```

### GCOVカーネルのビルド
次にビルドを行う。その前に、カスタムであることが分かるように.configに設定する。
例として、Linux raspberrypi 6.6.x-v8-MY_CUSTOM_KERNELとする設定を以下に記す。
```
$ uname -a
Linux raspberrypi 6.6.x-v8
$ emacs .config
$ cat .config | grep LOCALVERSION 
CONFIG_LOCALVERSION="-v8-MY_CUSTOM_KERNEL"
```
ビルドを実行する。おおよそ3時間くらいかかる。
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
再起動後にGCOVカーネルが動作するようになる。
```
$ uname -a
Linux raspberrypi 6.6.x-v8-MY_CUSTOM_KERNEL
```

## カバレッジデータの収集
/sys/kernel/debug/gcov/home/user/Desktop/linux-rpi-6.6.yに*.gcnoと*.gcdaがある。
*.gcnoは、/home/user/Desktop/linux-rpi-6.6.yにある\*.gcnoのリンク。

適当にpingを打ってカバレッジデータが書き込まれるかを確認する。
注意としては、/sys以下にある*.gcdaをlsで確認するとサイズが0バイト?と出力されるが、
見えていないだけで別のところにコピーすると正確なサイズが読み取れる。
/sys以下にあっても*.gcdaは書き込まれていることは、catで標準出力すれば分かる。
```
$ ping 127.0.0.1
$ ls -l /sys/kernel/debug/gcov/home/user/Desktop/linux-rpi-6.6.y/net/ipv4/icmp.gcda
-rw--------- 1 root root 0 Jan 1 1970 /sys/kernel/debug/gcov/home/user/Desktop/linux-rpi-6.6.y/net/ipv4/icmp.gcda
$ sudo cp /sys/kernel/debug/gcov/home/user/Desktop/linux-rpi-6.6.y/net/ipv4/icmp.gcda ~/Desktop
$ ls -l ~/Desktop/icmp.gcda
-rw--------- 1 root root 5224 Jul 25 08:59 /home/user/Desktop/linux-rpi-6.6.y/net/ipv4/icmp.gcda
$ cat /sys/kernel/debug/gcov/home/user/Desktop/linux-rpi-6.6.y/net/ipv4/icmp.gcda
```

Q. カバレッジデータが書き込まれるタイミングは?　おそらくshutdown/reboot時?

## lcovを用いたカバレッジデータの整理
```
cd /home/user/Desktop
mkdir ./cov-temp
sudo cp -a /sys/kernel/debug/gcov/home/user/Desktop/linux-rpi-6.6.y/ ./temp/
cp -a /home/user/Desktop/linux-rpi-6.6.y/* ./temp/linux-rpi-6.6.y/
cd ./temp/linux-rpi-6.6.y/
lcov -c -d . -o cov.info -rc lcov_branch_coverage=1
genhtml -o cov cov.info --branch-coverage
```

