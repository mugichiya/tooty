# lcovによるカバレッジ測定方法

## TL;DR
- このマークダウンは、gcov/lcovを用いたカバレッジ計測の備忘録
- Ubuntuでは下記の手順で測定可能

## インストール
```
$ sudo apt install lcov
```

## カバレッジ情報抽出のオプション
CFLAGSとして、`-fprofile-arcs -ftest-coverage -coverage`を追加する。
例えば、libjpeg-turboの場合は以下の通り。
```
$ cd /home/user/Desktop
$ git clone https://github.com/libjpeg-turbo/libjpeg-turbo
$ cd ./libjpeg-turbo
$ emacs ./CMakeLists.txt
  (L02) set(CMAKE_C_FLAGS ${CMAKE_C_FLAGS} "-fprofile-arcs -ftest-coverage -coverage")
$ mkdir build
$ cd ./build
$ cmake ../
$ make
```

また、dillo-plusのビルドを例とした場合は以下の通り。
```
$ cd /home/user/Desktop
$ git https://github.com/crossbowerbt/dillo-plus
$ sudo apt install libiconv-hook1 libfltk1.3-dev libglx-dev libgl-dev libopengl-dev libssl-dev libxrender-dev libxfixes-dev libxcursor-dev libxext-dev libxinerama-dev libfontconfig1-dev libxft-dev libfontconfig-dev
$ cd ./dillo-plus
$ emacs ./Makefile.options
　(L18)  CFLAGS ?= -g -O2 -fprofile-arcs -ftest-coverage -coverage
  (L20)  INCLUDES ?= -I. -I.. -I/usr/local/include -I/home/user/Desktop/libjpeg-turbo -I/home/user/Desktop/libjpeg-turbo/build
  (L21)  LDFLAGS ?= -L/usr/local/lib -L/home/user/Desktop/libjpeg-turbo -L/home/user/Desktop/libjpeg-turbo/build
$ make
```

## テストケースの実行
以下のようにライブラリの場所を指定して起動し、適当にjpegファイルを開く。

```
$ LD_LIBRARY_PATH=/home/user/Desktop/libjpeg-turbo/build ./src/dillo
```

## カバレッジ情報の整理
ビルド時に*.gcnoが生成され、テストケースを実行すると*.gcdaが生成される。
これらのファイルがあるパスで以下のようにlcovを実行する。
genhtmlでcovというフォルダができる。その中のindex.htmlをブラウザで開く。

```
$ cd /home/user/Desktop/dillo-plus/src
$ lcov -c -d . -o cov.info -rc lcov_branch_coverage=1
$ genhtml -o cov cov.info　--branch-coverage
```

libjpeg-turboのカバレッジを確認する場合は以下の通り。
```
$ cd /home/user/Desktop/libjpeg-turbo/build/sharedlib/CMakeFiles/jpeg.dir/__/
$ lcov -c -d . -o cov.info -rc lcov_branch_coverage=1
$ genhtml -o cov cov.info　--branch-coverage
```
## 注意
*.gcdaの生成や追記されるタイミングは、テストケースを実行して**正常終了**した時。
なので、`ctrl+c`やオーバーフローしたものはカウントされない。
特に、ブラウザを開いたまま何万件とテストケースを実行して、`Ctrl+c`をしてしまったときがキツイ。
なぜならば、そこまで実行させたテストケースのカバレッジは一切書き込まれない。
