# Ubuntu to RaspberryPI Cross Compile and Debug using Eclipse
## Introduction
Some time ago I have encountered the need to develop similarly straighforward code for Raspberyy Pi platform. After a little bit of back and forth considerations between python and C++, I have chosen to experiment with C++ option.
The next iteration of decisions was whether to do development on the Raspberry Pi platform itself, or on the Ubuntu box and then cross compile code for Raspberry PI. My choice was to do development on Ubuntu, since Ubuntu is much more powerfull machine than Raspberry Pi.
In terms of IDE, my default choice is Eclipse. Obviously, there are many great IDEs out there - Visual Studio, Clion to name a few, but I like Eclipse. Last but not the least - we will use CMake.
## Acknowledgments. 
This is derived work inspired, primarily, by [this excellent guide](https://tttapa.github.io/Pages/Raspberry-Pi/C++-Development-RPiOS/) and many other distinguished contributors. Where possible, I will provide links to the original source, so that the reader can investigate further on their own, if required.
## Preliminaries
In this repository, we will reference to the **host** machine - Ubuntu box, which we will use for all types of code development, and **target** machine - Raspberry PI box - this is where we want to run the code (eventually).
### Hardware
#### Windows Auxiliary machine
For reasons completely ouside of the scope of this repository, we will be using occasionally Windows machine. Under different circumstances, we may as well use only Ubuntu box.
#### Host machine
Ubuntu 22.04.1 LTS. well, may be this is not exactly description of the hardware, my feeling is that it is not really importnat to mention anyhting about **host** hardware at this junction.
#### Target machine
[GeeekPi Raspberry Pi 4 8GB Starter Kit](https://www.amazon.com/gp/product/B09LYP7QH3/ref=ppx_yo_dt_b_search_asin_title?ie=UTF8&psc=1) with [Seeed Studio ReSpeaker 4-Mic Array for Raspberry Pi, Raspberry Pi Microphone](https://www.amazon.com/gp/product/B076SSR1W1/ref=ppx_yo_dt_b_search_asin_title?ie=UTF8&psc=1) and [GPS Module GPS NEO-6M](https://www.amazon.com/gp/product/B07P8YMVNT/ref=ppx_yo_dt_b_search_asin_title?ie=UTF8&psc=1). Longer term plans include subsequent development using this equipment.
### Software
#### Windows Auxiliary machine
Using Raspberry Pi Imager version 1.7.3 we install Raspberry PI 64 bit image onto 64 GB [SanDisk Micro SD memory card](https://www.amazon.com/gp/product/B08MWW8MVY/ref=ppx_yo_dt_b_search_asin_title?ie=UTF8&psc=1):
![Raspberry PI Image Installation]
#### Target machine
With newly burned image, we insert Micro SD memory card into the slot on the Raspberry Pi box. We also connect Terminal, mouse and keyboard, power up and follow the motions. We will omit describing the routine steps, like WiFi set up, updates and so on, we will mention only important ones.
  * Default User Name and Password - Instead of the default user name **pi** and default password **raspberry** we are going to use user name **voldemort** with corresponding password, to illustrate the maturity of the author of this repo.
  * Headless Resolution - since most of the development will be done on the **host** machine, we are going to use VNC to connect to the Raspberry Pi box, and to support operation with display connected, set up it to appropriate value (like 1920 x 1080) using **Raspberry Pi Configuration** utility.
  * Enable **SSH** and **VNC** interfaces using **![Raspberry Pi Configuration]** utility.
  * Setting up **VNC** interface requires [additional steps](https://superuser.com/questions/1532602/remmina-unknown-authentication-scheme-from-vnc-server-13-5-6-130-192) for me, since I am getting error message "Unknown authentication scheme from VNC server: 13, 5, 6, 130, 192" (I am using Remmina application on Ubuntu **host** machine):
    `sudo vncpasswd -service` 
   
     Now, edit the file 
     `/root/.vnc/config.d/vncserver-x11`
     using this command
     `sudo nano /root/.vnc/config.d/vncserver-x11`
     and add the following line at the end of the file:
     `Authentication=VncAuth`
    
     Now your config file should look more or less like mine:

   ```
     _AnlLastConnTime=int64:0000000000000000
     _LastUpdateCheckSuccessTime=int64:01d9404a78664d28
     _LastUpdateCheckTime=int64:01d9404a78664d28
     Password=1996ddb0ce9442feb79a4a14dcb08a2c
     Authentication=VncAuth
   ``` 
   Eventually, restart the VNC server service with `sudo systemctl restart vncserver-x11-serviced` command
   ##### Seeed Voice Card Driver Installation
   As we mentioned in the **Hardware** section, we are using Seeed Voice Card, and we need to install the drivers:
   ```
   sudo apt-get update
   sudo apt-get upgrade
   mkdir ~/Git
   cd ~/Git
   ```
   Now, we were told to use @HinTak [repo](https://github.com/HinTak/seeed-voicecard). So we do:
   
   ```
   git clone https://github.com/HinTak/seeed-voicecard.git
   cd seeed-voicecard/
   sudo ./install.sh   
   ```
   with some luck, the installation script will go through, and you will see this message at the end of the script execution:
   ```
   Initialized empty Git repository in /etc/voicecard/.git/
git add --all
git commit -m "origin configures"
[master (root-commit) a09f22b] origin configures
 7 files changed, 1482 insertions(+)
 create mode 100644 ac108_6mic.state
 create mode 100644 ac108_asound.state
 create mode 100644 asound_2mic.conf
 create mode 100644 asound_4mic.conf
 create mode 100644 asound_6mic.conf
 create mode 100644 dkms.conf
 create mode 100644 wm8960_asound.state
Created symlink /etc/systemd/system/sysinit.target.wants/seeed-voicecard.service → /lib/systemd/system/seeed-voicecard.service.
------------------------------------------------------
Please reboot your raspberry pi to apply all settings
Enjoy!
------------------------------------------------------

   ```
Worth to mention, that in my case menu bar disappears. Apparently this is [known phenomenon](https://github.com/respeaker/seeed-voicecard/issues/319), but still a bit [annoying](https://forum.seeedstudio.com/t/lxde-panel-removed-when-installing-respeaker-microphone-drivers/266045). One possible solution to this problem could be located [here](https://raspberrypi.stackexchange.com/questions/122579/after-fresh-install-of-raspberry-os-the-menu-bar-is-missing-in-tightvnc-session):
```
sudo apt remove lxplug-volumepulse
```

Nevertheless, the installation was successful, in a sense, that we can see the installation of the audio card:
```
 arecord -L
null
    Discard all samples (playback) or generate zero samples (capture)
lavrate
    Rate Converter Plugin Using Libav/FFmpeg Library
samplerate
    Rate Converter Plugin Using Samplerate Library
speexrate
    Rate Converter Plugin Using Speex Resampler
jack
    JACK Audio Connection Kit
oss
    Open Sound System
pulse
    PulseAudio Sound Server
upmix
    Plugin for channel upmix (4,6,8)
vdownmix
    Plugin for channel downmix (stereo) with a simple spacialization
default
    Playback/recording through the PulseAudio sound server
playback
ac108
usbstream:CARD=Headphones
    bcm2835 Headphones
    USB Stream Output
usbstream:CARD=vc4hdmi0
    vc4-hdmi-0
    USB Stream Output
usbstream:CARD=vc4hdmi1
    vc4-hdmi-1
    USB Stream Output
hw:CARD=seeed4micvoicec,DEV=0
    seeed-4mic-voicecard, bcm2835-i2s-ac10x-codec0 ac10x-codec0-0
    Direct hardware device without any conversions
plughw:CARD=seeed4micvoicec,DEV=0
    seeed-4mic-voicecard, bcm2835-i2s-ac10x-codec0 ac10x-codec0-0
    Hardware device with all software conversions
sysdefault:CARD=seeed4micvoicec
    seeed-4mic-voicecard, bcm2835-i2s-ac10x-codec0 ac10x-codec0-0
    Default Audio Device
dsnoop:CARD=seeed4micvoicec,DEV=0
    seeed-4mic-voicecard, bcm2835-i2s-ac10x-codec0 ac10x-codec0-0
    Direct sample snooping device
usbstream:CARD=seeed4micvoicec
    seeed-4mic-voicecard
    USB Stream Output

```

#### Host machine
##### SYSROOT
First of all we need to create sysroot folder on the local machine. So far I have found two methods to do this: 
  * The first one is to create version of Raspberry Pi OS in a folder on your computer, like how it is advised [here](https://tttapa.github.io/Pages/Raspberry-Pi/C++-Development-RPiOS/Development-setup.html) or [there](https://forums.raspberrypi.com/viewtopic.php?t=343710#p2059499). While advertised advantages of this method are very attractive and elegant, unfortunatelly, this method did not work for me.
  * The second [method](https://raspberrypi.stackexchange.com/questions/108351/cross-compiling-and-sysroot) is more like brute force approach. In this method, we create `sysroot` folder on the **host** machine, and manually rsync to copy files from the **target**:
```
mkdir sysroot
cd sysroot
sudo rsync -avz --rsync-path="sudo rsync" voldemort@raspberrypi_ip:/etc .
sudo rsync -avz voldemort@raspberrypi_ip:/lib .
sudo rsync -avz voldemort@raspberrypi_ip:/usr/include usr
sudo rsync -avz voldemort@raspberrypi_ip:/usr/lib usr
sudo rsync -avz voldemort@raspberrypi_ip:/usr/local usr
sudo rsync -avz voldemort@raspberrypi_ip:/home/voldemort/.local/ .local
```
Attentive reader may notice that here I have few more lines as compared to the original post. This is becasue we need to repair absolute symlinks with relative ones, and some of the symlinks are pointing to the `/etc` location, for example:
```
138307961 lrwxrwxrwx  1 root root       48 Sep 21 19:52 libblas.so.3 -> /etc/alternatives/libblas.so.3-aarch64-linux-gnu
138308251 lrwxrwxrwx  1 root root       50 Sep 21 19:52 liblapack.so.3 -> /etc/alternatives/liblapack.so.3-aarch64-linux-gnu
```
hence we need to 
```
sudo rsync -avz --rsync-path="sudo rsync" voldemort@raspberrypi_ip:/etc .
```
Assuming we followed the [steps](https://raspberrypi.stackexchange.com/questions/108351/cross-compiling-and-sysroot) we have `sysroot-relativelinks.py` code available, so we execute it:
```
./sysroot-relativelinks.py sysroot
```
So now we see that absolute links are replaced with relative ones:
```
138307961 lrwxrwxrwx  1 root root       56 Feb 16 00:18 libblas.so.3 -> ../../../etc/alternatives/libblas.so.3-aarch64-linux-gnu
138308251 lrwxrwxrwx  1 root root       58 Feb 16 00:18 liblapack.so.3 -> ../../../etc/alternatives/liblapack.so.3-aarch64-linux-gnu
```
##### Toolchain
Absolutely religiously following steps from [this repository](https://github.com/tttapa/docker-arm-cross-toolchain) to install [aarch64-rpi3-linux-gnu](https://github.com/tttapa/docker-arm-cross-toolchain/releases/latest/download/x-tools-aarch64-rpi3-linux-gnu.tar.xz) (64-bit, RPi 2B rev. 1.2, RPi 3B/3B+, CM 3, RPi 4B/400, CM 4, RPi Zero 2 W) toolchain.

##### IDE Configuration
As we have metnioned before, our IDE of choice is **Eclipse**. Now, I can imagine a lot of people will come up with multiple interesting comments about merits of various different platforms, and how Eclipse is lacking and not modern and gross. And I agree vigorously.

###### Create New Project
File->New->Other->C++ Project<br>
![Select a Wizard]
<br>Press **Next**<br>
Name the Project <br>
![C++ Project]


[Raspberry PI Image Installation]: https://github.com/vslobody/Ubuntu-to-Raspberry-PI-Cross-Compile-and-Debug-using-Eclipse-/blob/main/src/common/images/RaspberryPIImage.png

[Raspberry Pi Configuration]: https://github.com/vslobody/Ubuntu-to-Raspberry-PI-Cross-Compile-and-Debug-using-Eclipse-/blob/main/src/common/images/remmina_Raspberry_192.168.1.205_20230214-082127.png

[Select a Wizard]: https://github.com/vslobody/Ubuntu-to-Raspberry-PI-Cross-Compile-and-Debug-using-Eclipse-/blob/main/src/common/images/SelectAWizard.png
[C++ Project](https://github.com/vslobody/Ubuntu-to-Raspberry-PI-Cross-Compile-and-Debug-using-Eclipse-/blob/main/src/common/images/CPlusPlusProject.png)
