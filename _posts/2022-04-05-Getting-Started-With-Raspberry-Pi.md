## Getting Started with a Raspberry Pi
In my last post, I reviewed several of the reasons why a Raspberry Pi computer may be appealing.  Once you purchase your Raspberry Pi, you need to do some basic steps to set it up before it can be used.  There are  additional recommended steps in order to improve security of the device and your network.  This article is not intended to be a comprehensive set-up guide, but rather a discussion of some of the key steps to get things going.

## Basic Set Up 
The Raspberry Pi board itself does not have any drive space to store files or the Operating System.  Many users will typically utilize a microSD card to act as the device's hard drive; there is a slot for the card on the bottom side of the Pi.  To begin, you need to make a decision on how you want to use your Pi and what Linux distribution you would like to run.

### NOOBS - The Easiest Way
Many of the retailers selling Raspberry Pi will also offer a microSD card preloaded with NOOBS (new out of box software) for purchase.  This is the best choice for someone who wants the quickest and easiest way to get started.  All you have to do is connect your power supply and desired peripherals, insert the NOOBS card into the SD slot, and power up your Pi.  Upon initial bootup, you will be prompted to select the OS you would like to install.  On the plus side, no network access or imaging software is necessary to get started.  However, I found that the version of Raspberry Pi OS installed with NOOBS included a bunch of software that I do not envision using.

### Installing the OS Yourself
One of the benefits of a Raspberry Pi is how simple it is to switch the OS.  All you need to do is load the desired version of Linux on a microSD card and insert it into the Pi.  The official [Raspberry Pi Imager](https://www.raspberrypi.com/software/) can be downloaded for any of the major OSs, and this tool can be used to flash several versions of Raspberry Pi OS (and perhaps a few other popular distros) to your microSD.  If you have a Raspberry Pi that's operational, it may have the Imager program already installed on it.  I have personally used a Pi to set up a couple other Pis.

### Are You Going Headless? Install Lite
Raspberry Pi can be used in headless mode, which means it is interacted with remotely via the command line instead of with a keyboard, mouse, and monitor like a conventional desktop.  If you are opting to go this way with your device, Raspberry Pi OS Lite may be the best bet for you.  This version of the OS takes up less space than those with a GUI interface, and it excludes GUI-related software/services to run more efficiently.
>Note:  If you are going headless, you will need to enable SSH (and wireless, if you are not using an Ethernet connection) in order to be able to connect to your Pi from another device.

## Securing Your Pi
The following steps are not required in order to begin using your Raspberry Pi.  However, they are steps that are highly recommended in order to enhance your the security of your device and network.  The folks at ITProTV have a handy YouTube video to walk you through [securing a Raspberry Pi on your network](https://www.youtube.com/watch?v=ukHcTCdOKrc).  
> A quick note about permissions:  some commands can only be executed by the Root account, AKA the "Super User."  Many Linux distros allow you to execute commands using ```sudo```, or Super User Do, to run the command with Root access without having to log in as the Root user.

### Change the Default Password
The default username for the device is "pi" and the default password is "raspberry" - the password should be changed as soon as possible.  There are a few different ways to do so.  
* You can open up a Terminal window and execute the following command:  type ```passwd``` followed by "Enter" and follow the prompts to enter and confirm your new password.
* Alternatively, you can open up a menu of the device's configuration settings by executing ```sudo raspi-config``` from the Terminal and selecting the option to change the password.  There are several other configuration options you may wish to explore while you are there.

### Create a New User Admin Account
Since the "pi" username is so widely known and will likely be used by hackers attempting to exploit your device, it is a good idea to create a new user account.  In the Terminal, run the following command ```sudo adduser new_username``` and insert your desired new username in place of new_username.  As part of the account creation, you will be prompted to enter a password for the new user.  There will also be prompts to enter optional demographic information about the user, should you opt to do so.  

It will also be helpful to give the new user account administrative privileges, and the ability to execute those as a "Super User."  To do so, run the following commands:  ```sudo gpasswd -a new_username adm``` and ```sudo gpasswd -a new_username sudo``` - as before, substitute your username in place of new_username.

Once these steps are done, you should test the new account to ensure it was set up correctly.  The easiest way to do that is to open a new Terminal window and use SSH to sign into the Pi with your new account.  You should also try running a command as a "Super User" to confirm you can do so.  If everything works as expected, you can close the initial Terminal window for the "pi" user.

### Disable the Pi Account
After verifying that your new user account is configured as an Administrator, you should consider how to handle the "pi" account.  You could leave it "as is" and accept the possible vulnerabilities this allows, you could delete the account, or you could disable it.  It may be best to go with the last option, as there could be some software applications you may encounter that will look for a "pi" account in order to run.  Running ```sudo passwd -l pi``` will lock the "pi" account so that no one can log in as this user.

### System Update
There are a couple important commands to know in order to ensure the software packages on your Raspberry Pi are up-to-date with the current version.  These use APT (Advanced Package Tool), which allows the OS to install, update, remove, and manage the software packages on the device.  There should be a file named _sources.list_ within the /etc/apt/ directory that stores the locations of the repositories for each of the installed software packages.
* __Update__ will resynchronize the package index files from their sources, using the information from the _sources.list_ file to fetch their location(s).  You can think of this as an inventory of the installed software packages to determine what needs to be upgraded.  Update should always be run before attempting to upgrade.
* __Upgrade__ will download and install new versions of the available packages using the respositories contained in the _sources.list_ file.  Upgrade cannot install new package versions that it does not know about, and for that reason it is important to always run an "update" before and "upgrade."
* A quick way to perform these tasks is to run ```sudo apt update && sudo apt upgrade -y``` to combine these commands.  Using the "-y" flag will answer any prompts with "yes" to give the system permission to make the necessary changes.
* Be sure to regularly keep your Raspberry Pi updated and upgraded.
