Aosp Android Automotive Android Framework Yocto Android App Development 

# **Build AOSP Android 16 Automotive OS for Raspberry Pi 5 and flash it.** 

This article outlines the process of building Android 16 Automotive OS for the Raspberry Pi 5, including the setup of the AOSP build environment, creating a flashable SD card image, building the BSP (kernel, device trees, overlays) with Yocto and replacing it in the Android build, flashing the device, and highlighting the issues, challenges, and key takeaways encountered along the way. 

**Prepared by DHINESH E** · <u>dhinesh701@gmail.com</u> 



<!-- Start of picture text -->
09:20 Android 16<br>@ 24°C<br>Automotive OS<br>Weather<br>on Raspberry Pi 5<br>Media<br>AOSP build · SD card image · flash<br>>>il —  — ———<br>− 22 + − 22 +<br>|<br>aon Oi<br><!-- End of picture text -->

The Raspberry Pi 5 serves as a robust platform for exploring Android Automotive and IoT innovations. This guide takes you through the process of building AOSP Android 16 for the Raspberry Pi 5, integrating it smoothly with the hardware, and discovering new possibilities beyond traditional use cases. Whether you’re an Android enthusiast or a developer eager to experiment with the latest technology, this article offers a clear and practical roadmap to get you started. 

Let’s dive in! 

## **Devices Configuration and Hardware Setup** 

### **Build PC configurations for me:** 

CPU: AMD Ryzen 9 9950TM-16 Core 

1 

- OS: Ubuntu 22.04.5 LTS 

- RAM: 64 GB 

- SSD: 500 GB 

### **Hardware List:** 

- SD Card 64 GB + USB card reader 

- Power supply Adapter 5.1 V 

- Rasberypi 5 Modle B Board without computation modle. 

- UCB to Type C Cable 

- Mini HDMi to HDMI cable 



<!-- Start of picture text -->
CcSS ><br>ee Ate &<br>WeaaSo N 7 a<br>= . co<br>7<br>e es<br>eo: Oe \ cena<br>we<br>= 2.x MICRO-HOMI® PORTS:<br><!-- End of picture text -->

### **Tools on Build Machine:** 

- Git, Repo 

- OpenJDK-17-jdk 

- imager_1.8.5_amd64 or Balena-etcher 

The setup described above is required for this task. However, if you have alternative configurations that work reliably, those are equally suitable to get started. 

### **OpenJDK-17-jdk** 

```
sudo apt-get update
sudo apt-get install openjdk-17-jdk
java -version
```

2 

### **Git** 

```
sudo apt-get install git
git config --global user.name "Your Name"
git config --global user.email "youremail@domain.com"
```

## **Download Android 16 AOSP Source for Raspberry Pi 5** 

### **Prepare build environment (Ubuntu 22.04 LTS recommended)** 

```
sudo apt-get update
```

```
sudo apt-get install git-core gnupg flex bison gperf build-essential \
    zip curl zlib1g-dev gcc-multilib g++-multilib libc6-dev-i386 lib32ncurses5-dev \
    x11proto-core-dev libx11-dev lib32z-dev ccache libgl1-mesa-dev libxml2-utils \
    xsltproc unzip fontconfig
```

The packages installed using `sudo apt-get install` . These are essential for setting up the Android Open Source Project (AOSP) build environment. `git-core` is used for source code management and repository synchronisation, while `gnupg` provides secure authentication and verification of downloads. Tools such as `flex` , `bison` , and `gperf` are required for parsing, code generation, and configuration processing. The `build-essential` package installs core compilation tools, including `gcc` , `g++` , and `make` . Utilities like `zip` , `unzip` , and `curl` handle packaging, extraction, and downloading of files. Libraries such as `zlib1g-dev` and `lib32z-dev` Provide the compression support required by Android build tools. On 64-bit hosts, `gcc-multilib` , `g++-multilib` , and `libc6-dev-i386` enable 32-bit compatibility, while `lib32ncurses5-dev` providing terminal handling for 32-bit builds. X11 libraries, including `x11proto-core-dev` and `libx11-dev` , are necessary for building graphical components. `ccache` improves build performance by caching compilation outputs, and `libgl1-mesa-dev` supplies OpenGL libraries for rendering. `libxml2-utils` and `xsltproc` enable XML and XSLT processing for build scripts, while `fontconfig` ensuring proper font handling and configuration. Together, these tools provide the compilers, libraries, and system dependencies required to build Android from source successfully. 

Installing these packages may take some time. Please allow the process to complete, and ensure all packages are installed successfully. If any errors occur, monitor the installation logs to identify and resolve them. 

### **Install the repo tool:** 

If you have built AOSP before, you may have already completed this step and have `repo` installed and set up on your PC. 

You can skip the “Install the repo tool” step if it was already installed during a previous AOSP setup. In that case, you will see the following message: 

3 

```
... A new version of repo (2.54) is available.
... You should upgrade soon:
```

Then, specifically repeat “Install the repo tool:”. After repeating “Install the repo tool:”, continue with the next step. 

```
mkdir -p ~/bin
curl https://storage.googleapis.com/git-repo-downloads/repo > ~/bin/repo
chmod a+x ~/bin/repo
export PATH=~/bin:$PATH
```

### **Create Workspace:** 

```
mkdir Android16
cd Android16
```

### **Initialise repo:** 

I am using the AOSP Android 16.0.0_r1 branch, which is the initial public release of Android 16. This branch provides a stable and well-documented baseline, making it ideal for development, experimentation, and building custom images for the Raspberry Pi 5. Additionally, the local manifest repository https://github.com/raspberry-vanilla/android_local_manifest mentions updates for AOSP Android 16.0.0_r1, which is why we will use it for our setup. 

```
repo init -u https://android.googlesource.com/platform/manifest -b android-16.0.0_r1
```

#### **Output:** 

```
repo: reusing existing repo client checkout in ../..WorkSpace/AOSP/Android16
repo has been initialized in ../../WorkSpace/AOSP/Android16
```

### **Add the Raspberry Pi Local Manifest** 

You can find the local manifest for Raspberry Pi here: https://github.com/raspberryvanilla/android_local_manifest 

#### **Required tools:** 

```
sudo apt-get install dosfstools e2fsprogs fdisk kpartx mtools rsync
```

#### **Adding Local Manifest:** 

```
mkdir -p .repo/local_manifests
```

```
git clone https://github.com/raspberry-vanilla/android_local_manifest.git .repo/local_manife
sts -b android-16.0
```

4 

**|| OR ||** 

```
curl -o .repo/local_manifests/manifest_brcm_rpi.xml -L https://raw.githubusercontent.com/ras
pberry-vanilla/android_local_manifest/android-16.0/manifest_brcm_rpi.xml --create-dirs
```

#### **|| OR ||** 

_Or optionally, you can reduce download size by creating a shallow clone and removing unneeded projects:_ 

```
repo init -u https://android.googlesource.com/platform/manifest -b android-16.0.0_r1 --depth
=1
curl -o .repo/local_manifests/manifest_brcm_rpi.xml -L https://raw.githubusercontent.com/ras
pberry-vanilla/android_local_manifest/android-16.0/manifest_brcm_rpi.xml --create-dirs
curl -o .repo/local_manifests/remove_projects.xml -L https://raw.githubusercontent.com/raspb
erry-vanilla/android_local_manifest/android-16.0/remove_projects.xml
```

This manifest automatically pulls in `device/raspberry/*` , `kernel` , and vendor repos for RPi5. 

I ran the first command, and the output appears as follows: 

```
Cloning into '.repo/local_manifests'...
remote: Enumerating objects: 329, done.
remote: Counting objects: 100% (86/86), done.
remote: Compressing objects: 100% (26/26), done.
remote: Total 329 (delta 74), reused 60 (delta 60), pack-reused 243 (from 2)
Receiving objects: 100% (329/329), 45.64 KiB | 730.00 KiB/s, done.
Resolving deltas: 100% (166/166), done.
```

**Note:** The manifest in this repository https://github.com/raspberryvanilla/android_local_manifest has been updated for Android 16.0.0_r1. Ensure that you build AOSP for the corresponding branch if you plan to flash it on a Raspberry Pi. 

### **Sync source:** 

```
repo sync -j$(nproc --all)
or
repo sync -j[n] == repo sync -j12
```

In `-j<n>` , where `n` is the number of cores available on your build machine. It’s recommended to leave 4 cores for the system and use the remaining cores for the build. To ensure better performance and avoid repo sync failures, explicitly specify the number of cores when running repo sync using `repo sync -j<n>` . 

Here’s an optimised `repo sync` command 

5 

```
repo sync -c -j12 --force-sync --no-clone-bundle --no-tags
```

- `-c` → Sync only the current branch (saves space/time vs all branches). 

- `-j12` → Run 12 parallel jobs (matches your cores). 

- `--force-sync` → Cleans up and resyncs repos if something gets corrupted. 

- `--no-clone-bundle` → Disables using prebuilt Git bundle (smaller initial download, fewer failures). 

- `--no-tags` → Skips fetching Git tags (faster & smaller checkout, not needed for builds). 

I have not used any of these options. Try adding them and observing the result. 

#### **Repo sync is completed!** 



<!-- Start of picture text -->
Receiving objects: 100% (2794/2794), 1.88 MiB | 112.97 MiB/s, done.<br>Resolving deltas: 100% (14/14), done.<br>Receiving objects: 100% (2868/2868), 962.98 KiB | 82.09 MiB/s, done.eiving objects: 0% (1/2868)<br>Resolving deltas: 100% (19/19), done.<br>Receiving objects: 100% (3262/3262), 1.15 MiB | 73.92 MB/s, done.<br>Resolving deltas: 100% (211/211), done.<br>Receiving objects: 100% (3876/3876), 1.51 MiB | 73.51 MiB/s, done.eceiving objects: 0% (1/3876)<br>Resolving deltas: 100% (114/114), done.<br>Receiving objects: 100% (13073/13073), 3.18 MiB | 69.29 MiB/s, done.eiving objects: 0% (1/13073)<br>Resolving deltas: 100% (7579/7579), done.<br>Receiving objects: 100% (22454/22454), 3.42 MiB | 68.66 MiB/s, done.eiving objects: 0% (1/22454)<br>Resolving deltas: 100% (11860/11860), done.<br>Receiving objects: 100% (3418/3418), 1.02 MiB | 61.15 MiB/s, done.ecetving objects: 0% (1/3418)<br>Resolving deltas: 100% (128/128), done.<br>Receiving objects: 100% (5468/5468), 7.51 MiB | 124.00 MiB/s, done.ceiving objects: 0% (1/5468)<br>Resolving deltas: 100% (3819/3819), done.<br>Receiving objects: 100% (3611/3611), 1.10 MiB | 86.72 MiB/s, done.eceiving objects: 0% (1/3611)<br>Resolving deltas: 100% (591/591), done.<br>Receiving objects: 100% (4520/4520), 1.49 MiB | 76.23 MB/s, done.eceiving objects: 82% (103138/125777), 45.<br>Resolving deltas: 100% (550/550), done.<br>Receiving objects: 100% (125777/125777), 134.71 MiB | 99.17 MiB/s, done.ng objects: 87% (109426/125777), 45.<br>Resolving deltas: 100% (109906/109906), done.8 platform/cts @ ctsResolving deltas: 95% (104411/109906)<br>Receiving objects: 100% (2830/2830), 1.84 MiB | 104.83 MiB/s, done.ceiving objects: 0% (1/2830)<br>Resolving deltas: 100% (97/97), done.<br>Syncing: 25% (250/982) 16:51 | 12 jobs | 16:49 platform/cts @ cts[]<br><!-- End of picture text -->

_Repo sync completed after 2 hrs 13 min!!_ 

### **Set up Android build environment:** 

```
source build/envsetup.sh
```

#### **Check for lunch options:** 

```
lunch
```

#### **Output Lunch Options:** 

As observed when building Android 15, the lunch options are similarly available in Android 16. 

6 

```
aosp_rpi4_car-trunk_staging-userdebug
aosp_rpi4-trunk_staging-userdebug
aosp_rpi4_tv-trunk_staging-userdebug
---> aosp_rpi5_car-trunk_staging-userdebug
aosp_rpi5-trunk_staging-userdebug
aosp_rpi5_tv-trunk_staging-userdebug
```

While building Android 15, the lunch options were not visible when I ran the `lunch` command. Refer to the **"Build Issues"** section below for details. 

Lunch option Android 16 for Rpi 5 is → 

```
lunch aosp_rpi5_car-bp2a-userdebug
```

#### **Output:** 

```
============================================
PLATFORM_VERSION_CODENAME=REL
PLATFORM_VERSION=16
TARGET_PRODUCT=aosp_rpi5_car
TARGET_BUILD_VARIANT=userdebug
TARGET_ARCH=arm64
TARGET_ARCH_VARIANT=armv8-a
TARGET_CPU_VARIANT=cortex-a76
HOST_OS=linux
HOST_OS_EXTRA=Linux-6.8.0-64-generic-x86_64-Ubuntu-22.04.5-LTS
HOST_CROSS_OS=windows
BUILD_ID=BP2A.250605.031.A2
OUT_DIR=out
SOONG_ONLY=false
============================================
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
  Wondering whether to use user, userdebug or eng?
  user       The builds that ship to users. Reduced debugability.
  userdebug  High fidelity to user builds but with some debugging options
             enabled. Best suited for performance testing or day-to-day use
             with debugging enabled.
  eng        More debugging options enabled and faster build times, but
             runtime performance tradeoffs. Best suited for day-to-day
             local development when not doing performance testing.
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
```

## **Build AOSP-16 System Images** 

```
make bootimage systemimage vendorimage -j$(nproc)
# nproc can be replaced accordingly, 12 for me.
make bootimage systemimage vendorimage -j12
```

7 

_The build has started! Time to grab a coffee._ 



<!-- Start of picture text -->
PLATFORM_VERSION_CODENAME=REL<br>PLATFORM_VERSION=16<br>TARGET_PRODUCT=aosp_rpiS_car<br>TARGET_BUILD_VARIANT=userdebug<br>TARGET_ARCH=arm64<br>TARGET_ARCH_VARIANT=armv8-a<br>TARGET_CPU_VARIANT=cortex-a76<br>HOST_OS=linux<br>HOST_OS_EXTRA=Linux-6.8.0-64-generic-x86_64-Ubuntu-22.04.5-LTS<br>HOST_CROSS_OS=windows<br>BUILD_ID=BP2A.250605.031.A2<br>OUT_DIR=out<br>SOONG_ONLY=false<br>[ 99% 132/133] analyzing Android.bp files and generating ninja file at out/soong/build.aosp_rpiS_car.ninja<br>analyzing Android.bp files and generating ninja file at out/soong/build.aosp_rpiS_car.ninja<br><!-- End of picture text -->

#### **Build Completed!!** 



<!-- Start of picture text -->
[ 99% 150357/150366] //external/icu/android_icu4j:stable.i18n.module.platform.api.stubs.source metalava merg|<br>external /icu/android_icu4j/src/main/java/android/icu/text/Transliterator. java:455: Public class and<br>ext.StringTransform (ErrorWhenNew) [HiddenSuperclass<br>external/icu/android_icu4j/src/main/java/android/icu/text/Transliterator.java:455: Public class and<br>ext.Transform (ErrorWhenNew) [HiddenSuperclass<br>external/icu/android_icu4j/src/main/java/android/icu/text/UCharacterIterator.java:30: Public class<br>d.icu.text.UForwardCharacterIterator (ErrorwhenNew) [HiddenSuperclass]<br>[ 99% 150365/150366] //frameworks/base/api:systen-api-stubs-docs-non-updatable metalava merged (priority: ©)<br>frameworks /base/telecomm/java/android/teLecom/TelecomManager . java:2853: When a method overload is nj<br>ext.createPackageContextAsUser() and re-obtain the relevant Manager, and no new API should be added (Errorwh,<br>metalava wrote updated baseline to /media/narendra147h/UbuntuDrive/WorkSpace/AOSP/Android16/out/soong/.temp/<br>[100% 150366/150366] //frameworks/base/api:system-api-stubs-docs-non-updatable check current API (priority<br><!-- End of picture text -->

This will generate files such as `out/target/product/rpi5/system.img` , `boot.img` , and others. With the Raspberry Pi build now complete, we can proceed to prepare the flashing image. 

#### **Let's check that!!** 

```
cd out/target/product/rpi5/
ls -la
```

8 



<!-- Start of picture text -->
4096 Sep 7 11:47<br>4096 Sep 7 10:26<br>2132029 Sep 7 10:41 all_modules.txt<br>14 Sep 7 11:46 android-info.txt<br>4096 Sep 7 11:46<br>134217728 Sep 7 11:46 boot.img<br>82 Sep 7 10:26 build_fingerprint.txt<br>53 Sep 7 10:41 build_thumbprint.txt<br>169381 Sep 7 10:41 clean_steps.mk<br>37 Sep 7 10:41 .copied_headers_list<br>4096 Sep 7 11:46<br>12288 Sep 7 11:46<br>4096 Sep 7 11:46<br>4096 Sep 7 11:38<br>3379189 Sep 7 10:41 .installable_files.previous<br>539076 Sep 7 11:46 installed-files.json<br>286 Sep 7 11:46 installed-files-ramdisk.json<br>62 Sep 7 11:46 installed-files-ramdisk.txt<br>3139 Sep 7 11:46 installed-files-root.json<br>674 Sep 7 11:46 installed-files-root.txt<br>186925 Sep 7 11:46 installed-files.txt<br>89794 Sep 7 11:46 installed-files-vendor.json<br>28145 Sep 7 11:46 installed-files-vendor.txt<br>35371520 Sep 7 10:48<br>4096 Sep 7 11:46<br>49 Sep 7 10:42 previous_build_config.mk<br>© Sep 7 10:42 previous_use_partial_compile.txt<br>16619 Sep 7 11:43 product_packages.txt<br>4096 Sep 7 11:46<br>1831354 Sep 7 11:46 ramdisk.img<br>4096 Sep 7 11:46<br>4096 Sep 7 11:46<br>4096 Sep 7 11:48<br>4096 Sep 7 11:46<br>2684354560 Sep 7 11:47 system.img<br>4096 Sep 7 11:46<br>268435456 Sep 7 11:46 vendor.img<br><!-- End of picture text -->

The `RaspberryVanillaAOSP16–20250907-rpi5_car.img` has been generated, which is our final target. Let’s proceed to build the final image. 

### **Build RaspberryVanillaAOSP16–20250907-rpi5_car.img** 

```
croot # go back to root dir
```

In the Root dir of AOPS “Android16/”, you will see the following files: 



<!-- Start of picture text -->
25 Sep 7 00:58 -><br>25 Sep 7 00:58 -><br>25 Sep 7 00:58 -><br>25 Sep 7 00:58 -><br>4096 Sep 7 08:42<br><!-- End of picture text -->

#### **Run:** 

```
./rpi5-mkimg.sh
```

9 

#### **Success Output:** 



<!-- Start of picture text -->
Image mounted as /dev/100p20<br>Copying boot...<br>128+0 records in<br>128+0 records out<br>134217728 bytes (134 MB, 128 MiB) copted, 0.120549 s, 1.1 GB/s<br>Copying systen...<br>2560+8 records in<br>256040 records out<br>2684354560 bytes (2.7 GB, 2.5 GiB) copied, 1.3428 s, 2.0 GB/s<br>Copying vendor...<br>256+0 records in<br>256+0 records out<br>268435456 bytes (268 MB, 256 MLB) copied, 0.631024 s, 425 MB/s<br>Creating userdata...<br>nke2fs 1.46.5 (30-Dec-2621)<br>Discarding device blocks: done<br>Creating filesystem with 2996080 4k blocks and 749248 inodes<br>Filesystem UID: 4b6b304b-74e8-4211-a282-b7bd47596862<br>Superblock backups stored on blocks:<br>32768, 98304, 163840, 229376, 294912, 819200, 884736, 1605632, 2654208,<br>Allocating group tables: done<br>Writing tnode tables: done<br>Creating journal (16384 blocks): done<br>Writing superblocks and filesystem accounting information: done<br>Done, created /media/narendra147h/Ubuntudrive/Workspace/A0SP/Androidi6/out<br>target /product/rpis/RaspberryVantLLaA0SP16-20250907-rptS_car-ing!<br><!-- End of picture text -->

**Note:** The final part of the output — `"Done, created out/target/product/rpi5/RaspberryVanillaAOSP16–20250907-rpi5_car.img!"` — should appear when the build completes successfully. If any errors occur, this message will not be displayed, and an error will be shown instead. 

### **Verify Images** 

Once the build is complete, verify that the following files are present: `RaspberryVanillaAOSP16–20250907-rpi5_car.img` , `system.img` , `vendor.img` , and `boot.img` . 

```
cd ../../android-aosp/out/target/product/rpi5/
ls -la
```

10 



<!-- Start of picture text -->
4096 Sep 7 12:03<br>4096 Sep 7 10:26<br>2132029 Sep 7 10:41 all_modules.txt<br>14 Sep 7 11:46 android-info.txt<br>4096 Sep 7 11:46<br>134217728 Sep 7 11:46 boot.img<br>82 Sep 7 10:26 build_fingerprint.txt<br>53 Sep 7 10:41 build_thumbprint.txt<br>169381 Sep 7 10:41 clean_steps.mk<br>37 Sep 7 10:41 .copied_headers_list<br>4096 Sep 7 11:46<br>12288 Sep 7 11:46<br>4096 Sep 7 11:46<br>4096 Sep 7 11:38<br>3379189 Sep 7 10:41 .installable_files.previous<br>539076 Sep 7 11:46 installed-files.json<br>286 Sep 7 11:46 installed-files-ramdisk.json<br>62 Sep 7 11:46 installed-files-ramdisk.txt<br>3139 Sep 7 11:46 installed-files-root.json<br>674 Sep 7 11:46 installed-files-root.txt<br>186925 Sep 7 11:46 installed-files.txt<br>89794 Sep 7 11:46 installed-files-vendor.json<br>28145 Sep 7 11:46 installed-files-vendor.txt<br>35371520 Sep 7 10:48<br>4096 Sep 7 11:46<br>49 Sep 7 10:42 previous_build_config.mk<br>© Sep 7 10:42 previous_use_partial_compile.txt<br>16619 Sep 7 11:43 product_packages.txt<br>4096 Sep 7 11:46<br>1831354 Sep 7 11:46 ramdisk.img<br>15360000000 Sep 7 12:03 RaspberryVanillaAOSP16-20250907-rpi5_car.img<br>4096 Sep 7 11:46<br>4096 Sep 7 11:46<br>4096 Sep 7 11:48<br>4096 Sep 7 11:46<br>2684354560 Sep 7 11:47 system.img<br>4096 Sep 7 11:46<br>268435456 Sep 7 11:46 vendor.img<br><!-- End of picture text -->

Output contains data `RaspberryVanillaAOSP16–20250907-rpi5_car.img` generated with a size of 15360000000. 

## **Yocto: Build the Raspberry Pi 5 BSP (Kernel, DTBs, Overlays)** 

The Android build for Raspberry Pi 5 does not compile the Linux kernel. It takes a ready-made kernel `Image` , device tree blobs ( `.dtb` ), overlays ( `.dtbo` ) and kernel modules from the BSP folder `device/brcm/rpi5-kernel` and packs them into `boot.img` . This section shows how to build those BSP files with the **Yocto Project** ( `metaraspberrypi` layer) and replace them in the Android 16 source tree. 

#### **Before you start:** 

- Android 16 on Raspberry Pi 5 uses Linux kernel **6.12** . Build the same kernel series in Yocto. 

- The Yocto kernel ( `linux-raspberrypi` ) comes from the Raspberry Pi kernel tree, not the Android Common Kernel. It **must** have the Android kernel config 

11 

options added (Step Y5); otherwise Android will not boot. 

Keep a backup of the original BSP files so you can roll back (Step Y8). 

### **Y1. Yocto release used** 

This guide uses **Yocto Project 6.0 “Wrynose” (LTS)** , supported until April 2030, with the `wrynose` branch of `meta-raspberrypi` . From Yocto 6.0 the separate _poky_ repository is no longer used; the layers (bitbake, openembedded-core, meta-yocto) are cloned individually. 

|**Repository**|**Branch**|
|---|---|
|bitbake|`2.18`|
|openembedded-core|`wrynose`|
|meta-yocto (poky distro)|`wrynose`|
|meta-raspberrypi (BSP layer)|`wrynose`|



### **Y2. Install Yocto host packages (Ubuntu 22.04 / 24.04)** 

```
sudo apt-get update
sudo apt-get install build-essential chrpath cpio debianutils diffstat file gawk gcc \
    git iputils-ping libacl1 lz4 locales python3 python3-git python3-jinja2 \
    python3-pexpect python3-pip python3-subunit socat texinfo unzip wget xz-utils zstd
sudo locale-gen en_US.UTF-8
```

Plan for at least **100 GB** of free disk space for the Yocto build directory. 

### **Y3. Yocto sync (download the layers)** 

```
mkdir -p ~/yocto-rpi5/layers
cd ~/yocto-rpi5/layers
```

```
git clone -b 2.18    https://git.openembedded.org/bitbake
git clone -b wrynose https://git.openembedded.org/openembedded-core
git clone -b wrynose https://git.yoctoproject.org/meta-yocto
git clone -b wrynose https://github.com/agherzan/meta-raspberrypi
```

#### To **re-sync** (update) all layers later: 

```
cd ~/yocto-rpi5/layers
for d in bitbake openembedded-core meta-yocto meta-raspberrypi; do
    git -C $d pull --ff-only
done
```

12 

### **Y4. Create the build directory and add layers** 

```
cd ~/yocto-rpi5
source layers/openembedded-core/oe-init-build-env build
bitbake-layers add-layer ../layers/meta-yocto/meta-poky
bitbake-layers add-layer ../layers/meta-raspberrypi
bitbake-layers show-layers
```

Edit `conf/local.conf` and add: 

```
MACHINE = "raspberrypi5"
DISTRO = "poky"
# Android 16 uses kernel 6.12
PREFERRED_VERSION_linux-raspberrypi = "6.12.%"
# Also build the overlays used by Android's config.txt
RPI_KERNEL_DEVICETREE_OVERLAYS:append = " overlays/cma.dtbo"
# Parallel build (adjust to your CPU)
BB_NUMBER_THREADS = "12"
PARALLEL_MAKE = "-j 12"
```

### **Y5. Add the Android kernel configuration (custom layer)** 

Create a small custom layer that adds Android’s required kernel options to the Yocto kernel recipe: 

```
cd ~/yocto-rpi5/build
bitbake-layers create-layer ../layers/meta-rpi-android
bitbake-layers add-layer ../layers/meta-rpi-android
mkdir -p ../layers/meta-rpi-android/recipes-kernel/linux/files
```

Copy Android’s base kernel config from the Android 16 source tree (the `kernel/configs` project) into the layer as a config fragment: 

```
cp ~/Android16/kernel/configs/b/android-6.12/android-base.config \
   ~/yocto-rpi5/layers/meta-rpi-android/recipes-kernel/linux/files/android-base.cfg
```

Create `../layers/meta-rpi-android/recipes-kernel/linux/linux-raspberrypi_%.bbappend` : 

```
FILESEXTRAPATHS:prepend := "${THISDIR}/files:"
SRC_URI += "file://android-base.cfg"
```

The fragment enables Binder/BinderFS, SELinux, cgroups/BPF, FUSE, dm-verity and the other options Android 16 expects. 

### **Y6. Yocto build procedure** 

Build only the BSP pieces Android needs (kernel Image, DTBs, overlays, modules): 

13 

```
cd ~/yocto-rpi5
source layers/openembedded-core/oe-init-build-env build
bitbake virtual/kernel
```

Check that the Android options were applied (look for warnings about options that could not be set): 

```
bitbake -c kernel_configcheck -f virtual/kernel
```

_Optional:_ build a complete Yocto Linux image to test the board with Yocto itself: 

```
bitbake core-image-base
```

**Build output** is in `build/tmp/deploy/images/raspberrypi5/` : 

- `Image` — kernel image 

`bcm2712-rpi-5-b.dtb` (and other `bcm2712*.dtb` ) — device trees 

- `*.dtbo` (under `overlays/` ) — device tree overlays 

- `modules-raspberrypi5.tgz` — kernel modules 

`core-image-base-raspberrypi5*.wic.bz2` — full Yocto SD image (only if you built it) 

```
ls -la build/tmp/deploy/images/raspberrypi5/
```

## **Replace the Yocto Build Images in the Android BSP Layer** 

The Android BSP folder `device/brcm/rpi5-kernel` contains: 

```
device/brcm/rpi5-kernel/
├── Image                       # kernel  → copied to boot.img
├── bcm2712-rpi-5-b.dtb         # device trees (+ rpi-500, cm5, d0 variants)
├── overlays/                   # *.dtbo overlays → boot.img/overlays
└── modules/                    # kernel modules (*.ko)
```

When you run `make bootimage` , the Android build copies `Image` , `bcm2712*-rpi-*.dtb` and `overlays/*` from this folder into the FAT32 `boot.img` , together with `config.txt` , `cmdline.txt` and `ramdisk.img` . 

### **Y7. Set paths** 

```
export AOSP=~/Android16
export DEPLOY=~/yocto-rpi5/build/tmp/deploy/images/raspberrypi5
export KDIR=$AOSP/device/brcm/rpi5-kernel
```

14 

### **Y8. Back up the original BSP files** 

```
cd $KDIR
git checkout -b yocto-kernel        # work on a separate branch
# To roll back later:  git checkout android-16.0 -- .   (or: git checkout .)
```

### **Y9. Copy kernel Image and device trees** 

```
cp -L $DEPLOY/Image $KDIR/Image
for dtb in $KDIR/bcm2712*.dtb; do
    name=$(basename $dtb)
    src=$(find -L $DEPLOY -maxdepth 2 -name "$name" | head -1)
    if [ -n "$src" ]; then cp -L "$src" "$dtb"; echo "updated $name"
    else echo "NOT built by Yocto, kept original: $name"; fi
done
```

### **Y10. Copy overlays** 

```
for dtbo in $KDIR/overlays/*.dtbo; do
    name=$(basename $dtbo)
    src=$(find -L $DEPLOY -name "$name" | head -1)
    [ -n "$src" ] && cp -L "$src" "$dtbo"
done
# Overlays used by Android's config.txt must be present:
ls $KDIR/overlays/vc4-kms-v3d*.dtbo $KDIR/overlays/cma.dtbo $KDIR/overlays/dwc2.dtbo
```

Overlays that Yocto did not build keep their original files. 

### **Y11. Copy kernel modules** 

Modules must come from the **same** kernel build as `Image` , or they will fail to load. 

```
rm -rf /tmp/yocto-modules && mkdir /tmp/yocto-modules
tar -xzf $DEPLOY/modules-raspberrypi5.tgz -C /tmp/yocto-modules
for ko in $KDIR/modules/*.ko; do
    name=$(basename $ko)
    src=$(find /tmp/yocto-modules -name "$name" | head -1)
    if [ -n "$src" ]; then cp "$src" "$ko"
    else echo "MISSING in Yocto build: $name"; fi
done
```

If any module is reported missing, enable it as `=m` (or built-in `=y` ) in your kernel config fragment and rebuild (Step Y6). 

### **Y12. Verify versions match** 

```
strings $KDIR/Image | grep -m1 "Linux version"
modinfo -F vermagic $KDIR/modules/$(ls $KDIR/modules | head -1)
```

Both should show the same kernel version ( **6.12.x** ). 

15 

### **Y13. Rebuild the Android images with the Yocto BSP** 

```
cd $AOSP
source build/envsetup.sh
lunch aosp_rpi5_car-bp2a-userdebug
make bootimage systemimage vendorimage -j$(nproc)
# mkimg refuses to overwrite an image with the same date — remove it first
rm -f out/target/product/rpi5/RaspberryVanillaAOSP16-$(date +%Y%m%d)-rpi5_car.img
./rpi5-mkimg.sh
```

Flash the new `RaspberryVanillaAOSP16-<YYYYMMDD>-rpi5_car.img` exactly as described in 

#### **Prepare SD Card** . 

### **Y14. Alternative: replace files in an already-flashed image or SD card** 

Without rebuilding Android, you can replace the kernel files directly in the boot partition (partition 1, FAT32): 

```
# On an .img file
sudo kpartx -av out/target/product/rpi5/RaspberryVanillaAOSP16-<YYYYMMDD>-rpi5_car.img
sudo mkdir -p /mnt/rpiboot
sudo mount /dev/mapper/loop0p1 /mnt/rpiboot      # use the loop name kpartx printed
# (On an SD card instead: sudo mount /dev/sdX1 /mnt/rpiboot)
sudo cp $KDIR/Image /mnt/rpiboot/
sudo cp $KDIR/bcm2712*.dtb /mnt/rpiboot/
sudo cp $KDIR/overlays/*.dtbo /mnt/rpiboot/overlays/
sync
sudo umount /mnt/rpiboot
sudo kpartx -d out/target/product/rpi5/RaspberryVanillaAOSP16-<YYYYMMDD>-rpi5_car.img
```

Kernel modules are not stored in the boot partition, so module changes need the full rebuild in Step Y13. 

### **Y15. Check on the device** 

```
adb shell uname -a
adb shell cat /proc/version
adb shell lsmod
```

#### **Troubleshooting:** 

- **Rainbow screen / no boot:** `Image` or `bcm2712-rpi-5-b.dtb` is missing or wrong — re-check Step Y9. 

- **Boot loop at Android logo:** usually missing Android kernel options — run `bitbake -c kernel_configcheck -f virtual/kernel` and fix the reported options. **No display:** `vc4-kms-v3d` / `cma` overlays missing — re-check Step Y10. 

- **Roll back:** `cd $KDIR && git checkout .` , then rebuild boot image. 

16 

- **Reference kernel:** the prebuilt Android kernel is built from the Android Common Kernel branch `common-android16-6.12-lts` with the raspberry-vanilla kernel manifest ( `android-16.0` ) — compare its `.config` with yours if something does not work. 

## **Prepare SD Card** 

Once the AOSP system images are built, there are two easy-to-use software options to flash them onto an SD card: Raspberry Pi Imager or Balena Etcher. Both are available for Windows and Linux. 

For this guide, I will use Raspberry Pi Imager for specific reasons mentioned in the “Issues” section. However, both tools are excellent and straightforward to use. 

#### **Steps to Flash the SD Card:** 

1. Open Raspberry Pi Imager. 

2. Select the Raspberry Pi device: **Raspberry Pi 5** . 

3. Under Operating System, choose **Custom** and select `RaspberryVanillaAOSP16– 20250907-rpi5_car.img` . 

4. Select the SD card under **Storage** . 

Click “Yes” without changing any settings. If you need to modify any options, feel free to do so. 

The tool will first write the image and then verify it; both steps may take some time, so it’s a good opportunity to grab a coffee. 

17 



<!-- Start of picture text -->
RaspberryPi Imager v1.8.5 - og xX<br>S Raspberry Pi<br>Raspberry Pi Device Operating System Storage<br>Writing... 0%<br>CANCEL WRITE<br><!-- End of picture text -->



<!-- Start of picture text -->
Raspberry Pi Imager v1.8.5 5 © &<br>8 Raspberry Pi<br>Raspberry Pi Device Operating System Storage<br>Verifying... 7%<br><!-- End of picture text -->

18 



<!-- Start of picture text -->
Raspberry Pi Imager v1.9.3 - ag x<br>Write Successful x<br>RaspberryVanillaAOSP16-20250907-rpi5_car.img has been written to<br>Generic STORAGE DEVICE<br>You can now remove the SD card from the reader<br>CONTINUE<br><!-- End of picture text -->

The `RaspberryVanillaAOSP16–20250907-rpi5_car.img` The SD card is now ready to flash onto the Raspberry Pi 5 board. 

After successfully flashing the SD card with — `RaspberryVanillaAOSP16–20250907rpi5_car.img` Your SD card partitions will appear as shown below. 



<!-- Start of picture text -->
.<br>/dev/sda - GParted - o x<br>GParted Edit View Device Partition Help<br>30 e iv) @ /dev/sda (59.48GiB) ~<br>/dev/sda4<br>11.43 GiB<br>/dev/sda1 & Fat32_/media/nar... boot 128.00MiB 39.19MiB_—88.81 MiB boot, lba<br>/dev/sda2 A ext4 /media/nar... / 2.50 GiB 1.61GiB 909.59 MiB<br>/dev/sda3 ext4 /media/nar... vendor  256.00MiB 176.94MiB 79.06 MiB<br>/dev/sda4 A ext4 /media/nar... userdata 11.43GiB 470.70 MiB 10.97 GiB<br>unallocated unallocated 45.17 GiB - 5<br>O operations pending<br><!-- End of picture text -->

19 

Here is the complete article to prepare the SD card! 

##### **Preparing SD card to Flash Raspberry PI-5 with Android-15** 

This article explains all the steps for SD Card recovery and preparing it for flashing. These steps are common and not… 

## **Start Raspberry Pi 5 Board with AOSP 16 flashed to SD card** 

Insert the SD card into the Raspberry Pi 5 board and power it on. The board will boot and flash Android 16. If the power LED blinks, it indicates the Pi is starting up. After a few minutes, the Android logo will appear on the screen. 

Finally, I can see Android-16 Automotive OS on Display 



20 



Android Automotive + Rpi 5 B 

## **Issue and Key Points Captured** 

The installation of tools required for the build environment can take a significant amount of time. 

Installing `openjdk-17-headless` alone took nearly an hour. 

```
sudo apt-get install git-core gnupg flex bison gperf build-essential \
    zip curl zlib1g-dev gcc-multilib g++-multilib libc6-dev-i386 lib32ncurses5-dev \
    x11proto-core-dev libx11-dev lib32z-dev ccache libgl1-mesa-dev libxml2-utils \
    xsltproc unzip fontconfig
```

### **Open-JDK 17 installation error:** 

```
33% [11 openjdk-17-jdk-headless 25.8 MB/71.3 MB 36%]
Get:12 http://in.archive.ubuntu.com/ubuntu jammy-updates/universe amd64 openjdk-…
Fetched 72.6 MB in 2h 29min 26s (8,094 B/s)
```

```
E: Failed to fetch http://security.ubuntu.com/ubuntu/pool/universe/o/openjdk-17/…
```

```
E: Unable to fetch some archives, maybe run apt-get update or try with --fix-missing?
```

#### **Solution:** 

```
E: Unable to fetch some archives, maybe run apt-get update or try with --fix-missing?
```

### **Build Issue:** 

#### **lunch command:** 

21 

```
lunch
No target specified. See lunch --help
```

**lunch --help:** 

```
The lunch command selects the configuration to use for subsequent
Android builds.
```

```
Usage: lunch TARGET_PRODUCT [TARGET_RELEASE [TARGET_BUILD_VARIANT]]
   Choose the product, release and variant to use. If not
   supplied, TARGET_RELEASE will be 'trunk_staging' and
   TARGET_BUILD_VARIANT will be 'eng'
Usage: lunch TARGET_PRODUCT-TARGET_RELEASE-TARGET_BUILD_VARIANT
   Chose the product, release and variant to use. This
   legacy format is maintained for compatibility.
Note that the previous interactive menu and list of hard-coded
list of curated targets has been removed. If you would like the
list of products, release configs for a particular product, or
variants, run list_products list_releases or list_variants
respectively.
```

If you execute lunch and want to see menu options that probably will not show, in that case, you need to have a look at “Android16/device/generic/car/” in AndroidProducts.mk configurations. 

```
COMMON_LUNCH_CHOICES := \
    gsi_car_arm64-trunk_staging-userdebug \
    gsi_car_x86_64-trunk_staging-userdebug \
    sdk_car_arm64-trunk_staging-userdebug \
    sdk_car_md_x86_64-trunk_staging-userdebug \
    sdk_car_cw_x86_64-trunk_staging-userdebug \
    sdk_car_x86_64-trunk_staging-userdebug \
```

The above options are for a car emulator and other variants. To find the correct lunch option, look at → 

```
cat ./brcm/rpi5/AndroidProducts.mk
```

```
PRODUCT_MAKEFILES := \
    $(LOCAL_DIR)/aosp_rpi5.mk \
    $(LOCAL_DIR)/aosp_rpi5_car.mk \
    $(LOCAL_DIR)/aosp_rpi5_tv.mk
COMMON_LUNCH_CHOICES := \
    aosp_rpi5-trunk_staging-userdebug \
    ---> aosp_rpi5_car-trunk_staging-userdebug \
    aosp_rpi5_tv-trunk_staging-userdebug
```

When you run → 

22 

```
lunch aosp_rpi5_car-trunk_staging-userdebug
```

#### You may see → 

```
panic: Missing config trunk_staging.    Trace=[trunk_staging]
goroutine 1 [running]:
main.main()
    build/soong/cmd/release_config/release_config/main.go:71 +0x14c5
In file included from build/make/core/config.mk:419:
In file included from build/make/core/envsetup.mk:51:
build/make/core/release_config.mk:142: error: release-config failed to run.
10:04:49 dumpvars failed with: exit status 1
```

#### **Solution:** 

```
lunch aosp_rpi5_car-bp2a-userdebug
```

### **Flashing Issue:** 

Sometimes, the Raspberry Pi board may fail to boot or display output. In such cases, ensure that the SD card was flashed correctly. If needed, reformat the SD card with the FAT32 file system and flash the image again. 

I have noted a few issues encountered while flashing the Raspberry Pi 5 with Android 15. Please refer to the article if you face similar problems. 

##### **Build AOSP Android 15 Automotive OS for Raspberry Pi 5 and flash it.** 

This article will cover building Android-15 Automotive OS, building Raspberry PI5 Image, preparing setup for aosp… 

_Thank you for reading!_ 

_Questions and suggestions are welcome._ 

_Thank you!_ 

**Prepared by DHINESH E** · <u>dhinesh701@gmail.com</u> 

23 

