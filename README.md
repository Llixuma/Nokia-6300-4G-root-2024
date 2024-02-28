##### please do these steps in a vm or a seperate pc that you dont mind wiping if you dont trust the needed programs to keep your system stable

### I'm using arch Linux in these steps, keep in mind that the steps might be more cumbersome on other distros on account of not having an aur helper

## important: if edl-git does not work on arch, you can download the liveiso from the edl repo and extract and upload the files from there

#### 1. 
Install edl-git through yay (more reliable than going through the manual python setup for edl)
``` bash
yay -S edl-git
```
#### 2. 
Boot the phone into edl mode (from off state hold star + hash and insert usb cable)
#### 3.
``` bash
# just the boot partition
edl r boot boot.img

# for exploration you can get the entire flash with
edl rf flashdump.img
# this dump can be mounted in kde with Disk Image Mounter,
# exposing the "persist, cache, modem, system and data" partitions for exploration in your file manager of choice
```
#### 4. 
Copy boot.img to a preferably empty project directory
#### 5. 
Install abootimg:
``` bash
yay -S abootimg
```
#### 6. 
Download adbd,bootpatch.sh,slua and sluac from 
https://gitlab.com/suborg/8k-boot-patcher
#### 7. 
Go through bootpatch.sh (manual if you prefer or want to explore the extracted files)
```
EARLYBOOT_FILE=init.qcom.early_boot.sh 
was omitted from the script here for copy paste friendliness.
Usage of $(EARLYBOOT_FILE) was replaced with init.qcom.early_boot.sh
```
###### bootpatch.sh:
``` bash
#!/bin/sh
# assume we have image dir mounted to /image and image/boot.img present
cd /image
cp boot.img boot-orig.img
echo 'Boot image found, patching...'
abootimg -x boot.img
mkdir initrd
cd initrd
cat ../initrd.img | gunzip | cpio -vid
# initrd root patch process start
cp /home/8k/adbd ./sbin/
cp /home/8k/slua ./sbin/
cp /home/8k/sluac ./sbin/
sed -i 's/ro\.secure.*/ro.secure=0/' ./default.prop
sed -i 's/ro\.debuggable.*/ro.debuggable=1/' ./default.prop
sed -i 's/.*perf_harden.*/security.perf_harden=0/' ./default.prop
sed -i '/.*reload_policy.*/d' ./init.rc
echo 'setenforce 0' >> ./init.qcom.early_boot.sh
echo 'echo -n 1 > /data/enforce' >> ./init.qcom.early_boot.sh
echo 'mount -o bind /data/enforce /sys/fs/selinux/enforce' >> ./init.qcom.early_boot.sh
# initrd root patch process end
find . | cpio --create --format='newc' | gzip > ../myinitrd.img
cd ..
# bootimg.cfg patch process start
sed -i 's/^cmdline.*/& androidboot.selinux=permissive enforcing=0/' bootimg.cfg
# bootimg.cfg patch process end
abootimg -u boot.img -f bootimg.cfg -r myinitrd.img
rm -rf *initrd* bootimg.cfg zImage
echo 'Boot image patched!'
# ...Nothing can stop an idea whose time has come
```
#### 8. 
Upload the created boot.img with
``` bash
edl w boot boot.img
```
#### 9. 
reboot to OS with
``` bash
edl reset
```
#### 10. 
(after enabling debug mode with `*#*#33284#*#*`)
you should have a root shell with adb shell now:
``` shell
user@localpc ~> adb shell
* daemon not running; starting now at tcp:5037
* daemon started successfully
root@Nokia 6300 4G:/ # whoami
root
root@Nokia 6300 4G:/ #
```

should you encounter any errors with the setup itself and have feedback regarding this tutorial, please let me know how to improve it (besides making it NOT arch linux dependant)
