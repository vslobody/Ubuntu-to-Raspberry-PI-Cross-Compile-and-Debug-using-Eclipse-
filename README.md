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
While the following instructions are more pertinet to the project I am pursuing, providing them here nevertheless, for completeness properties. Need to install packages that will be required down the road later.
```
sudo apt-get update
sudo apt-get install libfftw3-dev libconfig-dev libasound2-dev libgconf-2-4 libpulse-dev python3-matplotlib cmake
```
We also planning to use header only **matplolib-cpp** library:
```
git clone https://github.com/lava/matplotlib-cpp.git
cd matplotlib-cpp/
mkdir build
cd build/
cmake ..
make
sudo make install
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
We want to use **Cmake4eclipse** wizard.
Name the Project <br>
![C++ Project] <br>
Press **Next**<br>
Press **Finish**<br>
<br><br>
Now we need to create CMake file that describes the target platform, called toolchain file **aarch64-rpi3-linux-gnu.cmake**.
```
# Cross-compilation system information
SET(CMAKE_SYSTEM_NAME Linux)
SET(CMAKE_SYSTEM_VERSION 1)

set(CMAKE_SYSTEM_PROCESSOR "arm")
# Specify the cross compiler
SET(CMAKE_C_COMPILER /home/voldemaro/eclipse/raspberrypi/opt/x-tools/aarch64-rpi3-linux-gnu/bin/aarch64-rpi3-linux-gnu-gcc)
SET(CMAKE_CXX_COMPILER /home/voldemaro/eclipse/raspberrypi/opt/x-tools/aarch64-rpi3-linux-gnu/bin/aarch64-rpi3-linux-gnu-g++)

# where is the target environment located
set(CMAKE_SYSROOT /home/voldemaro/eclipse/raspberrypi/sysroot)
set(CMAKE_FIND_ROOT_PATH ${CMAKE_SYSROOT})

# SET(CMAKE_EXE_LINKER_FLAGS "${CMAKE_EXE_LINKER_FLAGS} --sysroot=${CMAKE_FIND_ROOT_PATH}")
SET(CMAKE_SHARED_LINKER_FLAGS "${CMAKE_SHARED_LINKER_FLAGS} --sysroot=${CMAKE_FIND_ROOT_PATH}")
SET(CMAKE_MODULE_LINKER_FLAGS "${CMAKE_MODULE_LINKER_FLAGS} --sysroot=${CMAKE_FIND_ROOT_PATH}")


# adjust the default behavior of the FIND_XXX() commands:
# search programs in the host environment
set(CMAKE_FIND_ROOT_PATH_MODE_PROGRAM NEVER)

# search headers and libraries in the target environment
set(CMAKE_FIND_ROOT_PATH_MODE_LIBRARY ONLY)
set(CMAKE_FIND_ROOT_PATH_MODE_INCLUDE ONLY)
```
We will add **cmake** folder to our project and put this file there.
Now we want to instruct CMake to use this file by setting the **CMAKE_TOOLCHAIN_FILE** <br>
![Add new CMake Cache Entry]
<br>
![Properties for RPI_X_Compile]
<br>
Since we want to be able to debug our code, we need to set up our project for Remote Debugging. Click **File -> New -> Other**. In the window that appears, double-click **Remote System Explorer** to expand it and select **Connection**.<br>
![Connection]
<br>
Click **Next**. Select **Linux**.
<br>
![Connection Linux]
<br>
Click **Next**. At Host name, enter the IP address of the Raspberry Pi (192.168.1.205 in the example).

At **Connection name**, enter a name for the connection (**RPI** in the example).<br>
![Connection Name]
<br>

Click **Next**. Under **Configuration**, select **ssh.files**.<br>
![New Connection]
<br>
Click **Next**. Select **processes.shell.Linux**. <br>
![New Connection 2]
<br>
Click **Next**. Select **ssh.shells**.<br>
![New Connection 3]
<br>
Click **Finish**.
In the **Remote Systems** tab, right-click the connection and select **Properties**. Set the connection's user to **voldemort**: click Host. Click the small icon just to the right of Default User ID and enter **voldemort**.<br>
![Properties for RPI]
<br>

###### Sample Code
We need to have a code to debug, aren't we? To get there, we will make a little detour. Without going into a lot of details, the original plan was to use **Raspberry Pi** platform to do some cute audio processing, so we equipped it with Seeed Audio Card [Seeed Audio Card](https://www.amazon.com/gp/product/B076SSR1W1/ref=ppx_yo_dt_b_search_asin_title?ie=UTF8&psc=1). We would like to use [odas](https://github.com/introlab/odas.git) library. So we copy it to the **odas** folder of the project. We also create sample **main.cpp** file in **src** folder and corresponding header file in the **include** folder. And, of course, we have **CMakeLists.txt** file.
We build the project
```00:31:28 Buildscript generation: RPI_X_Compile::Debug in /home/voldemaro/eclipse-workspace/RPI_X_Compile/_build/Debug
cmake -DCMAKE_BUILD_TYPE:STRING=Debug -G Ninja -DCMAKE_TOOLCHAIN_FILE:FILEPATH=/home/voldemaro/eclipse-workspace/RPI_X_Compile/cmake/aarch64-rpi3-linux-gnu.cmake -DCMAKE_EXPORT_COMPILE_COMMANDS:BOOL=ON /home/voldemaro/eclipse-workspace/RPI_X_Compile 
-- The C compiler identification is GNU 12.2.0
-- The CXX compiler identification is GNU 12.2.0
-- Detecting C compiler ABI info
-- Detecting C compiler ABI info - done
-- Check for working C compiler: /home/voldemaro/eclipse/raspberrypi/opt/x-tools/aarch64-rpi3-linux-gnu/bin/aarch64-rpi3-linux-gnu-gcc - skipped
-- Detecting C compile features
-- Detecting C compile features - done
-- Detecting CXX compiler ABI info
-- Detecting CXX compiler ABI info - done
-- Check for working CXX compiler: /home/voldemaro/eclipse/raspberrypi/opt/x-tools/aarch64-rpi3-linux-gnu/bin/aarch64-rpi3-linux-gnu-g++ - skipped
-- Detecting CXX compile features
-- Detecting CXX compile features - done
CMake Deprecation Warning at odas/CMakeLists.txt:2 (cmake_minimum_required):
  Compatibility with CMake < 2.8.12 will be removed from a future version of
  CMake.

  Update the VERSION argument <min> value or use a ...<max> suffix to tell
  CMake that the project does not need compatibility with older versions.


-- Found PkgConfig: /usr/bin/pkg-config (found version "0.29.2") 
-- Checking for module 'fftw3f'
--   Found fftw3f, version 3.3.8
-- Checking for module 'alsa'
--   Found alsa, version 1.2.6.1
-- Checking for module 'libconfig'
--   Found libconfig, version 1.5
-- Checking for module 'libpulse-simple'
--   Found libpulse-simple, version 15.99.1
-- Configuring done
-- Generating done
-- Build files have been written to: /home/voldemaro/eclipse-workspace/RPI_X_Compile/_build/Debug
00:31:31 Buildscript generation finished (took 3404 ms)
00:31:31 **** Build of configuration Debug for project RPI_X_Compile ****
/usr/bin/ninja -j 8 all 
[1/169] Building C object odas/CMakeFiles/odas.dir/src/general/format.o
[2/169] Building C object odas/CMakeFiles/odas.dir/src/general/interface.o
[3/169] Building C object CMakeFiles/RPI_X_Compile.dir/odas/demo/odaslive/profiler.c.o
[4/169] Building C object odas/CMakeFiles/odas.dir/src/general/samplerate.o
[5/169] Building C object odas/CMakeFiles/odas.dir/src/general/soundspeed.o
[6/169] Building C object odas/CMakeFiles/odas.dir/src/general/link.o
[7/169] Building C object odas/CMakeFiles/odas.dir/src/general/mic.o
[8/169] Building C object odas/CMakeFiles/odas.dir/src/general/spatialfilter.o
[9/169] Building C object odas/CMakeFiles/odas.dir/src/init/combining.o
[10/169] Building C object odas/CMakeFiles/odas.dir/src/general/thread.o
[11/169] Building C object odas/CMakeFiles/odas.dir/src/init/delay.o
[12/169] Building C object odas/CMakeFiles/odas.dir/src/init/hit.o
[13/169] Building C object odas/CMakeFiles/odas.dir/src/init/directivity.o
[14/169] Building C object odas/CMakeFiles/odas.dir/src/init/linking.o
[15/169] Building C object odas/CMakeFiles/odas.dir/src/init/scanning.o
[16/169] Building C object odas/CMakeFiles/odas.dir/src/signal/acorr.o
[17/169] Building C object odas/CMakeFiles/odas.dir/src/init/windowing.o
[18/169] Building C object odas/CMakeFiles/odas.dir/src/init/space.o
[19/169] Building C object odas/CMakeFiles/odas.dir/src/signal/aimg.o
[20/169] Building C object odas/CMakeFiles/odas.dir/src/signal/area.o
[21/169] Building C object odas/CMakeFiles/odas.dir/src/signal/assignation.o
[22/169] Building C object odas/CMakeFiles/odas.dir/src/signal/beampattern.o
[23/169] Building C object odas/CMakeFiles/odas.dir/src/signal/category.o
[24/169] Building C object odas/CMakeFiles/odas.dir/src/signal/coherence.o
[25/169] Building C object odas/CMakeFiles/odas.dir/src/signal/delta.o
[26/169] Building C object odas/CMakeFiles/odas.dir/src/signal/demixing.o
[27/169] Building C object odas/CMakeFiles/odas.dir/src/signal/env.o
[28/169] Building C object CMakeFiles/RPI_X_Compile.dir/odas/demo/odaslive/configs.c.o
[29/169] Building C object odas/CMakeFiles/odas.dir/src/signal/frame.o
[30/169] Building C object odas/CMakeFiles/odas.dir/src/signal/freq.o
[31/169] Building C object odas/CMakeFiles/odas.dir/src/signal/gain.o
[32/169] Building C object CMakeFiles/RPI_X_Compile.dir/odas/demo/odaslive/threads.c.o
[33/169] Building C object odas/CMakeFiles/odas.dir/src/signal/index.o
[34/169] Building C object CMakeFiles/RPI_X_Compile.dir/odas/demo/odaslive/objects.c.o
[35/169] Building C object CMakeFiles/RPI_X_Compile.dir/odas/demo/odaslive/parameters.c.o
[36/169] Building C object odas/CMakeFiles/odas.dir/src/signal/hop.o
[37/169] Building C object odas/CMakeFiles/odas.dir/src/signal/kalman.o
[38/169] Building C object odas/CMakeFiles/odas.dir/src/signal/map.o
[39/169] Building C object odas/CMakeFiles/odas.dir/src/signal/mask.o
[40/169] Building C object odas/CMakeFiles/odas.dir/src/signal/mixture.o
[41/169] Building C object odas/CMakeFiles/odas.dir/src/signal/pot.o
[42/169] Building C object odas/CMakeFiles/odas.dir/src/signal/pitch.o
[43/169] Building C object odas/CMakeFiles/odas.dir/src/signal/pair.o
[44/169] Building C object odas/CMakeFiles/odas.dir/src/signal/particle.o
[45/169] Building C object odas/CMakeFiles/odas.dir/src/signal/point.o
[46/169] Building C object odas/CMakeFiles/odas.dir/src/signal/postprob.o
[47/169] Building C object odas/CMakeFiles/odas.dir/src/signal/scan.o
[48/169] Building C object odas/CMakeFiles/odas.dir/src/signal/spatialindex.o
[49/169] Building C object odas/CMakeFiles/odas.dir/src/signal/spatialgain.o
[50/169] Building C object odas/CMakeFiles/odas.dir/src/signal/spatialmask.o
[51/169] Building C object odas/CMakeFiles/odas.dir/src/signal/tau.o
[52/169] Building C object odas/CMakeFiles/odas.dir/src/signal/steer.o
[53/169] Building C object odas/CMakeFiles/odas.dir/src/signal/target.o
[54/169] Building C object odas/CMakeFiles/odas.dir/src/signal/tdoa.o
[55/169] Building C object odas/CMakeFiles/odas.dir/src/signal/triangle.o
[56/169] Building C object odas/CMakeFiles/odas.dir/src/signal/track.o
[57/169] Building C object odas/CMakeFiles/odas.dir/src/signal/window.o
[58/169] Building C object odas/CMakeFiles/odas.dir/src/asink/asnk_pots.o
[59/169] Building C object odas/CMakeFiles/odas.dir/src/system/acorr2pitch.o
[60/169] Building C object odas/CMakeFiles/odas.dir/src/signal/xcorr.o
[61/169] Building C object odas/CMakeFiles/odas.dir/src/system/demixing2env.o
[62/169] Building C object odas/CMakeFiles/odas.dir/src/system/hop2frame.o
[63/169] Building C object odas/CMakeFiles/odas.dir/src/system/hop2hop.o
[64/169] Building C object odas/CMakeFiles/odas.dir/src/system/freq2acorr.o
[65/169] Building C object odas/CMakeFiles/odas.dir/src/system/demixing2freq.o
[66/169] Building CXX object CMakeFiles/RPI_X_Compile.dir/src/main.cpp.o
[67/169] Building C object odas/CMakeFiles/odas.dir/src/system/freq2env.o
[68/169] Building C object odas/CMakeFiles/odas.dir/src/system/frame2freq.o
[69/169] Building C object odas/CMakeFiles/odas.dir/src/system/freq2freq.o
[70/169] Building C object odas/CMakeFiles/odas.dir/src/system/gain2mask.o
[71/169] Building C object odas/CMakeFiles/odas.dir/src/system/freq2frame.o
[72/169] Building C object odas/CMakeFiles/odas.dir/src/system/freq2xcorr.o
[73/169] Building C object odas/CMakeFiles/odas.dir/src/system/frame2hop.o
[74/169] Building C object odas/CMakeFiles/odas.dir/src/system/kalman2coherence.o
[75/169] Building C object odas/CMakeFiles/odas.dir/src/system/particle2coherence.o
[76/169] Building C object odas/CMakeFiles/odas.dir/src/system/mixture2mixture.o
[77/169] Building C object odas/CMakeFiles/odas.dir/src/system/env2env.o
[78/169] Building C object odas/CMakeFiles/odas.dir/src/system/kalman2kalman.o
[79/169] Building C object odas/CMakeFiles/odas.dir/src/system/pitch2category.o
[80/169] Building C object odas/CMakeFiles/odas.dir/src/system/track2gain.o
[81/169] Building C object odas/CMakeFiles/odas.dir/src/system/particle2particle.o
[82/169] Building C object odas/CMakeFiles/odas.dir/src/system/track2steer.o
[83/169] Building C object odas/CMakeFiles/odas.dir/src/system/xcorr2aimg.o
[84/169] Building C object odas/CMakeFiles/odas.dir/src/system/steer2demixing.o
[85/169] Building C object odas/CMakeFiles/odas.dir/src/system/xcorr2xcorr.o
[86/169] Building C object odas/CMakeFiles/odas.dir/src/utils/fft.o
[87/169] Building C object odas/CMakeFiles/odas.dir/src/utils/cmatrix.o
[88/169] Building C object odas/CMakeFiles/odas.dir/src/utils/fifo.o
[89/169] Building C object odas/CMakeFiles/odas.dir/src/utils/inverse.o
[90/169] Building C object odas/CMakeFiles/odas.dir/src/utils/gaussjordan.o
[91/169] Building C object odas/CMakeFiles/odas.dir/src/utils/pcm.o
[92/169] Building C object odas/CMakeFiles/odas.dir/src/utils/gaussian.o
[93/169] Building C object odas/CMakeFiles/odas.dir/src/utils/random.o
[94/169] Building C object odas/CMakeFiles/odas.dir/src/aconnector/acon_categories.o
[95/169] Building C object odas/CMakeFiles/odas.dir/src/utils/transcendental.o
[96/169] Building C object odas/CMakeFiles/odas.dir/src/utils/matrix.o
[97/169] Building C object odas/CMakeFiles/odas.dir/src/aconnector/acon_hops.o
[98/169] Building C object odas/CMakeFiles/odas.dir/src/aconnector/acon_pots.o
[99/169] Building C object odas/CMakeFiles/odas.dir/src/aconnector/acon_spectra.o
[100/169] Building C object odas/CMakeFiles/odas.dir/src/aconnector/acon_powers.o
[101/169] Building C object odas/CMakeFiles/odas.dir/src/aconnector/acon_targets.o
[102/169] Building C object odas/CMakeFiles/odas.dir/src/aconnector/acon_tracks.o
[103/169] Building C object odas/CMakeFiles/odas.dir/src/amessage/amsg_categories.o
[104/169] Building C object odas/CMakeFiles/odas.dir/src/ainjector/ainj_targets.o
[105/169] Building C object odas/CMakeFiles/odas.dir/src/amessage/amsg_hops.o
[106/169] Building C object odas/CMakeFiles/odas.dir/src/amessage/amsg_pots.o
[107/169] Building C object odas/CMakeFiles/odas.dir/src/amessage/amsg_spectra.o
[108/169] Building C object odas/CMakeFiles/odas.dir/src/amessage/amsg_powers.o
[109/169] Building C object odas/CMakeFiles/odas.dir/src/amessage/amsg_targets.o
[110/169] Building C object odas/CMakeFiles/odas.dir/src/amessage/amsg_tracks.o
[111/169] Building C object odas/CMakeFiles/odas.dir/src/amodule/amod_classify.o
[112/169] Building C object odas/CMakeFiles/odas.dir/src/amodule/amod_mapping.o
[113/169] Building C object odas/CMakeFiles/odas.dir/src/amodule/amod_istft.o
[114/169] Building C object odas/CMakeFiles/odas.dir/src/amodule/amod_noise.o
[115/169] Building C object odas/CMakeFiles/odas.dir/src/amodule/amod_ssl.o
[116/169] Building C object odas/CMakeFiles/odas.dir/src/amodule/amod_resample.o
[117/169] Building C object odas/CMakeFiles/odas.dir/src/amodule/amod_sss.o
[118/169] Building C object odas/CMakeFiles/odas.dir/src/amodule/amod_volume.o
[119/169] Building C object odas/CMakeFiles/odas.dir/src/amodule/amod_sst.o
[120/169] Building C object odas/CMakeFiles/odas.dir/src/asink/asnk_categories.o
[121/169] Building C object odas/CMakeFiles/odas.dir/src/amodule/amod_stft.o
[122/169] Building C object odas/CMakeFiles/odas.dir/src/asink/asnk_powers.o
[123/169] Building C object odas/CMakeFiles/odas.dir/src/connector/con_categories.o
[124/169] Building C object odas/CMakeFiles/odas.dir/src/asink/asnk_spectra.o
[125/169] Building C object odas/CMakeFiles/odas.dir/src/asink/asnk_tracks.o
[126/169] Building C object odas/CMakeFiles/odas.dir/src/connector/con_hops.o
[127/169] Building C object odas/CMakeFiles/odas.dir/src/asink/asnk_hops.o
[128/169] Building C object odas/CMakeFiles/odas.dir/src/connector/con_pots.o
[129/169] Building C object odas/CMakeFiles/odas.dir/src/connector/con_powers.o
[130/169] Building C object odas/CMakeFiles/odas.dir/src/asource/asrc_hops.o
[131/169] Building C object odas/CMakeFiles/odas.dir/src/connector/con_spectra.o
[132/169] Building C object odas/CMakeFiles/odas.dir/src/connector/con_targets.o
[133/169] Building C object odas/CMakeFiles/odas.dir/src/connector/con_tracks.o
[134/169] Building C object odas/CMakeFiles/odas.dir/src/injector/inj_targets.o
[135/169] Building C object odas/CMakeFiles/odas.dir/src/message/msg_hops.o
[136/169] Building C object odas/CMakeFiles/odas.dir/src/message/msg_categories.o
[137/169] Building C object odas/CMakeFiles/odas.dir/src/message/msg_pots.o
[138/169] Building C object odas/CMakeFiles/odas.dir/src/message/msg_targets.o
[139/169] Building C object odas/CMakeFiles/odas.dir/src/message/msg_powers.o
[140/169] Building C object odas/CMakeFiles/odas.dir/src/message/msg_spectra.o
[141/169] Building C object odas/CMakeFiles/odas.dir/src/message/msg_tracks.o
[142/169] Building C object odas/CMakeFiles/odas.dir/src/module/mod_classify.o
[143/169] Building C object odas/CMakeFiles/odas.dir/src/module/mod_istft.o
[144/169] Building C object odas/CMakeFiles/odas.dir/src/module/mod_noise.o
[145/169] Building C object odas/CMakeFiles/odas.dir/src/module/mod_mapping.o
[146/169] Building C object odas/CMakeFiles/odas.dir/src/module/mod_ssl.o
[147/169] Building C object odas/CMakeFiles/odas.dir/src/module/mod_resample.o
[148/169] Building C object odas/CMakeFiles/odas.dir/src/module/mod_stft.o
[149/169] Building C object odas/CMakeFiles/odas.dir/src/module/mod_volume.o
[150/169] Building C object odas/CMakeFiles/odas.dir/src/sink/snk_powers.o
[151/169] Building C object odas/CMakeFiles/odas.dir/src/sink/snk_hops.o
[152/169] Building C object odas/CMakeFiles/odas.dir/src/module/mod_sss.o
[153/169] Building C object odas/CMakeFiles/odas.dir/src/sink/snk_categories.o
[154/169] Building C object odas/CMakeFiles/odas.dir/src/module/mod_sst.o
[155/169] Building C object odas/CMakeFiles/odas.dir/src/sink/snk_spectra.o
[156/169] Building C object odas/CMakeFiles/odas.dir/src/sink/snk_pots.o
[157/169] Building C object odas/CMakeFiles/odas.dir/src/sink/snk_tracks.o
[158/169] Building C object odas/CMakeFiles/odaslive.dir/demo/odaslive/profiler.o
[159/169] Building C object odas/CMakeFiles/odasserver.dir/demo/odasserver/main.o
[160/169] Building C object odas/CMakeFiles/odas.dir/src/source/src_hops.o
[161/169] Building C object odas/CMakeFiles/odaslive.dir/demo/odaslive/main.o
[162/169] Building C object odas/CMakeFiles/odaslive.dir/demo/odaslive/configs.o
[163/169] Building C object odas/CMakeFiles/odaslive.dir/demo/odaslive/objects.o
[164/169] Building C object odas/CMakeFiles/odaslive.dir/demo/odaslive/threads.o
[165/169] Building C object odas/CMakeFiles/odaslive.dir/demo/odaslive/parameters.o
[166/169] Linking C shared library odas/lib/libodas.so
[167/169] Linking C executable odas/bin/odasserver
[168/169] Linking C executable odas/bin/odaslive
[169/169] Linking CXX executable RPI_X_Compile

00:31:38 Build Finished. 0 errors, 0 warnings. (took 7s.136ms)
```
###### Set up the Debug Configuration
Now we will try to debug it on target.

In Eclipse, select **Run -> Debug Configurations**.

Under C/C++ Remote Application, press **New Launch Configuration** and observe some settings auto-populated for your project. <br>

Other settings need to be manually populated. <br>

[Raspberry PI Image Installation]: https://github.com/vslobody/Ubuntu-to-Raspberry-PI-Cross-Compile-and-Debug-using-Eclipse-/blob/main/src/common/images/RaspberryPIImage.png

[Raspberry Pi Configuration]: https://github.com/vslobody/Ubuntu-to-Raspberry-PI-Cross-Compile-and-Debug-using-Eclipse-/blob/main/src/common/images/remmina_Raspberry_192.168.1.205_20230214-082127.png

[Select a Wizard]: https://github.com/vslobody/Ubuntu-to-Raspberry-PI-Cross-Compile-and-Debug-using-Eclipse-/blob/main/src/common/images/SelectAWizard.png

[Add new CMake Cache Entry]: https://github.com/vslobody/Ubuntu-to-Raspberry-PI-Cross-Compile-and-Debug-using-Eclipse-/blob/main/src/common/images/AddNewCMakeCacheEntry_1.png

[Properties for RPI_X_Compile]: https://github.com/vslobody/Ubuntu-to-Raspberry-PI-Cross-Compile-and-Debug-using-Eclipse-/blob/main/src/common/images/AddNewCMakeCacheEntry_2.png

[Connection]: https://github.com/vslobody/Ubuntu-to-Raspberry-PI-Cross-Compile-and-Debug-using-Eclipse-/blob/main/src/common/images/Connection.png

[Connection Linux]: https://github.com/vslobody/Ubuntu-to-Raspberry-PI-Cross-Compile-and-Debug-using-Eclipse-/blob/main/src/common/images/CoonectionLinux.png

[Connection Name]: https://github.com/vslobody/Ubuntu-to-Raspberry-PI-Cross-Compile-and-Debug-using-Eclipse-/blob/main/src/common/images/ConnectionName.png

[New Connection]: https://github.com/vslobody/Ubuntu-to-Raspberry-PI-Cross-Compile-and-Debug-using-Eclipse-/blob/main/src/common/images/NewConnection.png

[New Connection 2]: https://github.com/vslobody/Ubuntu-to-Raspberry-PI-Cross-Compile-and-Debug-using-Eclipse-/blob/main/src/common/images/NewConnection2.png

[New Connection 3]: https://github.com/vslobody/Ubuntu-to-Raspberry-PI-Cross-Compile-and-Debug-using-Eclipse-/blob/main/src/common/images/NewConnection3.png

[Properties for RPI]: https://github.com/vslobody/Ubuntu-to-Raspberry-PI-Cross-Compile-and-Debug-using-Eclipse-/blob/main/src/common/images/Properties%20for%20RPI.png

[C++ Project]: https://github.com/vslobody/Ubuntu-to-Raspberry-PI-Cross-Compile-and-Debug-using-Eclipse-/blob/main/src/common/images/CPlusPlusProject.png
