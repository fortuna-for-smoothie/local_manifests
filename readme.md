# Sailfish OS 5.1.0.11

Notes on building and porting Sailfish OS for fortuna3g on hybris-18.1

### Before making hybris-hal and droidmedia

Since hybris 16 you'll need to clone libhybris manually before building hybris-hal:

```sh
cd $ANDROID_ROOT/external
git clone --recurse-submodules https://github.com/mer-hybris/libhybris.git
cd $ANDROID_ROOT
hybris-patches/apply-patches.sh --mb
```

### defconfig location

Target defconfig defined in here
```sh
code device/samsung/msm8916-common/BoardConfigCommon.mk
```
Target defconfig itself is this
```sh
code kernel/samsung/msm8916/arch/arm/configs/msm8916_sec_defconfig
```

### Packaging droid-hal expects python

During `rpm/dhd/helpers/build_packages.sh --droid-hal` you will likely encounter the following:

```log
...
++ awk '/PLATFORM_VERSION([A-Z0-9.]*|_LAST_STABLE) := ([0-9.]+)/ { print $3; }'
++ awk -F. '{print $1}'
+ android_version_major=11
++ readlink -e .
+ ANDROID_ROOT=/home/smooth-e/hadk
+ echo Building local tools
Building local tools
+ '[' 11 -ge 8 ']'
+ /home/smooth-e/hadk/build/make/tools/fs_config/fs_config_generator.py aidarray /home/smooth-e/hadk/system/core/libcutils/include/private/android_filesystem_config.h
/usr/bin/env: ‘python’: No such file or directory
...
```

Install python2 into your target to fix this:

```sh
sb2 -t $VENDOR-$DEVICE-$PORT_ARCH -m sdk-install -R zypper in python
```

Thanks to [question](https://piggz.co.uk/sailfishos-porters-archive/index.php?log=2024-12-14.txt#line26)
and [answer](https://piggz.co.uk/sailfishos-porters-archive/index.php?log=2024-12-04.txt#line22)

### Packaging droid-hal fails

Build with `make hybris-hal` in hadk on top of hybris-18.1 currently does not put anything into here:

```sh
$ANDROID_ROOT/out/target/product/fortuna3g/root/
```

but the build script expects at least one .rc file in this directory. 
Currently the workaround is to create an empty dummy .rc file and call it a day. 
Also we use a fork of droid-hal-hal-device with patches to the build script commenting out other related dirs and files. 
Execute this for a fix:

```sh
# Create an empty rc file, if not present already
touch $ANDROID_ROOT/out/target/product/fortuna3g/root/dummy.rc
# Package droid-hal
rpm/dhd/helpers/build_packages.sh --droid-hal
```

### Getting telnet under WSL

In case you are using WSL, this how you get telnet set up and connect to your device. Thanks to [this SailfishOS Forum answer](https://forum.sailfishos.org/t/jolla-phone-2026-windows-11-has-anyone-successfully-used-usb-developer-mode-rndis/30616/6).

First, get latest usbipd-win from GitHub and install it: https://github.com/dorssel/usbipd-win

Second, you will need to compile and enable a module for usb networking to work under WSL. Execute these:

```sh
cd $HOME

# Clone and build the module, tested on official Ubutnu 20.04 for WSL
sudo apt update
sudo apt install -y build-essential flex bison libssl-dev libelf-dev bc dwarves git usbutils
git clone --depth 1 --branch linux-msft-wsl-$(uname -r | cut -d- -f1) \
    https://github.com/microsoft/WSL2-Linux-Kernel.git
cd WSL2-Linux-Kernel
zcat /proc/config.gz > .config
scripts/config --module USB_NET_RNDIS_HOST
make olddefconfig
make modules_prepare
make M=drivers/net/usb modules KBUILD_MODPOST_WARN=1

# Enable the kernel module
sudo modprobe usbnet
sudo rmmod rndis_host cdc_ncm cdc_ether 2>/dev/null
sudo insmod ~/WSL2-Linux-Kernel/drivers/net/usb/cdc_ether.ko
sudo insmod ~/WSL2-Linux-Kernel/drivers/net/usb/rndis_host.ko
```

When your device is connected, attach it to WSL. You can get the bus id with `usbipd list`. You may need to bind the connection before attaching. usbipd requires administrator access.

```bat
usbipd attach --wsl --busid 2-2
```

When attached, and if telnet is advertised by the device, `ip ad` will display and interface, for example `usb0`.
We need to enable it and add an ip:

```sh
sudo ip link set usb0 up
sudo ip addr add 192.168.2.14/24 dev usb0
```

Next you can use telnet as per the HADK handbook. For example:

```sh
telnet 192.168.2.15 23
```
