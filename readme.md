Since hybris 16 you'll need to clone libhybris manually before building hybris-hal:
```sh
cd $ANDROID_ROOT/external
git clone --recurse-submodules https://github.com/mer-hybris/libhybris.git
cd $ANDROID_ROOT
hybris-patches/apply-patches.sh --mb
```
