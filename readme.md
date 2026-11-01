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

### Failed build dependencies while packaging --version

While building with `--version` you will encounter an error like this:

```
error: Failed build dependencies:
        droid-config is needed by droid-hal-version-fortuna3g-0.0.1-202611011838.aarch64
        droid-config-preinit-plugins is needed by droid-hal-version-fortuna3g-0.0.1-202611011838.aarch64
        droid-config-pulseaudio-settings is needed by droid-hal-version-fortuna3g-0.0.1-202611011838.aarch64
        droid-config-sailfish is needed by droid-hal-version-fortuna3g-0.0.1-202611011838.aarch64
        droid-hal is needed by droid-hal-version-fortuna3g-0.0.1-202611011838.aarch64
        droid-hal-kernel is needed by droid-hal-version-fortuna3g-0.0.1-202611011838.aarch64
        hybris-libsensorfw-qt5 is needed by droid-hal-version-fortuna3g-0.0.1-202611011838.aarch64
        mce-plugin-libhybris is needed by droid-hal-version-fortuna3g-0.0.1-202611011838.aarch64
        ngfd-plugin-native-vibrator is needed by droid-hal-version-fortuna3g-0.0.1-202611011838.aarch64
        pulseaudio-modules-droid is needed by droid-hal-version-fortuna3g-0.0.1-202611011838.aarch64
        qt5-qpa-hwcomposer-plugin is needed by droid-hal-version-fortuna3g-0.0.1-202611011838.aarch64
        qtscenegraph-adaptation is needed by droid-hal-version-fortuna3g-0.0.1-202611011838.aarch64
```

This is due to both packages ofono-config-binder (a system package) and your droid-config-fortuna3g 
providing the file `/etc/ofono/binder.conf`. For a fix, after packaging --droid-hal, --configs, --mw and --gg 
execute the following:

```sh
sb2 -t $VENDOR-$DEVICE-$PORT_ARCH -R -m sdk-install \
    zypper in droid-local-repo/$DEVICE/droid-config-$DEVICE*.rpm -ofono-configs-binder
```

This will remove the ofono-config-binder package and install your device-specific packages. 
During the installation you will encounter this or similar a couple of times:

```
Problem: nothing provides 'pulseaudio-modules-droid >= 11.1.66' needed by the to be installed droid-config-fortuna3g-pulseaudio-settings-1-202611011807.aarch64
 Solution 1: do not install droid-config-fortuna3g-pulseaudio-settings-1-202611011807.aarch64
 Solution 2: break droid-config-fortuna3g-pulseaudio-settings-1-202611011807.aarch64 by ignoring some of its dependencies

Choose from above solutions by number or skip, retry or cancel [1/2/s/r/c/d/?] (c): 2
```

As shown, select option 2 to break and install, then select yes to install and remove ofono-configs-binder.

IRC logs where I got the solution and clues:

- https://piggz.co.uk/sailfishos-porters-archive/index.php?log=2023-10-29.txt#line75
- https://piggz.co.uk/sailfishos-porters-archive/index.php?log=2026-05-15.txt#line215 (at the bottom of the log)
