Post Date:  April 20, 2022

### Background
In my last post, I discussed the steps I went through to create a Network Attached Storage (NAS) device using a Raspberry Pi.  One of the primary purposes for this endeavor was to create an effective way to back up important data from my devices.  Please check out that [post](https://drewbrinkley.github.io/2022/04/16/Setting-Up-a-Raspberry-Pi-NAS.html) if you would like information on how to set up and configure your own Raspberry Pi NAS.

It is simple enough to use the standard ways to drag-and-drop or copy-paste files from a local device's GUI (graphical user interface) into the Shared directory on the NAS.  However, I wanted to add some automation to the task of file management.  After going through the steps in this post, I have a script that will synchronize files from my Raspberry Pi lab setup to my NAS without any further action on my part.

>Whoa there!  I have now mentioned two Raspberry Pi devices in this post.  That's right - I have multiple instances of this interesting and functional little devices!  In order to simplify nomenclature in this post, I will be referring to the Pi that I use as a coding  / IT lab as RP1 and the Pi running my NAS as RP2.

### Connecting Pis with SSH
Since my NAS is running on RP2 that is in headless mode (it does not have an attached display, keyboard, or mouse), I have to establish a remote connection to the Pi from another device in order to perform any tasks or configurations.  The standard way to do this is to use SSH (secure shell).  

In order to establish the remote connection, I hop in to the Terminal of RP1 and enter ```ssh <user_name>@<IP_address>``` (you will replace <user_name> with the account you have on your remote device and <IP_address> with the IP address of the remote device on your network).  By default, I am prompted to enter the password for the account that is being used to log into RP2.

### Configuring SSH Keys to Login Without a Password
My file-syncing plan is going to require frequent remote communication between RP1 and RP2.  I do not want to worry about having to enter a password every time this SSH connection is made.  In this step, I will be configuring SSH keys to provide authentication in lieu of a password.

SSH keys consist of a pair of public/private keys.  Conceptually, the public key acts like a lock, and the private key is the key that will open that lock.  The public key can be shared with as many different hosts/servers as you want, and done without regard to who may be able to see it.  The private key is what must be kept protected, as anyone who has that will be able to gain access to the public key.

To generate the key pair, I typed ```ssh-keygen``` into the Terminal of RP1.  The key-generation process requires a location to store the pair, and it will default to ```/home/pi/.ssh/id_rsa``` unless another path is specified.  There will also be a prompt to enter a passphrase (this step can be bypassed by pressing 'ENTER').  When completed, the system will show you the path where the public key has been saved and a random image that represents the key's fingerprint.

After the keys are created, the public key must be shared with the remote host (RP2).  While still in RP1's Terminal, I typed ```ssh-copy-id <user_name>@<IP_address>``` (again, replacing <user_name> and <IP_address> with the appropriate values for RP2).  This should prompt for the user password for the remote host.  Once entered, the SSH key pair should be configured between RP1 and RP2.

### File Synchornization with Rsync
For my hands-off file management strategy to work, I employed Rsync.  Rsync is a tool that can be used to copy files in Linux.  The files can be copied locally or rsync can push/pull files between local and remote hosts.

I focused in on backing up a specific directory on RP1 that contains all of my notes from reading, learning, and writing.  I use GitHub as a repository for any coding projects of significance, and I did not feel it was necessary to maintain a full backup of RP1 since it's easy enough to restore a new version of Raspberry Pi OS if something should necessitate it.

There are a wide range of options for configuring Rsync.  I used the following in my command:
* -a :  archive mode, which syncs data recursively while also maintaining file permissions and ownership, symbolic links, and timestamps
* -v :  verbose output, to display details of the transfer
* -u :  update, which will compare the timestamps for the local and remote hosts, and only transfer files that are new or or have a more recent timestamp on the sender 
* -e :  remote shell, which allows you to specify which protocol will be used for remote transfers
 
I entered ```rsync -avue ssh /<local_path>/ <user_name>@<IP_address>:<remote_path>``` to specify that SSH will be used to connect RP1 and RP2.  If you are following this example, replace:
- <local_path> with the path to the directory on your local client you would like to back up,
- <user_name> with the account you use to connect to your remote host,
- <IP_address> with the IP address of your remote host, and
- <remote_path> with the locaion on your remote host where you would like to store your archived files.
The -v flag provided some confirmation that the transfer was completed.  However, I wanted further confirmation so I made a SSH connection into my RP2 NAS, navigated to the archive directory, and used the ```ls``` command to verify that all of my files were there.  Success!

### Automating the Synchronization
Now that I had a working Rsync command, the final piece of the puzzle was to automate the file synchornization.  To do that, I used Linux's _cron_ service to create a schedule task for executing Rsync.  The general syntax for a cron job is "* * * * * <command>" - each "* " represents the time components (minute, hour, day of month, month, and day of week) that can be used in the scheduling, and <command> can be replaced with whatever command you desire to run at the scheduled time.

To do create my job, I entered ```crontab -e``` into the Terminal on RP1.  The first time you do so, you will be prompted to select which editor you want to use to make changes to the cron table.  At the bottom of the file, I inserted ```0 2 * * * rsync -avue ssh /<local_path>/ <user_name>@<IP_address>:<remote_path>``` - this will run my Rsync job at 2:00 a.m. daily (the *s will remain as wild cards and will run every date, month, and day of the week since no specific values are entered in those positions).  

I created a new file called _Test.md_ on RP1 so I could see if things worked properly.  When I checked the archive directory in RP2 the next day, _Test.md_ had been synchronized to my NAS as expected.

### Making an Adjustment to Delete Files
As written, my Rsync command was great for transferring new and modified files from RP1 to RP2.  However, things would not be in perfect harmony if I deleted files on RP1.  I needed to make an adjustment to my script to ensure the file deletions were also performed on the NAS.  

I opened up my cron table and added the _--delete_ flag to my script:  ```0 2 * * * rsync -avue --delete ssh /<local_path>/ <user_name>@<IP_address>:<remote_path>```.  I added a new file named _Test2.md_ and deleted my original test file, _Test.md_ on RP1 so I could check the functionality the next day.
- __I broke things.__  When I checked in on the status of RP2 the next day, I discovered that things had not gone according to plan.  The original test file, _Test.md_, was still on the NAS even though I had deleted it on RP1.  Also, the new test file, _Test2.md_, had not been copied over.  
- __Confirming the job ran.__ After a couple days of optimistically checking, I was unable to observe any difference in the NAS's archive directory.  I had also made some real changes to files in RP1, and nothing was syncing with RP2.  I decided to check the log files on RP1 to see if my cron job was running.  I opened up a Terminal window oand navigated to the ```/var/log``` directory, then used ```cat syslog``` to display the system log.  I scrolled back to 2:00 a.m. and verified that my cron Rsync job initiated, but there was a message stating that it failed to execute due to "--delete:  No such file or directory."
- __Fixing the script.__  I hit the internet to research what was wrong with my script.  I realized that the _-e_ flag was expecting to see a protocol for the remote connection, and my command was telling it to use _--delete_ instead of _ssh_ now.  A quick change to the order of my syntax resolved that, and the format of the final command was  ```0 2 * * * rsync -avue ssh --delete /<local_path>/ <user_name>@<IP_address>:<remote_path>```.
When I checked the status the next day, I confirmed that all the changes (additions, modifications, and deletions) in RP1 were syncing to RP2.  

### Final Thoughts
After completing this project, I had successfully built a local file server on my network and automated the process of synchronizing files between two devices.  This project had a number of significant steps:
- Assembling hardware to build a NAS
- Formatting, partitioning, and mounting new drives
- Creating a filesystem
- Configuring a RAID array to ensure redundancy in file storage
- Setting up a file share for my local network
- Generating and deploying SSH keys to authenticate to a remote server
- Using Rsync to synchronize file storage between two devices
- Creating a script to automate the file sharing process
I also had to perform some troubleshooting when my work produced some unexpected results.  This provided a great opportunity to get some experience with some activities that have usage in networking and systems administration activities in an enterprise environment (albeit, on a different scale).  I learned a lot in this project, and I would encourage you to try it out if you have a spare Raspberry Pi and a couple of extra drives.