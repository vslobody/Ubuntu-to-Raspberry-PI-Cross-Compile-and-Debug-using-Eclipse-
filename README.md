# Ubuntu to RaspberryPI Cross Compile and Debug using Eclipse
## Introduction
Some time ago I have encountered the need to develop similarly straighforward code for Raspberyy Pi platform. After a little bit of back and forth considerations between python and C++, I have chosen to experiment with C++ option.
The next iteration of decisions was whether to do development on the Raspberry Pi platform itself, or on the Ubuntu box and then cross compile code for Raspberry PI. My choice was to do development on Ubuntu, since Ubuntu is much more powerfull machine than Raspberry Pi.
In terms of IDE, my default choice is Eclipse. Obviously, there are many great IDEs out there - Visual Studio, Clion to name a few, but I like Eclipse. Last but not the least - we will use CMake.
## Acknowledgments. 
This is derived work inspired, primarily, by [this excellent guide](https://tttapa.github.io/Pages/Raspberry-Pi/C++-Development-RPiOS/) and many other distinguished contributors. 
