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
   ##### Seeed Voice Card Driver installation
   As we mentioned in the **Hardware** section, we are using Seed Voice Card, and we need to install the drivers:
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
   with some luck, the installation script will go through, andd you will see this message at the end of the script execution:
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

[Raspberry PI Image Installation]: https://github.com/vslobody/Ubuntu-to-Raspberry-PI-Cross-Compile-and-Debug-using-Eclipse-/blob/main/src/common/images/RaspberryPIImage.png

[Raspberry Pi Configuration]: https://github.com/vslobody/Ubuntu-to-Raspberry-PI-Cross-Compile-and-Debug-using-Eclipse-/blob/main/src/common/images/remmina_Raspberry_192.168.1.205_20230214-082127.png
