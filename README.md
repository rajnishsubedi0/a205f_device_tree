# a205f_device_tree
Device tree for a205f with android 11 manifest
# My experience with it
Main error i encounter was about prebuilt kernel and dtbo image. Name mismatched as that was defined in BoardConfig.mk. 
# Building instruction
https://xdaforums.com/t/guide-to-twrp-building.4515895/

# Instruction from Xda i have followed
1. Setup
--------

sudo apt update
sudo apt upgrade
sudo apt-get install git-all

sudo apt install python-is-python3

git config --global user.email "your email"
git config --global user.name "your name"


2. swapfile
-----------

sudo swapoff -a

sudo dd if=/dev/zero of=/swapfile bs=1G count=8
sudo mkswap /swapfile
sudo swapon /swapfile

free -m

3. repo
-------

mkdir -p ~/.bin
PATH="${HOME}/.bin:${PATH}"
curl https://storage.googleapis.com/git-repo-downloads/repo > ~/.bin/repo
chmod a+rx ~/.bin/repo



Basic Device Tree
==================

mkdir TWRP
cd TWRP
git init
git clone https://github.com/twrpdtgen/twrpdtgen
pip3 install twrpdtgen
sudo apt install cpio
copy stock recovery.img to ~/TWRP/twrpdtgen

cd twrpdtgen
python3 -m twrpdtgen recovery.img

rename ~/TWRP/twrpdtgen/output/samsung/a12s/omni_a12s.mk to twrp_a12s.mk

delete vendorsetup.sh

Add this to recovery.fstab

# Removable storage
/external_sd vfat /dev/block/mmcblk1p1 /dev/block/mmcblk1 flags=storage;wipeingui;removable
/usb-otg auto /dev/block/sda1 /dev/block/sda flags=display="USB-OTG";storage;wipeingui;removable

Make a copy of recovery.fstab and rename to twrp.flags
Move both files recovery.fstab and twrp.flags to recovery/root/system/etc
(create these folders)

open twrp_a12s.mk and
1. delete the 3 lines below # Inherit from those products, Most specific first.
replace those 3 lines with this
$(call inherit-product, $(SRC_TARGET_DIR)/product/aosp_base.mk)

2. delete $(call inherit-product, vendor/omni/config/gsm.mk)
3. change omni to twrp (2 places)

open AndroidProducts.mk
change omni to twrp (4 spots)

Optional: add extra items to BoardConfig.mk


cd ~
sudo apt install git aria2 -y
git clone https://gitlab.com/OrangeFox/misc/scripts
cd scripts
sudo bash setup/android_build_env.sh
sudo bash setup/install_android_sdk.sh


• Make a directory called ~/a12s-twrp
cd ~/a12s-twrp

repo init --depth=1 -u https://github.com/minimal-manifest-twrp/platform_manifest_twrp_aosp.git -b twrp-11


repo sync -c --no-clone-bundle --no-tags --optimized-fetch --prune --force-sync -j1

copy ~/TWRP/twrpdtgen/output/samsung to /a12s-twrp/device


Building
========

export ALLOW_MISSING_DEPENDENCIES=true
. build/envsetup.sh
lunch twrp_a12s-eng
mka recoveryimage

Recovery.img at out/target/product/a12s/recovery.img
 
