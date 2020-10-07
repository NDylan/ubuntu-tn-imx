Technexion Ubuntu 20.04 LTS Image Builder
===========================

## Support Hardware
--------
|System-On-Module|Baseboard|
|---|---|
|PICO-IMX8M-MINI|PI<br>WIZARD (WIP)|
|EDM-G-IMX8M-PLUS|EVK(WIP)|

****
## Contents
* [Overview](#Overview)
* [Build-Image (for developer)](#Build-Image)
    * Download the source code
    * Compiling Environment Setup
    * Build a runtime Image
    * Flash the image to the target board
* [Quick-Start (for user)](#Quick-Start)
    * Run the apps using debug console/ssh
    * Playback video
    * `glmark` for GPU testing
    * Run QT5 applications
    * Control WiFi connection using `nmcli`
    * Docker conatiner
    * Expand rootfs partition
    * Weston Keyboard shortcuts
    * Chromium

* [Apps-Developing](#Apps-Developing)
* [Known-Limitations](#Known-issues)

****
### Overview
-----------
Technexion Ubuntu rootfs(Root Filesystem) was be generated by bash scripts, that help the
users to produce a customized ubuntu image using Cananical unique tool named `debootstrap`,
then adapt QEMU to config custom packages and tools from host PC.

**Default username:password = ubuntu:ubuntu**

**Superuser username:password = root:root**

We recommended adapt ubuntu user as normal use, and adapt root user to do deveoping and debuging because Weston is running on root permission, it will not worked if you call GUI relate commands using "ubuntu user".

Features:
* LTS version until 2025/04
* Weston Desktop with GPU accelelation v6.0
* Wayland compositor v1.18.0
* gstreamer1.0 v1.16.2
* VPU libraries v1.18
* QT5 with GPU acceleration  v5.12.8
* apt-get package manager
* openGL with GPU accelelation
* docker container v19.03.8
* Swap parition implementation using zram

****
### Build-Image
-----------

#### Download the source code

Github way (Prepare git command first is recommended)

Install git first:

    $ sudo apt-get install repo

Download source code:

    $ git clone https://github.com/TechNexion-customization/ubuntu-tn-imx8.git

#### Compiling environment setup

General Packages Installation (Ubuntu 18.04 or above)

    Install necessary packages
    $ sudo apt-get install gawk wget git git-core diffstat unzip texinfo gcc-multilib build-essential \
    chrpath socat cpio python python3 python3-pip python3-pexpect \
    xz-utils debianutils iputils-ping libsdl1.2-dev xterm \
    language-pack-en coreutils texi2html file docbook-utils \
    python-pysqlite2 help2man desktop-file-utils \
    libgl1-mesa-dev libglu1-mesa-dev mercurial autoconf automake \
    groff curl lzop asciidoc u-boot-tools libreoffice-writer \
    sshpass ssh-askpass zip xz-utils kpartx vim screen bison flex \
    debootstrap qemu-system-arm qemu-user-static libssl-dev

    Install cross-compiler
    $ sudo apt-get install gcc-aarch64-linux-gnu 


****
#### Build a runtime Image


Lazy way, make a runtime image directly:
Note that this BSP does support PICO-IMX8MM with PI only at this moment.

    $ make all
    (it will download the latest u-boot, kernel and rootfs and compile them automatically)

You also can compile the source code separatically:

    $ make u-boot (download and compile the latest u-boot only)
    $ make kernel (download and compile the latest kernel only)
    $ make rootfs (download and compile the latest rootfs only)
    $ make image (package runtime image only, depend on above three items done)
    
Remove all compiled objects and image:

    $ make clean
   
#### Flash the image to the target board

Output relative image files of path:

    $ ls output
    kernel  rootfs.tgz  u-boot  ubuntu.img

We provide two ways for image flashing:

**1. `uuu` way**

It's a modular flash tool base on serial download mode, download the tools from technexion website as following link:

Download: [prebuilt binary](ftp://ftp.technexion.net/development_resources/development_tools/installer/imx-mfg-uuu-tool_20200327.zip)

Step 1. Connect a Type-C cable between host PC and the board

Step 2. Change the boot mode from eMMC mode to serial download mode

Step 3. Issue uuu command to flash the compiled Ubuntu image to eMMC:

    $ sudo uuu/linux64/uuu -b emmc_img imx8mm/pico-imx8mm-flash.bin ubuntu.img

Step 4. Change back boot mode from serial download mode to eMMC mode, it should be works!

For Windows OS or want to know the detail users , please click this [link](https://github.com/TechNexion/u-boot-edm/wiki/Use-mfgtool-%22uuu%22-to-flash-eMMC)

**2. `ums` way**

It's a easy way base on u-boot prompt of eMMC boot mode, but the disadvantage is the speed is a little bit lower than `uuu` way, another limitation is it must be existed a works u-boot inside the eMMC, if not, you still need adapt `uuu` way at first.

Step 1. Keep eMMC boot mode

Step 2. Connect a Type-C cable between host PC and the board for image flashing

Step 3. Connect a micro-usb cable between host PC and the board for debeg cosole

Step 4. press enter key on debug console when boot up u-boot, let u-boot into prompt

Step 5. Target board side: Issue ums command to mount eMMC as a mass storage:

    # ums 0 mmc 2

Step 6. Host PC side: Adapt basic `dd` command is enough for image flashing:

    $ sudo dd if=ubuntu.img of=/dev/sdx bs=1M (sdx means the device node which ums mounted storage)
    $ sync

**QCA9377:** If you want to enable  QCA9377 WiFi/Bluetooth functions, please copy relate firmware files to specific path of rootfs partition as following:

    WiFi:
    $ sudo cp -a <firmware path>/qca9377/bdwlan30.bin mnt/lib/firmware/qca9377/
    $ sudo cp -a <firmware path>/qca9377/otp30.bin mnt/lib/firmware/qca9377/
    $ sudo cp -a <firmware path>/qca9377/qwlan30.bin mnt/lib/firmware/qca9377/
    $ sudo cp -a <firmware path>/qca9377/utf30.bin mnt/lib/firmware/qca9377/
    $ sudo cp -a <firmware path>/wlan/cfg.dat mnt/lib/firmware/wlan/cfg.dat
    $ sudo cp -a <firmware path>/wlan/qca9377/qcom_cfg.ini mnt/lib/firmware/wlan/qca9377/qcom_cfg.ini
    
    Bluetooth:
    $ sudo cp -a <firmware path>/qca/nvm_tlv_3.2.bin mnt/lib/firmware/qca/nvm_tlv_3.2.bin
    $ sudo cp -a <firmware path>/qca/rampatch_tlv_3.2.tlv mnt/lib/firmware/qca/rampatch_tlv_3.2.tlv
    
    Please contact sales@technexion.com to get firmware files.

****
### Quick-Start
-----------

#### Run the apps using debug console/ssh

Due to Wayland protocol need root permission, so the users need export wayland necessary variables with commands if adapt **ubuntu user**, for examples:

    $ sudo -E glmark-es2-wayland
    $ sudo -E chromium --no-sandbox --test-type
    (Note that 'sudo -E' is necessary when run any graphic base apps)

If the users login using **root user**, just issue commands without 'sudo -E' directly when run graphic base apps.

#### Playback video

Adapt gstreamer-1.0 which supports avi, mp4, mkv and webm format files, please change to root user and issue quick command to play video:

    # gplay-1.0 /home/ubuntu/mnt/test.mp4
    
    You should be see a resolution problem because ILI9881C is a portrait screen, please issue standard command to playback on fullscreen mode:
    
    # gst-launch-1.0 playbin uri=file:///home/ubuntu/mnt/test.mp4 video-sink="glimagesink render-rectangle=<1,1,1280,720>"
    
    Of course if your panel is base on landscape screen such as HDMI, you can use gplay directly.


#### `glmark` for GPU benchmark testing

    # glmark2-es2-wayland


#### Run QT5 applications

We support libQT5 relate libraries, you can develop your apps using QT-Creator, and running on Ubuntu directly, or copy executable QT apps from Technexion Yocto 3.0, it also works.

    Example:
    # root@technexion:/home/ubuntu/QtDemo# ./QtDemo 
    Using Wayland-EGL
    Using the 'xdg-shell' shell integration


#### Control WiFi connection using `nmcli`

Ubuntu adapt network-manager service to manage network status, so please use `nmcli` to change the configuration if you need:

    Example: WiFi Station mode 
    1. Scan exist WiFi hotspots
    $ sudo nmcli device wifi list
    2. Make a connection
    $ sudo nmcli device wifi connect SSID-Name password wireless-password
    
    Example: WiFi AP mode
    $ sudo nmcli con add type wifi ifname wlan0 con-name Hostspot autoconnect yes ssid Hostspot
    $ sudo nmcli con modify Hostspot 802-11-wireless.mode ap 802-11-wireless.band a ipv4.method shared
    $ sudo nmcli con modify Hostspot wifi-sec.key-mgmt wpa-psk
    $ sudo nmcli con modify Hostspot wifi-sec.psk "veryveryhardpassword1234"
    $ sudo nmcli con up Hostspot
    

#### Docker conatiner

For OS virtualization requirement, we enable docker engine in Ubuntu, the users can use docker commands to pull exist containers from dockerhub, of course it can build in Ubuntu using dockerfile, note that you have to choose aarch64 strcuture base container.


#### Expand rootfs partition

After flash the ubuntu image to eMMC, you'll see a pop-up window at first boot as following figure:

![expand_rootfs](figures/expand_rootfs.png)

The system will starting expand rootfs partition and wait for about 10 seconds, it will auto reboot and you can start enjoy your Ubuntu. 


#### Weston Keyboard shortcuts

|#|shortcut set|function|
|---|---|----
|1|super + s|make a screenshot of the desktop
|2|super + r|record start/stop a video of the desktop
|3|super + Tab|swich active windows


#### Chromium

|#|method|command|
|---|---|----
|1| click icon| no need, icon is exist on left top side of weston desktop
|2| basic browser| # chromium --no-sandbox --test-type
|3| kiosk mode (full screen)| # chromium --no-sandbox --test-type --start-fullscreen www.technexion.com



****
### Apps-Developing
-----------

**Toolchain**

**1. Non-GUI/Web GUI applications**

Example: IoT, Industrial or Machine Learning

|#|Language|libraries|
|---|---|----
|1|JAVA|apt-get install openjdk11-jdk
|2|Python|apt-get install python3-pip
|2|JavaScript|apt-get install nodejs npm
|2|Golang|apt-get install golang
|2|C/C++|apt-get install gcc

The users also can compile the program on host PC side using cross-compiler if adapt non-OO programming language.

**2. GUI applications with HW acceleration**

Example: QT5, Wayland

|#|Language|libraries|
|---|---|----
|1|C++|QT5
|2|C|Wayland

Wayland example:

    $ git clone https://gitlab.com/hdante/hello_wayland.git
    $ sudo apt-get install make libwayland-dev
    $ cd hello_wayland
    $ make
    $ ./hello_wayland

QT example (coming soon)

We recommended developing GUI applications on host PC side, it's saving eMMC usage for develop libraries especially huge HMI system, so Technexion will provide a Graphic SDK for host PC side use soon.


****
### Known-Limitations
-----------

1. Our Ubunut does support HW acceleration on Wayland, it means our weston, Wayland, QT5 and gstreamer-1.0 relate libraries all tweaked already, so please don't remove them and re-install same package via apt-get, it will install no HW acceleration library without tweaked from Ubuntu package management server.

2. This Ubuntu is base on Wayland graphic protocol, so Xorg base app/librareis will be execute invalided, don't spend time to install Xorg relate programs.
