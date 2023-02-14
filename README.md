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
 
[Raspberry PI Image Installation]: https://github.com/vslobody/Ubuntu-to-Raspberry-PI-Cross-Compile-and-Debug-using-Eclipse-/blob/main/src/common/images/RaspberryPIImage.png
