# Dipper-Mi8-TWRP-DUAL-BOOT
Dipper--Xiaomi Mi8 Guide to How to make a dual-boot MOD for TWRP
<hr>
# Making a DualBoot Mod for TWRP

I figure out that there's a growing interest in it(android-dual-booting) so I make a guide on it

## What you will need
* A working brain
* Magisk zip
* Some binaries (Some of these are included in twrp but varies by device so I'd just pack your own)
  * From magisk zip: busybox, magiskboot, magiskinit (rename magiskinit64 to magiskinit and use it if device is 64bit)
  * For ext4 partitions: e2fsdroid, mke2fs
  * For f2fs partitions: sload.f2fs, mkfs.f2fs
  * Sgdisk (for partitioning)
* ramdisk-recovery.cpio extracted from your device's twrp zip
* My dualboot repo works as a good starting point (feel free to fork and modify however)
  * I left some helpful comments in it to guide you on to what it does in each file
* Partition layout of your device
  * THIS IS THE TRICKY PART, MAKE SURE YOU GOT IT RIGHT OR YOU'LL END UP WITH A BRICK
  * Use sgdisk in existing twrp. For example: `sgdisk /dev/block/sda --print`
  * You will need to find the locations of boot, system, vendor, userdata, metadata (if applicable)
  * If across multiple blocks, you'll need to change this sgdisk function throughout the zip

## Overview of contents of each file
* META-INF/com/google/android:
  * updater-script: Just a placeholder file, don't change this
  * update-binary: The starting point for the zip
    * Sets up the environment (sets some variables, functions needed immediately, busybox, etc.)
      * Change target on line 13 to the path to boot block (good chance it's the same as mine)
      * Change the print stuff to whatever you want to call it/author(s) and such
      * May need to change other variables to match what's in magisk zip
    * Runs a check to ensure that zip is being flashed on a supported device. Changes you'll need to make
      * Line 65 - change to supported devices
      * Line 70 - change to function for userdata partition (if needed)
    * Checks if data is encrypted (can't reformat that, zip has workarounds build in for it)
    * Get the partiiton layout
      * As noted above, you may need to rework this
    * Checks if zip is in userdata - obviously can't repartition if it is
    * Goes through user options
    * Patches the twrp cpio for selected options
      * You may need to change this if the files in twrp are different (unlikely)
    * Adds needed binaries
    * Repartitions userdata as selected by user (all that magic is located in tools/functions.sh)
    * Installs modified twrp
    * You can remove the 2SI stuff at lines 310-313 if using magisk 21+ tools
    * Mounts partitions
      * Change to block paths (remove odm and persist if you don't have them) - lines 317-327
    * Installs dualboot stuff - after you get partitoning figured out, this will be the other sore spot
      * Sepolicy patching is tricky, you'll need to change this based on where it's located on your device
      * Note that order matters, some locations take priority over the other - odm > vendor > /system/etc/selinux > root (monolithic sepolicy)
      * The commented out stuff at lines 348-349 and 363-370 are for cil patching. I kept booting to fastboot and didn't feel like putting in the effort to figure it out when we can just force monolithic sepolicy creation instead. If you want to put the effort in to figuring it out and succeed, let me know
      * Some of these policies may need changed (you may need to add more). To debug, grab full logcats and dmesg logs and look for denials around when the datacommon service is run
      * I made magisk optional which made the sepolicy setting much more complicated. Easier thing to do is to just make magisk mandatory and just set context to magisk rather than shell
      * You may need to change /system_root to location of root on your device
      * Note that in android 11, init.rc was moved from root to /system/etc/init/hw
    * Installs magisk if selected
      * You may need to change this with each magisk update. Most magisk install stuff is taken straight from the magisk zip
    * Repacks boot img with new twrp and cleans up

* tools/functions.sh:
  * Contains all functions taken from magisk's util_functions that were modified (located at bottom of file). I did it this way so I can keep track of things easier (you can just copy/paste everything in magiskcommon from magisk zip) and because util_functions isn't called if magisk isn't being installed
  * What you may need to change:
    * chooseport: if it doens't work on your device, use the volume key method with keycheck binary. See [here](https://forum.xda-developers.com/android/software/guide-volume-key-selection-flashable-zip-t3773410) and [here](https://github.com/Zackptg5/MMT-Extended-Addons/tree/master/Volume-Key-Selector)
    * select_size: change to supported userdata sizes
    * The partitioning stuff will need changed based on your partition locations
      * This will likely be the most difficult part, take care that you get it right
      * No real guide for this. It'll vary by device so you'll need to find out for yourself. If you don't feel comfortable doing this, save yourself a brick and don't bother

* init.mount_datacommon.sh:
  * The script that's called at boot to mount common data partition
  * You may need to change sdcardfs on 11 to /mnt/pass_through/0/emulate/0/CommonData (line 19)

* tools/magisk.sh:
  * Install magisk
  * You'll likely need to update this with future magisk releases - just pull relevant stuff from magisk zip scripts

* magiskcommon/*:
  * These files are taken straight from the magisk zip, no changes needed here
