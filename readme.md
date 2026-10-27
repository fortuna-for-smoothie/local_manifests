```sh
mkdir -p android/lineage-18.1
cd android/lineage-18.1/

# Install repo
mkdir -p ~/.bin
PATH="${HOME}/.bin:${PATH}" # or add to ~/.bashrc
curl https://storage.googleapis.com/git-repo-downloads/repo > ~/.bin/repo
chmod a+rx ~/.bin/repo

git config --global user.name username
git config --global user.email user@example.com

repo init -u https://github.com/LineageOS/android.git -b lineage-18.1 --git-lfs
git clone https://github.com/fortuna-for-smoothie/local_manifests .repo/local_manifests
repo sync -c -j4 --fail-fast # --recurse-submodules fails

sudo apt install python-is-python3
sudo apt install bc bison build-essential ccache curl erofs-utils flex g++-multilib gcc-multilib git git-lfs gnupg gperf imagemagick kmod protobuf-compiler python3-protobuf lib32readline-dev lib32z1-dev libdw-dev libelf-dev libgnutls28-dev lz4 libsdl1.2-dev libssl-dev libxml2 libxml2-utils lzop pngcrush rsync schedtool squashfs-tools xsltproc xxd zip zlib1g-dev
sudo apt install lib32ncurses5-dev libncurses5 libncurses5-dev

# Start building
source build/envsetup.sh
lunch lineage_fortuna3g-userdebug

# Test if basic components are present
# You may need to create an empty Android.bp file
m -j4 nothing 

# Start building an actual image
m -j4 bacon
```
