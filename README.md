# hackintosh-full-installer
Repo for a script that can complete part of the task of creating a full macOS Catalina Hackintosh installer entirely from Linux.

This script is based on instructions in [this YouTube video](https://www.youtube.com/watch?v=fqK-dUdChwE) which is Part 1 of a two part series explaining how to create a full macOS Catalina installer created using tools in Windows.

However the **create-installer** script in this repo contains all of the commands needed to partition, format and add the macOS installation files entirely from Linux so none of these programs are needed: Boot Disk Utility or Paragon Partition Manager or Paragon HFS+ for Windows.

### Required files
Create a folder in your linux distros home folder called **install**

This screenshot shows the required contents of that folder:

![image](https://user-images.githubusercontent.com/32464325/141846600-85a2d07c-cbe8-4701-8ae6-484c9bd5413f.png)

The Shared Support folder comes from downloading the macOS Catalina installation files using the [gibMacOS tool](https://github.com/corpnewt/gibMacOS). Click on the Green 'Code' button and download the .zip file to your home folder. Then right click on it and extract it to somewhere in your home folder.
```
# cd /home/flex/Documents/hackintosh/gibMacOS-master
# python3 gibMacOS.command
```
This gives a menu of macOS versions. Choose **macOS Catalina 10.15.7 (19H15)**
When everything has downloaded copy the required files into your **..\install\SharedSupport** folder
Rename **InstallESDDmg.pkg** to **InstallESD.dmg**

In **InstallInfo.plist** change this...

```
<key>Payload Image Info</key>
  <dict>
  <key>URL</key>
  <string>InstallESDDmg.pkg</string>
  <key>chunklistURL</key>
  <string>InstallESDDmg.chunklist</string>
  <key>chunklistid</key>
  <string>com.apple.chunklist.InstallESDDmg</string>
  <key>id</key>
  <string>com.apple.pkg.InstallESDDmg</string>
  <key>sha1</key>
  <string></string>
  <key>version</key>
  <string>10.15.7.0.0.1604077749</string>
  </dict>
```
To this...
```
<key>Payload Image Info</key>
  <dict>
  <key>URL</key>
  <string>InstallESD.dmg</string>
  <key>id</key>
  <string>com.apple.dmg.InstallESD</string>
  <key>sha1</key>
  <string></string>
  <key>version</key>
  <string>10.15.7.0.0.1604077749</string>
  </dict>
```
### Running the create-installer script
From a terminal change directory to the "install" directory which contains the create-installer script and you run it like this:
```
sudo script -c ./create-installer /home/flex/macOS/install/create-installer-output.txt
```
The output will appear in the Terminal and also get logged to a file.

### Notes
The script has only been tested on Manjaro and Ubuntu mate.

If running the script on an Arch distro ensure the **unMountIfMounted()** method is like this:
```
unMountIfMounted(){
    echo "Unmounting partitions on "$dev_block"..."
    ls ${dev_block}* | xargs -n1 umount -l
}
```

Else if running the script on an Ubuntu type distro ensure the **unMountIfMounted()** method is like this:

```
unMountIfMounted(){
    echo "Unmounting partitions on "$dev_block"..."
    ls ${dev_block}?* | xargs -n1 umount -l
}
```

The created macOS Catalina installer will not install macOS Catalina if the installer was created in Manjaro and you are using the **OpenHfsPlus.efi** driver in Opencore.
The **HfsPlus.efi** driver (from Apple) *is* able to run the macOS installer if created in Manjaro or Ubuntu mate.

The **macos-EFI.img** file contains a backup of the EFI partition from a working Catalina USB Installer made to install and run macOS Catalina on my laptop. This .img file was made using a dd command like this:
```
sudo dd if=/dev/sdb1 of=/home/flex/macos/install/macos-EFI.img bs=64k iflag=fullblock status=progress
```
Unless you happen to have a backup of your Opencore EFI folder stored in this way then you should comment out the call to the **copyEFIFolder()** method. Instead you simply create a folder called **EFI** in the root of the fat32 partition on the macOS installer and add your Opencore files/folders in to that.


#### Acknowledgments
**cvad** the developer of [Boot Disk Utility](http://cvad-mac.narod.ru/index/bootdiskutility_exe/0-5) provided valuable help with commands to write the Apple Disk Image to a USB drive. The partition structure created by this script is also based on that of **Boot Disk Utility**

**Igor Pavlov** the developer of [7-Zip](https://sourceforge.net/projects/sevenzip/) provided valuable help with commands to extract the Apple Disk Image (4.hfs from BaseSystem.dmg)
