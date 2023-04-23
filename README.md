## Running linux on the seL4 vm
It uses a raspberry pi 4b 4GB model, offering the follow functions:
- full-scale linux kernel from raspberry pi OS image
- passthrough devices including: uart port, usb, sdhc
- full-scale rootfs mounted from sd card
- wifi on board enabled

### Prerequisites
- assuming you have successfully ran the seL4 kernel in your raspberry pi (refer to the [official totuial](https://docs.sel4.systems/Hardware/Rpi4.html)).
- a sd card flashed with the raspberry 4b's official OS image (better using ```Raspberry OS Lite (64bit)```).

### Usage
1. initialize the camkes-vm repo
```
mkdir camkes-vm
cd camkes-vm
repo init -u https://github.com/seL4/camkes-vm-examples-manifest.git
repo sync
```

2. apply the patch file
```
git clone https://github.com/colorglass/sel4_vm_rpi4.git

# apply patch
cp sel4_vm_rpi4/apply-patch.sh ..
cp sel4_vm_rpi4/sel4_vm_rpi4.patch ..
source apply-patch.sh sel4_vm_rpi4.patch

cp sel4_vm_rpi4/linux projects/camkes-vm-images/rpi4/
cp sel4_vm_rpi4/rootfs.cpio.gz projects/camkes-vm-images/rpi4/
```

4. build project
```
mkdir build
cd build
../init-build.sh -DCAMKES_VM_APP=vm_minimal -DPLATFORM=rpi4
ninja
ls images/
capdl-loader-image-arm-bcm2711
```
5. copy the image ```capdl-loader-image-arm-bcm2711``` to the boot sector of sd card (you can also use usb or tftp to pass seL4 image)

6. modify the ```config.txt``` of the rpi4's firmware
```
# adding the follow lines

kernel=u-boot.bin       # you should alreadly added this
dtoverlay=disable-bt        # we use the pl011 as the primary uart
```
7. load seL4 image using u-boot
```
# using uart to get console

U-Boot> fatload usb 0:1 ${loadaddr} capdl-loader-image-arm-bcm2711
U-Boot> bootefi ${loadaddr}

```
