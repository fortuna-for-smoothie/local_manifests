Since hybris 16 you'll need to clone libhybris manually before building hybris-hal:
```sh
cd $ANDROID_ROOT/external
git clone --recurse-submodules https://github.com/mer-hybris/libhybris.git
cd $ANDROID_ROOT
hybris-patches/apply-patches.sh --mb
```

# Notes

Target defconfig defined in here
```sh
code device/samsung/msm8916-common/BoardConfigCommon.mk
```
Target defconfig itself is this
```sh
code kernel/samsung/msm8916/arch/arm/configs/msm8916_sec_defconfig
```
