## Creating Network Storage at Home
Post date:  April 16, 2022

### The Case for More Storage
 One common feature of nearly every mechanical device is that they are all going to break or fail at some point.  Now what if that device was used to store important personal and/or business data or documents?  And what if that device was also the only location where those files existed?  You can see where this is going, right?  You should make sure all these important files are backed up somehow.

I was especially interested in exploring a way to backup the files on the Raspberry Pi device I use to lab my learning efforts in IT and coding.  Like many other Pi users, I was relying on the Pi's microSD card for its onboard storage.  This seemed like a risky proposition given the relative unreliability of SD cards compared to other storage media.  In this post, I will outline the process I used to build and configure 

### What is a NAS?
In essence, a Network Attached Storage (NAS) device is a file server  situated on your local network.  It provides a mechanism to back up selected files while offering the additional benefit of file sharing among connected devices.  A NAS device may contain multiple drives within it, and these can be configured to work in a couple of ways:
-  JBOD (just a bunch of disks) - the drives work independently from one another and can be used in any desired capacity to aggregate storage; JBOD disks can be of any size, or  
- RAID (Redundant Array of Independent / Inexpensive Disks) - the drives can be configured to work together to provide enhanced redundancy and/or storage efficiency; the disks will ideally be of the same size to fully capitalize on their storage capacities.

NAS devices work much like popular cloud file-sharing services such as Google Drive, Apple iCloud, and Dropbox.  However, unlike these cloud-based offerings, NAS does not require an internet connection.  Additionally, a NAS offers increased privacy for your information since your files are not being shared with a third party service.  A privately controlled device that sits within your own network should also equate to more security for your storage as well.

If you are interested in network storage of your own, there are many commercially available options available for purchase.  This is the especially the advisable way to go if you are managing data storage on an enterprise level.  For my much smaller needs, I decided to build my own NAS using a Raspberry Pi to manage the RAID file storage.  

### Building My Raspberry Pi NAS
I referenced an article that appeared on [The MagPi Magazine](https://magpi.raspberrypi.com/articles/build-a-raspberry-pi-nas) for instructions on how to build and configure NAS and RAID on a Raspberry Pi.

**Step 1:  Gather the Materials**
- Raspberry Pi, with Raspberry Pi OS Lite installed and SSH enabled (See my previous [post](https://drewbrinkley.github.io/2022/04/05/Getting-Started-With-Raspberry-Pi.html) for instructions on setting up a Raspberry Pi)
- Power cable for the Raspberry Pi
- Ethernet cable 
- 1 TB SSD drive x 2 
- SATA to USB 3.0 Cable x 2 (I used a model that provided power to the connected SSD to be sure the drive's power needs did not exceed what the Pi was able to deliver via USB)

**Step 2:  Assemble the Hardware**
The hardware installation portion is very straightforward.  
- Use an Ethernet cable to physically connect your Raspberry Pi and router.  
- Connect the SATA cables to the SSDs and plug the other ends of the cables into the Raspberry Pi's USB 3.0 ports.
- Connect the power cables to the SATA adapters and Raspberry Pi and plug those into a nearby outlet.

**Step 3:  SSH into the Pi**
I am running the my Rasbperry Pi NAS in headless mode, so making any configuration changes to the device requires making a SSH connection from another device.  Once connected, I used the Terminal to make all of the necessary configuration changes as outlined below.

**Step 4:  Add the Drives**
I executed the command ```lsblk``` to see a list of all the block storage devices connected the Raspberry Pi.  (_Note:  You may need to wait a few moments after plugging the drives in to allow the Pi time to "see" those devices._)  The command will return a tree-like list of these devices.

One of the outputs you should see is 'mmcblk0' - this is likely going to be the microSD card that is running your Pi's OS.  A storage drive connected to the Pi should appear in the format 'sda' (Storage Device A), with any additional devices continuing  alphabetically (e.g. 'sdb' and 'sdc').  You should be able to differentiate your SD card from your SSD / hard drives based on their associated sizes.  In my case, my two SSD drives were labled as 'sba' and 'sdb.'

**Step 5:  Format the Drives**
Now that I identified the drive(s) associated with my NAS, I needed to format them so the Raspberry Pi knows how to store the data.  I executed the command ```sudo fdisk /dev/sda``` to format the first of my SSDs.  You should expect to see several prompts as _fdisk_ works:
- Enter 'n' to select a new partition
- If there is an error message indicating the existence of a partition already, delete it with 'd'
- Enter 'p' to indicate this is the primary partition for the drive
- Accept subsequent default values by pressing 'ENTER'
- Once complete, you should get a message indicating a new partition was created
- Enter 'w' to write the changes to the disk, and then _fdisk_ will exit.
Perform these same steps on every other drive to be added to the NAS, replacing 'sda' in the command with the relevant drive's name.

**Step 6:  Build the RAID Array**
A RAID array can be configured is several different ways, depending on the preferences for speed, efficiency, and redundancy.  I elected to go with a RAID-1 setup, which is also referred to as mirroring.  RAID-1 requires at least two drives, and it writes the same data to all drives in the array.  This maximizes the data's redundancy - if one drive fails, the full data exists on the other drive(s) in the RAID array.  All you need to do is replace the faulty drive and the system will rebuild itself without any data loss.  

To begin, I ran the command ```sudo apt install mdadm``` to install _mdadm_, a Linux package for administering RAID.  After the installation completed, I executed ```sudo mdadm --create --verbose /dev/md0 --level=mirror --raid-devices=2 /dev/sda1 /dev/sdb1``` to create the RAID-1 array from my two SSDs.

**Step 7:  Mount the Drive**
After the previous step, the Raspberry Pi should see the newly-created RAID array as a single device.  This had to be formatted, mounted, and configured for use as a virtual drive.  I executed the following commands:
- Create a parent directory for the RAID-1 array in the /mnt directory (the location intended for storage devices):   ```sudo mkdir -p /mnt/raid1```
- Build the file system and its path:  ```sudo mkfs.ext4 /dev/md0```
- Mount the drive, combining the file system and directory just created:  ```sudo mount /dev/md0 /mnt/raid1/```
- Confirm that your RAID-1 is functional ('lost+found' should be listed in the /mnt/raid1 directory):  ```ls -l /mnt/raid1/```
- Open the Nano text editor to amend the 'fstab' file, which is the Linux system's filesystem table:  ```sudo nano /etc/fstab```
- Tell the system to ignore writing instances when a file was accessed (but not modified) by adding this line:  ```/dev/md0 /mnt/raid1/ ext4 defaults,noatime 0 1``` (quit and save afterwards by using CTRL-X, then Y).
- Configure the RAID to automatically start up every time the Pi boots:  ```sudo mdadm --detail --scan | sudo tee -a /etc/mdadm/mdadm.conf```
- Reboot the Raspberry Pi.

**Step 8:  Install and Configure Samba**
Since I want my NAS to have file-sharing capacity for multiple devices (which use different operating systems) on my network, I needed to install and set up Samba to facilitate file sharing services.  Samba utilizes the file-sharing protocol SMB to allow this network file sharing.
* Install Samba:  ```sudo apt install samba samba-common-bin```
* Create a directory for file sharing:  ```sudo mkdir /mnt/raid1/shared```
* Allow all users with access to the shared directory to have reading and writing privileges:  ```sudo chmod -R 777 /mnt/raid1/shared```
* Open Nano to edit the Samba configurations:  ```sudo nano /etc/samba/smb.conf```
* Allow Samba to share the new directory on the local network by adding the following text to the bottom of the config file:  
	*```[shared]
	path=/mnt/raid1/shared
	writeable=Yes
	create mask=0777
	directory mask=0777
	public=no```
	Quit and save afterwards by using CTRL-X, then Y.
* Restart Samba:  ```sudo systemctl restart smbd```

**Provide Access to the Shared Directory**
The final step in this process is to give user access to the shared NAS directory.  I executed the following command to grant access and set a Samba password:  ```sudo smbpasswd -a user_name``` (you can replace user_name with whatever is appropriate for your situation).  Once done, this user can access the files shared by Samba on any Windows, Mac, Linux, or Chrome device on your network.  The files can be read or written to from any of these devices, as configured above.  

There are also steps that can be taken to add additional users, who can each have their own private directories on the NAS.  This was not necessary for my purposes, so I will not describe that process here.

### Next Steps
At this point, I had my own local file server on my home network.  I could connect to it from any of my devices, and access / modify any of the files as needed.  I also had a RAID-1 configured to provide a layer of redundancy in case one of my NAS drives failed.  

My next goal was to explore how to back up files to the NAS on a regular basis.  I will be making a future post to outline the use of rsync to synchronize important files on my primary Raspberry Pi to the NAS through the use of a script to automate this process.