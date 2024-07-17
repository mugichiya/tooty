# Dillo+ & libpngのカバレッジ

## TL;DR
- Dillo+によって開いたPNGのテストケースが網羅するコードカバレッジを計測する
- 計測するカバレッジは、Dillo+とlibpng
- Raspberry Piでないとできなかった

## Dillo+のビルド環境構築
```
$ sudo apt update
$ cd /home/user/Desktop
$ git https://github.com/crossbowerbt/dillo-plus
$ cd ./dillo-plus
$ sudo apt install -y libiconv-hook1 libfltk1.3-dev libglx-dev libgl-dev libopengl-dev libssl-dev libxrender-dev libxfixes-dev libxcursor-dev libxext-dev libxinerama-dev libfontconfig1-dev libxft-dev libfontconfig-dev
$ sudo apt install -y libjpeg-dev
$ emacs ./Makefile.options
　(L18)  CFLAGS ?= -g -O2 -fprofile-arcs -ftest-coverage -coverage
```

## libpngのビルド
```
$ sudo apt remove -y --purge libpng-dev
```
最初にlibpng-devを消すと、おまけでlibfontconfig-dev libxft-devgも一緒にRemoveされる。
なので、後でソースコードからビルドする。

```
$ cd ../
$ sudo apt autoremove
$ git clone https://github.com/pnggroup/libpng
$ cd ./libpng
$ ./configure
$ emacs Makefile
  (L616) CFLAGS = -g -O2 -fprofile-arcs -ftest-coverage -coverage
$ make
$ ls ./.libs
$ sudo make install
```
make installした後に、libpngを使用したターゲットを実行するとラズパイの場合は*.gcdaが生成される。
Intel CPU環境のUbuntuでも試したが、*.gcdaは生成されなかった。

## libfontconfigとlibxftのインストール
libpng-devと一緒に消されたライブラリは、freetype2→libfontconfig→libxftの順番でインストールする。

### freetype2のインストール
```
$ cd ../
$ git clone https://gitlab.freedesktop.org/freetype/freetype
$ cd ./freetype
$ sudo apt install -y libtool automake autoconf
$ ./autogen.sh
$ make
$ sudo make install
```

### libfontconfigのインストール
```
$ cd ../
$ git clone https://gitlab.freedesktop.org/fontconfig/fontconfig
$ cd fontconfig
$ sudo apt install -y gperf autopoint
$ ./autogen.sh
$ make
$ sudo make install
```

### libxftのインストール
```
$ cd /home/user/Desktop
$ git clone https://gitlab.freedesktop.org/xorg/lib/libxft
$ cd ./libxft
$ sudo apt install -y xutils-dev
$ ./autogen.sh
$ sudo make install
```

## やっとDillo+のビルド
```
$ cd /home/user/Desktop/dillo-plus
$ make
$ ./src/dillo
```

## カバレッジ情報の整理
テストケースを実行したら、以下のようにカバレッジを計測する。
```
$ cd /home/user/Desktop/dillo-plus/src
$ lcov -c -d . -o cov.info -rc lcov_branch_coverage=1
$ genhtml -o cov cov.info　--branch-coverage
```

libpngのカバレッジを確認する場合は以下の通り。
```
$ cd /home/user/Desktop/libpng/.libs
$ lcov -c -d . -o cov.info -rc lcov_branch_coverage=1
$ genhtml -o cov cov.info　--branch-coverage
```
## 後処理
テストが終わったらアンインストールを忘れないこと。
```
$ cd /home/user/Desktop
$ cd ./libxft
$ sudo make uninstall
$ cd ../fontconfig
$ sudo make uninstall
$ cd ../freetype
$ sudo make uninstall
$ cd ../libpng
$ sudo make uninstall
$ sudo apt install -y libpng-dev
```
