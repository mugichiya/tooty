# Ubuntuでファジングの環境構築

## 目次
+ ファジングの環境構築
+ カスタムイメージの作成から配布まで


## ベースとするOSの起動

+ Ubuntu16.04を起動

```
$ docker run -it -d --name ubuntu ubuntu:16.04
4c19c0672708dac37b42faa3198153d484eae371750572f068a16fc2e0b623e5
$ docker exec -it ubuntu bash
root@4c19c0672708:/# apt update
root@4c19c0672708:/# apt upgrade
```

## AFLのインストール

### ノーマルモード

```
root@4c19c0672708:/# apt install -y make gcc git gnuplot
root@4c19c0672708:/# git clone https://github.com/google/AFL.git /opt/AFL
root@4c19c0672708:/# make -C /opt/AFL/
```

### QEMUモード

```
root@4c19c0672708:/# apt install -y libtool libtool-bin wget python automake bison libglib2.0-dev
root@4c19c0672708:/# cd /opt/AFL/qemu_mode/
root@4c19c0672708:/opt/AFL/qemu_mode# ./build_qemu_support.sh
```


### LLVMモード(clang-3.x, llvm-3.x)

```
root@4c19c0672708:/# apt install -y llvm clang
root@4c19c0672708:/# make -C /opt/AFL/llvm_mode/
```

### /usr/local/bin/にインストール

```
root@4c19c0672708:/# make install -C /opt/AFL/
```

## ファジングのサンプル

### /home内にプロジェクトフォルダを作成

```
root@4c19c0672708:/# mkdir -p /home/proj0x/in
root@4c19c0672708:/# mkdir -p /home/proj0x/out
root@4c19c0672708:/# mkdir -p /home/proj0x/src
root@4c19c0672708:/# mkdir -p /home/proj0x/graph
```

### ノーマルモード

```
root@4c19c0672708:/# cp /opt/AFL/test-instr.c /home/proj01/src/
root@4c19c0672708:/# cd /home/proj01/
root@4c19c0672708:/home/proj01# afl-gcc -o test-instr ./src/test-instr.c 
afl-cc 2.54b by <lcamtuf@google.com>
afl-as 2.54b by <lcamtuf@google.com>
[+] Instrumented 6 locations (64-bit, non-hardened mode, ratio 100%).
root@4c19c0672708:/home/proj01# echo aaa > ./in/seed
root@4c19c0672708:/home/proj01# afl-fuzz -i ./in/ -o ./out/ -- ./test-instr 
```

### サンプル（QEMUモード）

```
root@4c19c0672708:/# cp /opt/AFL/test-instr.c /home/proj02/src/
root@4c19c0672708:/# cd /home/proj02/
root@4c19c0672708:/home/proj02# gcc -o test-instr ./src/test-instr.c 
root@4c19c0672708:/home/proj02# echo aaa > ./in/seed
root@4c19c0672708:/home/proj02# afl-fuzz -Q -m none -i ./in/ -o ./out/ -- ./test-instr
```

### サンプル（LLVMモード）

```
root@4c19c0672708:/# cp /opt/AFL/test-instr.c /home/proj03/src/
root@4c19c0672708:/# cd /home/proj03/
root@4c19c0672708:/home/proj03# afl-clang-fast -o test-instr ./src/test-instr.c
afl-clang-fast 2.54b by <lszekeres@google.com>
afl-llvm-pass 2.54b by <lszekeres@google.com>
[+] Instrumented 6 locations (non-hardened mode, ratio 100%).
root@4c19c0672708:/home/proj03# echo aaa > ./in/seed
root@4c19c0672708:/home/proj03# afl-fuzz -i ./in/ -o ./out/ -- ./test-instr
```

### 結果の可視化とホスト共有

```
root@4c19c0672708:/# afl-plot ./proj01/out/ ./proj01/graph/
root@4c19c0672708:/# afl-plot ./proj02/out/ ./proj01/graph/
root@4c19c0672708:/# afl-plot ./proj03/out/ ./proj01/graph/
root@4c19c0672708:/# exit
exit
$ docker cp ubuntu:/home/ output
$ ls output
proj01	proj02	proj03
```

### Docker上での実行結果(macOS)

|Normal mode|QEMU mode|LLVM mode|
|:---:|:---:|:---:|
|~3900/sec |~1000/sec|~4000/sec|


### macOSでの実行結果

|Normal mode|QEMU mode|LLVM mode|
|:---:|:---:|:---:|
|~1300/sec |-|-|

```
WARNING: Fuzzing on MacOS X is slow because of the unusually high overhead of
fork() on this OS. Consider using Linux or *BSD. You can also use VirtualBox
(virtualbox.org) to put AFL inside a Linux or *BSD VM.
```

したがって，Docker上のUbuntuの方が実行速度が高い．

## コンテナからカスタムイメージを作成

```
$ docker stop ubuntu
$ docker commit ubuntu ubuntu:fuzz
sha256:a76174a3464d23d3c5e83d28a023ab64b5e261b4014457f3200b7b1248c12883
$ docker images
REPOSITORY                          TAG                 IMAGE ID            CREATED             SIZE
ubuntu                              fuzz                a76174a3464d        4 minutes ago       1.56GB
...
$ docker save ubuntu:fuzz -o ubuntu-fuzz.tar
$ ls -l ubuntu-fuzz.tar 
-rw-------  1 user  staff  1607966720 Sep 11 13:48 ubuntu-fuzz.tar
```

## カスタムイメージで配布

### 別マシンのUbuntu16.04でイメージの展開

```
# docker load -i ubuntu.tar
# docker run -it -d --name ubuntu-fuzz ubuntu:fuzz
# echo core >/proc/sys/kernel/core_pattern
# cd /sys/devices/system/cpu
# echo performance | tee cpu*/cpufreq/scaling_governor
# cd ~-
# docker exec -it ubuntu-fuzz bash
root@6d71652be8df:/# 
```

ロード後，ubuntu.tarは削除してもOK．

### Docker上での実行結果(ubuntu16.04)

|Normal mode|QEMU mode|LLVM mode|
|:---:|:---:|:---:|
|~5700/sec |~1200/sec|~5700/sec|

### Ubuntu16.04での実行結果

|Normal mode|QEMU mode|LLVM mode|
|:---:|:---:|:---:|
|~7000/sec |~1400/sec|~7200/sec|

## Docker HubへのPUSH
+ Docker Hubでリポジトリを作成
+ ローカルからPUSH

```
$ docker login
Username:
Password:
Login Succeeded
$ docker images 
REPOSITORY                          TAG                 IMAGE ID            CREATED             SIZE
ubuntu                              fuzz                a76174a3464d        2 days ago          1.56GB
...
$ docker tag a76174a3464d verifsec/ubuntu-afl:16.04
$ docker rmi ubuntu:fuzz
$ docker push verifsec/ubuntu-afl:16.04
```

+ 他のマシンからPULL

```
$ docker run -it -d --name afl verifsec/ubuntu-afl:16.04
Unable to find image 'verifsec/ubuntu-afl:16.04' locally
16.04: Pulling from verifsec/ubuntu-afl
f7277927d38a: Pull complete 
8d3eac894db4: Pull complete 
edf72af6d627: Pull complete 
3e4f86211d23: Pull complete 
1eeae5a57def: Pull complete 
Digest: sha256:a515e672d81c805539e6942c662f5516e7a08297a35b946f79ea6b0ce65f90eb
Status: Downloaded newer image for verifsec/ubuntu-afl:16.04
644d7d1e91b100d7bc59907a95a55985fed907b2ea073a10c778237a0fed9064
```

## 今後の予定
+ <strike>Dockerの基本的な操作</strike>（済）
+ <strike>Ubuntuでファジングの環境構築</strike>（済）
+ ARM向けファジング環境構築
+ クラウドプラットフォームの利用
+ OSS-fuzz
