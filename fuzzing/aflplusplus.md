# AFLplusplusの環境構築
edit: 2024/07/17

## TL;DR
- このマークダウンでは、AFL++によるFuzzing環境構築の備忘録
- Ubuntuでは下記の手順で環境構築可能

## インストール
```
$ cd /opt
$ sudo git clone https://github.com/AFLplusplus/AFLplusplus
$ sudo emacs ./AFLplusplus/GNUmakefile
(L01) CFLAGS += -DSIMPLE_FILES
$ sudo make -C ./AFLplusplus
```

## afl-ccによるコンパイル
例として、libjpeg-turboにおけるafl-ccの適用
```
$ mkdir /home/user/Desktop/afl-tc
$ cd /home/user/Desktop/afl-tc
$ git clone https://github.com/libjpeg-turbo/libjpeg-turbo
$ cd ./libjpeg-turbo
$ mkdir ./build
$ cd ./build
$ CC=/opt/AFLplusplus/afl-cc cmake ../
$ make
```

### Fuzzingの実行
例として、libjpeg-turbo/djpegのFuzzing
```
$ mkdir in
$ cp /path/to/valid.jpg ./in
$ sudo su
# echo core >/proc/sys/kernel/core_pattern
# cd /sys/devices/system/cpu
# echo performance | tee cpu*/cpufreq/scaling_governor
# exit
$ /opt/AFLplusplus/afl-fuzz -m none -i ./in -o out -- ./djpeg -gif @@
```

## その他、調査した方がいい事項
- afl-clang-ltoとかafl-clang-fastとか
- afl-fuzzの再開方法（たぶん、afl -i-だと思うけど）
- などなど
