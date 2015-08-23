&lt;wiki:gadget url="http://drobocapsule.googlecode.com/svn/wiki/adsense2.xml" border="0" width="468" height="60" /&gt;



# `Introduction` #

Apple makes it very difficult to use TimeMachine to backup to anything other than the expensive proprietary  TimeCapsule. This project enables one to use TimeMachine to backup to a Drobo/DroboShare over AFP via netatalk. This project is fully tested and stable. Multiple machines can be backed up to the same Drobo using different sparsebundles for each machine.

Note: I take no responsibility if this software destroys your applications or you lose all of your possessions.

# `Installation` #

## `Drive Formatting Considerations` ##
For the best results, you should be using ext3 as your filesystem on your Drobo. There is a caveat in that you won't be able to run Disk Utility on your Drobo unless you purchase a [$40 piece of software](http://www.paragon-software.com/home/extfs-mac/).
I also haven't tested this.

NTFS and FAT32 do not support symbolic links properly, so they won't work. The Linux implementation of HFS+ (what is running on the DroboShare) has a bug with deleting files as well as with resource forks. So, the only way to free up deleted space on your Drobo for files that have been deleted with BackMyFruitUp, is to run Disk Utility.

If you are **only** using your Drobo for storing TimeMachine backups, you are pretty safe with HFS+. I use this solution because I don't delete files on my Drobo and I only use it for TM.

## `Install DroboAdmin` ##
Install the [DroboApps Admin](http://drobo.com/droboapps/downloads/index.php?id=19) package using the standard [DroboApps installation instructions](http://drobo.com/droboapps/faq/index.html). Once you have the admin package installed, make sure to also install [Dropbear SSH](http://www.drobo.com/droboapps/downloads/index.php?id=18) and [Add Swap Memory](http://www.drobo.com/droboapps/downloads/index.php?id=9). Both of these are easily installed from the Admin package.

## `Confirm the DroboApps Folder Path` ##
Open Terminal.app and log into your DroboShare via SSH. ssh root@[DROBO IP HERE](YOUR.md). ex: ssh root@192.168.1.32

_Note: the default root password is stored in the Important Information tab of the DroboAdmin website. See ReadMe.pdf enclosed with the Drobo Admin download for more details._

![http://backmyfruitup.googlecode.com/svn/wiki/ssh-example.png](http://backmyfruitup.googlecode.com/svn/wiki/ssh-example.png)

Confirm that the path to your DroboApps folder is this: ` /mnt/DroboShares/Drobo/DroboApps `. **This is required for BackMyFruitUp to install properly.** The reason is because there is so many hard coded paths in the configuration files that it would be a real nightmare to try to make this any more dynamic than it is. If it is not this directory structure, you will need to use the latest >1.6.8 release of [Drobo Dashboard](http://drobo.com/support/updates.php) to rename your Drobo volume.

These command should also work to set the path if you don't have the above path:

```
ls -la /mnt/DroboShares
(take note of the name of the folder in this directory listing)
ln -s /mnt/DroboShares/NAME_FROM_LINE_ABOVE /mnt/DroboShares/Drobo
```

**Please note:** If you can't get the naming correct, then there isn't much I can do to help you. You can either try making a single large partition for your Drobo (which is when I see people generally having this problem) or I suggest contacting the Drobo support staff.

## `Get BackMyFruitUp` ##
If you have not already done so, download the BackMyFruitUp 3.1 package from the Downloads tab above. Do not expand the enclosed netatalk.tgz file.

## `Install BackMyFruitUp` ##
Install netatalk.tgz by coping the file into your DroboApps folder. You will need to restart the DroboShare to get things running. It is easiest to do do a restart by using the DroboAdmin webpage.

![http://backmyfruitup.googlecode.com/svn/wiki/tgz-in-folder.png](http://backmyfruitup.googlecode.com/svn/wiki/tgz-in-folder.png)

Once the DroboShare is restarted, the netatalk.tgz file will convert into a netatalk folder. This should cause 'DroboCapsule' to appear in the Finder under the SHARED heading next to the DrobShare heading. It should have an icon like an XServe server.

![http://backmyfruitup.googlecode.com/svn/wiki/properly-expanded.png](http://backmyfruitup.googlecode.com/svn/wiki/properly-expanded.png)

# `Details for TimeMachine Setup` #

## `Locate the DroboCapsule mount` ##
From a Finder window click on the DroboCapsule in the Shared list
and mount it as guest.  You should see two volumes, one titled Drobo and
another titled DroboCapsule (see image below).  These are mount points via AFP. They
differ from the mount via the Drobo Dashboard called DroboShare, which uses
SMB. Load the DroboCapsule volume from the mount point. This is where you will put the
sparsebundle that will be created in the next step.

![http://backmyfruitup.googlecode.com/svn/wiki/drobocapsule_mounts.png](http://backmyfruitup.googlecode.com/svn/wiki/drobocapsule_mounts.png)

## `Setup BackMyFruitUp` ##
This part is new in 3.1. OSX 10.6 introduced a change to the way TM backups work. Previously, it was possible to create a sparsebundle of a specific size and that would limit the size of the backup. The new change is that TM will automatically resize the sparsebundle to be the maximum size of the volume it is on. Therefore, there is a new option to netatalk which allows you to define the size of the volume. To do this, you need to edit the AppleVolumes.default configuration file by hand using your favorite text editor (TextEdit works fine). It is located in the directory tree outlined in the image below.

![http://backmyfruitup.googlecode.com/svn/wiki/applevolumes_default.png](http://backmyfruitup.googlecode.com/svn/wiki/applevolumes_default.png)

At the very bottom of the file, the important lines are:

```
/mnt/DroboShares/Drobo/DroboCapsule "DroboCapsule" options:usedots,invisibledots,tm volsizelimit:650000
/mnt/DroboShares/Drobo Drobo options:usedots,invisibledots veto:/DroboCapsule/
```

At the end, it shows the DroboCapsule volume has been limited in size to 650gigs. This is the default size that I've configured for BackMyFruitUp and you can change this to be any size assuming you have enough space on your Drobo. When choosing a size, you should consider the total amount of space for all of your backups.

Please note, I haven't tested changing the volsizelimit value once backups have been started. I don't think it is wise to make it a smaller number.

Once you change this file, you should restart your DroboShare using the DroboAdmin application in order to make the changes take effect.

## `Run Create Backup Volume` ##
Run the Create Backup Volume (CBV) application which you can download from the downloads tab above. It will ask you for the size of the volume and will create a new sparsebundle on your desktop. You should set this to a size which is about 100megs larger than the amount of space used on the drive you are backing up. This will take a few minutes to run.

After it is finished, copy the sparsebundle to the DroboCapsule share and delete it from your desktop. Note: You will need to run the CBV application on each Mac that you want to backup.

In the end, the size doesn't really matter `;-)`. As I said before, TM will resize your sparsebundle to be the maximum size of the volume.

It is very important to use my CBV application to create the sparsebundle because the default one created with TM has a band size of only 8mb (mine uses 64meg bands). This causes a whole bunch of small files to be created on the Drobo and performance will suffer greatly as a result. In fact, I wasn't even able to get a backup to succeed with the 8meg band size.

## `Setup TimeMachine` ##
Open up TimeMachine in System Preferences, Change Disk to DroboCapsule. If you don't see DroboCapsule, then click on it in the Finder to mount it as a guest user.

![http://backmyfruitup.googlecode.com/svn/wiki/tm-selection.png](http://backmyfruitup.googlecode.com/svn/wiki/tm-selection.png)

Remember, if you are on wireless, the first backup will take a long time. Sometimes it is better to just plug the machine in to a hub and back up the first time.

Now, start a backup as normal.

# `Upgrade` #

If you are upgrading from the previous version of BackMyFruitUp, all you need to do is first rename the existing netatalk folder to something else. Copy over the new netatalk.tgz and then restart your DroboShare using the DroboAdmin webpage. Please note that netatalk now runs as root by default. This will allow you to access all of your SMB files without problems. Once you have restarted and you have verified things are working, you can safely delete the older netatalk folder.

# `Uninstall` #

Same with upgrade. Rename the folder, restart. Delete the folder after a restart.

# `Full System Restore` #

A full system restore has been successfully tested using this system. Note, this will erase the machine you are restoring to and install a backed up image onto it. You can even choose at which point in time you would like to restore from. A full restore of my ~15gig laptop took about 30 minutes over a 100mbit wired ethernet connection. In order to do the restore, please follow the steps below.

#1. Boot up from your OSX 10.5 / 10.6 installation dvd.

#2. Select Terminal from the Utilities menu.

#3. Mount the Drobo drive on your machine by typing the following into the Terminal window exactly as shown while replacing IPADDRESS with the actual IP address of your DroboShare:

` mkdir /Volumes/DroboCapsule ; mount -t afp afp://IPADDRESS/DroboCapsule /Volumes/DroboCapsule `

#4. Quit Terminal and then select "Restore System From Backup..." in the Utilities menu.

#5. Your backups should show up and you can just follow the standard steps for a restore.

# `Users/groups permissions` #

This is left as an excercise for the reader. It is entirely possible to set this up, but will require reading and understanding the netatalk documentation. You will also need some knowledge of how to create and manage unix users and groups.

_Please note that it is not recommended to use this solution to do this._ The primary goal of BackMyFruitUp is only to provide an inexpensive/low power way to do TM backups and file access to a small number of users in a home/small office setting. The DroboShare is not a high performance file server at all and using a solution like this for doing that is kind of the wrong tool for the wrong job. If you want a much better, but still inexpensive tool, I'd suggest getting a Mac Mini and plugging your Drobo into that. You will also get a nice user interface for managing users and can integrate that into a central authentication store.

# `Common problems` #

  * If TimeMachine immediately turns off after selecting a volume and you see the error message below in the Console.app's logs, then open up the Keychain Access.app, search for Drobo and delete whatever key is there.

```
System Preferences[565] Time Machine: Error -25299 while storing password in keychain for username:  serverURL: 
afp://;AUTH=No%20User%20Authent@DroboCapsule._afpovertcp._tcp.local/DroboCapsule 
```

  * If TimeMachine fails to backup after a successful backup with the error (The backup volume could not be mounted.), MAKE SURE the DroboCapsule volume is not mounted. Click the little eject button next to it. It is fine to mount the Drobo volume at any time. It is also fine to mount the DroboCapsule volume AFTER TimeMachine has started. Just make sure it isn't mounted before TimeMachine starts up.

  * Sometimes netatalk can get into a funk. Especially if the DroboShare goes down during a copy. The symptom is that when trying to connect to the server the connection will fail (even though it was working fine before). If this happens, ensure nobody is connected to the server or trying to connect. Then, log into the DroboShare over ssh and `cd /mnt/DroboShares/Drobo; rm -rf .AppleDB .AppleDesktop .AppleDouble`. You may also have to remove these directories from the DroboCapsule folder as well. It seems that occasionally these directories full of files can sometimes get corrupted. Removing these folders fixes that problem and doesn't seem to affect anything else. Here is a screen shot example of deleting those folders...

![http://backmyfruitup.googlecode.com/svn/wiki/delete_apple_folders.png](http://backmyfruitup.googlecode.com/svn/wiki/delete_apple_folders.png)

  * If you are experiencing slow copies or backups, it is probably because you have antivirus software installed. Either disable it or somehow tell it to not check files during backups.

  * According to [Apple's documentation](http://developer.apple.com/mac/library/documentation/NetworkingInternetWeb/Conceptual/TimeMachineNetworkInterfaceSpecification/TimeMachineRequirements/TimeMachineRequirements.html#//apple_ref/doc/uid/TP40008951-CH100-SW1), it is important not to store other files on the DroboCapsule -> DroboCapsule share point other than the sparse images. If you need to put files on the server, put them in the DroboCapsule -> Drobo share point.

# `Getting Help` #

There is a [google group](http://groups.google.com/group/backmyfruitup) setup for discussion.

  * Please make sure that you read all of the [archives](http://groups.google.com/group/backmyfruitup) first before posting any questions.
  * If you haven't figured out your problem from the archives, then please make sure to give as much information as possible about what you have done in setup.
  * Include output of 'ps' and 'ls -la' when showing what is running on your DroboShare and paths to things. Include any relevant lines from Console.app.
  * You will most likely get a faster and more complete response if you remember to donate. I created this project in my spare time. It works fine for me and many other people. Supporting your issues takes even more time and energy, so donations keep me motivated to help you.

# `That annoying message` #

To turn off the annoying message, all you need to do is rename the MESSAGE.txt file to anything else and then reboot your DroboShare. You can also edit the file and put your own message there.

![http://backmyfruitup.googlecode.com/svn/wiki/message-no-more.png](http://backmyfruitup.googlecode.com/svn/wiki/message-no-more.png)