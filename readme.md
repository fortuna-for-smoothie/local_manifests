Since hybris 16 you'll need to clone libhybris manually before building hybris-hal:
```sh
cd $ANDROID_ROOT/external
git clone --recurse-submodules https://github.com/mer-hybris/libhybris.git
cd $ANDROID_ROOT
hybris-patches/apply-patches.sh --mb
```

# Notes

## defconfig location

Target defconfig defined in here
```sh
code device/samsung/msm8916-common/BoardConfigCommon.mk
```
Target defconfig itself is this
```sh
code kernel/samsung/msm8916/arch/arm/configs/msm8916_sec_defconfig
```

## Troubleshoot building userspace packages

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

Thansk to [question](https://piggz.co.uk/sailfishos-porters-archive/index.php?log=2024-12-14.txt#line26)
and [answer](https://piggz.co.uk/sailfishos-porters-archive/index.php?log=2024-12-04.txt#line22)
