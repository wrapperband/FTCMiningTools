
***[Guide] How to Mine crypto-currency with Linux on AMD Graphics cards GPU, how to install Sgminer v.5.1.2***

This guide is for GPU mining with Sgminer 5.1.2 on GNU/Linux (Ubuntu 14.04.0x LTS).  It is set up to mine Feathercoin, which uses the neoscrypt algorythm to encrypt each transaction, but will work with multiple algorithms from other alternate currencies, with an appropriate Sminer config file set-up.

Sgminer 5.1.2 is the same as version 5.1.1, except it has has some additional neoscrypt kernels which kave been let out in the wild and the option to return to version one, which is better with some cards. 

See "forked from" on Github, if you want to use the guide on the sgminer-dev source instead. 

In general the performance and complexity of GPU mineing comes down to the AMD drivers and lack of support for all previous cards versions in each release. here are also problems where certain versions of the AMD drivers will not work with versions of linux or sgminer. 

In order make a stable install which will work with the neoscrypt algorithm, specific versions are used, 14.04 LTS Ubuntu operating system, 14.9 AMD fglrx, 2.9 AMD application SDK and version 6 of AMD's ADL SDK.

This guide has been tested on R9 2** cards and is optimised for Neoscrypt on R9 280 in the guides default configuration.

**#1. install Ubuntu 14.04.0x**  
**Open a terminal and do an update to Ubuntu**  
Code:  
sudo apt-get update  
sudo apt-get upgrade -y  

**Install build essentials and dependencies**  
Code:  
sudo apt-get install -y git curl unzip automake autogen yasm autoconf dh-autoreconf build-essential pkg-config openssh-server screen libtool libcurl4-openssl-dev libtool libncurses5-dev libudev-dev gdebi gedit execstack dh-modaliases lib32gcc1 dkms  
  
sudo apt-get install -y xserver-xorg-core xserver-xorg-video-ati  

**#2. Manually Install correct AMD Catalyst drivers (14.9)**  
Code:  
sudo apt-get install linux-headers-generic  
sudo apt-get purge 'fglrx*'  
sudo rm /etc/X11/xorg.conf  
sudo apt-get install --reinstall -y xserver-xorg-core  
sudo dpkg-reconfigure xserver-xorg  
  
**Download 14.9 driver from AMD :**  
http://support.amd.com/en-us/download/desktop/previous/detail?os=Linux%20x86&rev=14.9  
Code:  
cd ~/  
mkdir fglrx4.9  
  
**Extract the AMD driver installer**  
Extract the files and folders from download of the AMD Catalyst™ 14.9 Proprietary Linux x86 Display Driver zip and copy / extract them into the fglrx4.9 directory   
  
run the install commands, after it finishes press exit, then install on the pop up.  
Code:  
cd  fglrx4.9  
sudo sh *.run  
  
**Initialise the graphics card, install xorg and dependencies.**  
Code:  
sudo aticonfig --adapter=all --initial  
sudo aptitude install -yr boinc-amd-opencl opencl-headers mesa-utils libglu1-mesa libgl1-mesa-glx libgl1-mesa-dri  
  
**#3. Reboot**  
  
**#4. install AMD App SDK**  
Code: 
cd ~/ 
mkdir  amd-app-sdk  
  
**Download APP-SDK from AMD :**  
http://developer.amd.com/tools-and-sdks/opencl-zone/amd-accelerated-parallel-processing-app-sdk/  
make sure it is version 2.9 AMD-APP-SDK-v2.9-lnx64.tgz extract it into /amd-app-sdk  
Code:  
cd amd-app-sdk  
chmod a+x *.sh  
sudo ./Install-AMD-App.sh  
sudo aticonfig --adapter=all --initial  
  
**#5. Reboot**  
  
**#6. Install sgminer 5.1.2**  
Code  
cd ~/  
git clone https://github.com/wrapperband/sgminer.git  
  
**#Download AMD/ADL_SDK_6.0.zip, extract the 'h files from include to ~/sgminer/adl_sdk**  
http://developer.amd.com/tools-and-sdks/graphics-development/display-library-adl-sdk/  
Make sure it is the version ADL_SDK_6.0
Code:  
cd ~/sgminer  
git submodule init  
git submodule update  
autoreconf -i -f  
CFLAGS="-O2 -Wall -march=native -std=gnu99" ./configure  
make  
(sudo make install  : optional install)  
  
**Check that the build worked**  
Code:  
./sgminer -n  
export DISPLAY=:0  
export GPU_MAX_ALLOC_PERCENT=100  
export GPU_USE_SYNC_OBJECTS=1  
./sgminer  

**#7. Create a sgminer start script**  
**#Create the script file to start sgminer correctly**  
Code:  
nano sgminer.sh   

**Copy in the script code and customise as required :**  
#!/bin/bash -         
#title           :sgminer.sh  
#description     :Start sgminer  
export DISPLAY=:0  
export GPU_MAX_ALLOC_PERCENT=100  
export GPU_USE_SYNC_OBJECTS=1  
./sgminer  

**#Make the bash script runnable**  
Code:  
chmod a+x *.sh  
  
**#8. Run the sgminer script**  
Code:  
cd ~/sgminer  
./sgminer.sh  
  
The different coin algorithms are stored in the sgminer/kernels directory. The kernels are created in C for AMD cl graphics processing programming interface. Esencially the GPU speeds up over CPU by parallel processing.  
  
There are four neoscypts available in the 5.1.2, neoscrypt.cl, neoscrypt.v1 and neoscrypt.v2 are marked as such and version 3 which is "optimised" for R9 280 (default neoscrypt kernel) and marked as neoscrypt.280.   
  
There is also a 79XX kernel version, with some of the 290 code replaced to handle older cards, which is included as a separate file neoscrypt.7690.  
It is worth experimenting to see which kernel works best with your setup, copy one of the neoscrypt files then run a Sgminer recompile.  

**Recompile sgminer after changing the kernel**  
Code:  
cd ~/sgminer  
rm *.bin  
autoreconf -i -f   &&  CFLAGS="-O2 -Wall -march=native -std=gnu99" ./configure && make && ./sgminer.sh  
  
The configuration file sgminer.conf is stored in the .sgminer hidden directory. Use Ctr-H in Gnome / Nautilus or Alt . in KDE / Dolphin, to show the hidden directories in the home directories.  
  
I have experimented and read about xIntensity, it made no difference to adjust that from 3 for neoscrypt. It gave exactly the same results with raw intensity of 5690.  
  
**An example sgminer.conf pointing towards p2pools, just replace the address with your own.**  
Code :  
nano ~/sgminer/sgminer.conf  
  
{  
  "pools": [  
    {  
      "name": "Neoscrypt Pool2P",  
      "url": "stratum+tcp://p2pool.neoscrypt.de:19327",  
      "user": "ftc address",  
      "pass": "password",  
      "no-extranonce": true  
       "priority": "1"  
    },  
    {  
      "name": "kosmoplovci Pool2P",  
      "url": "stratum+tcp://p2pool.kosmoplovci.org:19327",  
      "user": "ftc address",  
      "pass": "password",  
      "no-extranonce": true,  
      "priority": "2"  
    }  
  ]  ,
  "profiles": [],  
  "failover-only": true,  
  "algorithm": "neoscrypt",  
  "device": "all",  
  "xintensity": "3",  
  "thread-concurrency": "8192",  
  "worksize": "32",  
  "gpu-threads": "2",  
  "temp-cutoff": "95",
  "temp-overheat": "85",  
  "temp-target": "75",  
  "gpu-memdiff": "0",  
  "shares": "0",  
  "kernel-path": "/usr/local/bin",  
  "api-mcast-port": "4028",  
  "api-port": "4028",  
  "expiry": "12",  
  "failover-switch-delay": "60",  
  "gpu-dyninterval": "7",  
  "gpu-platform": "-1",  
  "hamsi-expand-big": "4",  
  "keccak-unroll": "0",  
  "log": "5",  
  "no-pool-disable": true,  
  "no-client-reconnect": true, 
  "queue": "0",  
  "scan-time": "5",  
  "tcp-keepalive": "30",  
  "temp-hysteresis": "3",  
  "watchpool-refresh": "30"  
}  
  