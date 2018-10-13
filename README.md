AntMiner S7
===========

Accessing the S7 is easy : ssh root@......, password = "admin". Accessing the web interface is also easy : root/root (because of the specific setup at ULille, this can be done by : ssh caza.lifl.fr -N -L 8000:mineur.lifl.fr:80, then target a web browser to local port 8000).

The AntMiner S7, a customized version of CGminer is installed. Fortunately, the code of this modified version is online in https://github.com/bitmaintech/cgminer, but beware! The code running on the S7 corresponds to the "bitmain_fan-ctrl" branch. It has code specific to the S7.

On the S7, a custom shell script monitors cgminer and restarts it if it stops. This script is also restarted if it exists!

When the S7 reboots, its state is restored to factory settings. This allows us to play without risk.

So, to **actually** stop CGminer on the S7, a possibility is to run the following script (scripts/relax.sh):

#!/bin/bash
rm /sbin/monitorcg
killall monitorcg
killall cgminer



Cross-compiling CGminer
=======================

The trick is to build a modified version of (the modified version of) CGminer. Compiling on the S7 is a PITA, so we cross-compile locally.

Our version of CGminer is accessible at https://github.com/cbouilla/cgminer. We removed nearly everything not useful to us. To cross-compile :

#!/bin/bash
CFLAGS="-g -O2 -Wall -Wextra" ./configure --host=arm-linux-gnueabihf --enable-bitmain --without-curses --disable-libcurl --disable-udev
make clean
make

One nasty caveat: we chose (this may have been a bad decision) to use 0MQ as our network middleware. This is a C++ library. We coulc cross-compile it, but we ran into a problem: our (local) version of the libc is much more recent than that on the S7 (eglibc...). The cross-compiled 0MQ wouldn't work. Compiling 0MQ on the S7 itself wouldn't work (not enough RAM). Workaround: use an image of the S7 found online as a "firmware upgrade" (http://diyhpl.us/~bryan/irc/bitcoin/bitmain-firmware/), mount it locally, use QEMU to pretend we're an ARM CPU, build, keep the built library.

The built library is in the "built_0MQ_lib" folder. Its content has to be copied to "/usr/" on the S7.

It relies on the C++ standard library (that's why C++ might have been a bad decision). We can install it on the S7, though. The simplest way is to copy the "opkg/libstdc++...." file to the miner, then run "opkg install libstdc++...." on the miner.


Running the modifier CGminer on the S7
======================================

Once 0MQ and the C++ standard library have been "installed" on the S7, the cross-compiled cgminer binary copied to /root, and the "normal" cgminer stopped, then we may safely run our own (script/run.sh):

LD_LIBRARY_PATH=/usr/local/lib ./cgminer --bitmain-dev /dev/bitmain-asic --bitmain-options 115200:32:8:5:700:0782:0706  --bitmain-hwerror --queue 8192

Of course, we must run this in a screen (otherwise it stops on SSH disconnect). Screen complains about our terminal type, so let's fool it:

export TERM=vt100
screen

This tries to connect to the "FOOBAR" server (which is legitimate). It must be setup.


Running the "FOOBAR" work server
================================

From a fresh (debian) AWS instance, install :

apt-get install build-essentials libzmq3-dev

tangle and upload server.c to the server, build it locally.

gcc -O3 -Wall -Wextra server.c -o server -lzmq

screen
./server --kind 0

