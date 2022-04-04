#Intro to Raspberry Pi
In this post, I will introduce an overview of Raspberry Pi and it's standard features, outline some of the benefits of using a Pi, and provide an overview of how I am utlizing these devices.  For more information on Raspberry Pi, feel free to check out the official website of the [Raspberry Pi Foundation](https://www.raspberrypi.org/).

##What is a Raspberry Pi Exactly?
Quite simply, a Raspberry Pi is a single-board computer that fits inside the palm of your hand.  There have been several generations, with the most recent of which being the Raspberry Pi 4 Model B.  This model comes with the following hardware features:
* USB-C power supply
* Micro HDMI port x2 for 4K display output
* USB 2 ports x2
* USB 3 ports x2
* Gigabit ethernet
* Built-in wireless and Bluetooth connectivity
* Choice of 1 GB, 2 GB, 4 GB, or 8 GB RAM
* Micro SD card slot
Raspberry Pis can be connected to a monitor(s), keyboard, and mouse and used in the same manner as any other desktop computer.  Alternatively, a Pi can be used in a "headless" configuration, where you connect to the device remotely and interact with it via the command line. In either case, you need to hook up your desired peripherals and connect to a power supply and the internet and you are off and running. Well, mostly - you need to intall the operating system (OS). 

Speaking of the OS, Raspberry Pi runs on the Linux platform.  The official supported OS is called Raspbian, which can be used with a GUI (graphical user interface) that should be intuitive to use for anyone who has been exposed to a machine running Windows or MacOS.  Raspbian can also be installed as a Lite version if you plan to use your device headlessly.  The OS is most typically installed on a micro SD card, which also acts as the device's hard drive.  

##What is the Point of a Raspberry Pi?
Raspberry Pis offer myriad benefits that could attract users to the platform.  Some of these include:
* __Affordability__.  The 1 GB Raspberry Pi 4 Model B starts at $35.  Prices can easily climb over $100 for kits that include cases, cooling devices (fan and/or heat sinks), SD cards, cables, SD card readers/writers, keyboards and mice, etc.  You may or may not need to buy a kit, depending on what items you already possess and how you hope to use your Pi.
* __Small Footprint__.  The Raspberry Pi itself is approximately the size of a credit card.  If you want to protect it with a case, that size grows a bit.  However, we are still talking about dimensions that are likely less than 5" x 3 " x 2" in size.  The small footprint also applies to energy consumption, which will be far less than other computers.
* __Ability to Customize__.  Would you like to run a different Linux distrubution?  No problem - flash the disro of choice on a micro SD, insert it into your Raspberry Pi, and go for it.  Remember those cases I referenced?  They come in a variety of designs, materials, and colors to suit your taste.
* __Diversity of Uses__.  The ways you can use a Raspberry Pi are too numerous to list in this article.  For reference, the top result in a quick internet search for "projects for raspberry pi" returned an article entitled _50 Cool Raspberry Pi Projects for March 2022_.  Despite its modest size and cost, a Rasbperry Pi is a powerful computer that can be employed in countless useful ways.

##How am I Using Mine?
At present, I have a stable of three Raspberry Pis running in my home.  I initially purchased a 8 GB kit to set up a technology learning lab.  The kit included a SD card loaded with NOOBS (New Out of Box Software), which is the most plug-and-play way to set up your Pi for the first time.  The primary use-case for this Pi initially focused on two main purposes:
- To gain familiarity in interacting with Linux through the command line.  This can be done without fear of doing irreparable harm to the computer; in a worst-case scenario where I manage to seriously break something, all that is required to fix things is to reflash the OS onto the SD card and start over.
* To learn Python.  NOOBS comes with a text editor and Python IDE (integrated development environment) installed by default.  However, I elected to install Visual Studio Code to use as my preferred editor.
However, I quickly found that I underestimated the performance of this little computer.  It can easily function as a "primary" personal computer for most of the average person's day-to-day computing needs.  I find myself using it as such when my wife is working from home on our Windows-based laptop.

Raspberry Pi # 2 is set up as a NAS (network-attached storage) Server.  This allows files to be saved from and accessible to all devices connected to my home network.  Stay tuned for a future post on how I set up and configured the NAS.

Finally, the third Raspberry Pi (which is actually a Pi Zero), is running Pi-hole, which is a program that acts as the DNS server for selected devices on my network.  Pi-hole offers the ability to block most pop-up advertisements to improve the network's security, privacy, and performance.  It also has several dashboard interfaces available to view information on the network traffic of connected devices.
