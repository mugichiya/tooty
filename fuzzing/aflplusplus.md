# AFLplusplusの環境構築
edit: 2024/08/21

## TL;DR
- このマークダウンは、AFL++によるFuzzing環境構築の備忘録
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
例として、libjpeg-turboにおけるafl-ccの適用。
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
例として、libjpeg-turbo/djpegのFuzzingは以下の通り。
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

## その他
### Fuzzingの再開
以前にFuzzingを行った際に生成されたディレクトリを指定し、`-i-`で再開できる。(レガシーのAFLと同じ方法)
```
/opt/AFLplusplus/afl-fuzz -m none -i- -o out -- ./djpeg -gif @@
```

### Clang/LLVM
LLVMモードにした方が、試行の速度が10~100倍大きくなる。ただし、Ubuntu 20.04では、対象のllvm/clangのバージョンをインストールできない(?)ため、LLVMモードを使用できない。
```
sudo apt update
sudo apt install llvm-17 clang-17
cd /opt/AFLplusplus
sudo make
sudo make install
export CC=afl-clang-fast
export CXX=afl-clang-fast++
```
