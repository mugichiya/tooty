#

【注意】 Ubuntu/RaspberryPiでないと*.gcdaが生成されない

$ sudo apt update
$ sudo raspi-config
<Finish>

$ cd /home/user/Desktop
$ git https://github.com/crossbowerbt/dillo-plus
$ cd ./dillo-plus
$ sudo apt install -y libiconv-hook1 libfltk1.3-dev libglx-dev libgl-dev libopengl-dev libssl-dev libxrender-dev libxfixes-dev libxcursor-dev libxext-dev libxinerama-dev libfontconfig1-dev libxft-dev libfontconfig-dev
$ sudo apt install -y libjpeg-dev
$ emacs ./Makefile.options
　(L18)  CFLAGS ?= -g -O2 -fprofile-arcs -ftest-coverage -coverage

$ cd ../
$ sudo apt remove -y --purge libpng-dev
$ sudo apt autoremove
$ git clone https://github.com/pnggroup/libpng
$ cd ./libpng
$ ./configure
$ emacs Makefile
  (L616) CFLAGS = -g -O2 -fprofile-arcs -ftest-coverage -coverage
$ make
$ ls ./.libs
$ sudo make install

$ cd ../
$ git clone https://gitlab.freedesktop.org/freetype/freetype
$ cd ./freetype
$ sudo apt install -y libtool automake autoconf
$ ./autogen.sh
$ make
$ sudo make install

$ cd ../
$ git clone https://gitlab.freedesktop.org/fontconfig/fontconfig
$ cd fontconfig
$ sudo apt install -y gperf autopoint
$ ./autogen.sh
$ make
$ sudo make install

$ cd /home/user/Desktop
$ git clone https://gitlab.freedesktop.org/xorg/lib/libxft
$ cd ./libxft
$ sudo apt install -y xutils-dev
$ ./autogen.sh
$ sudo make install

$ cd /home/user/Desktop/dillo-plus
$ make
$ ./src/dillo


$ cd /home/user/Desktop/dillo-plus/src
$ lcov -c -d . -o cov.info -rc lcov_branch_coverage=1
$ genhtml -o cov cov.info　--branch-coverage

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
