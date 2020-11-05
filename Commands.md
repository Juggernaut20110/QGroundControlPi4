https://www.interelectronix.com/qt-on-the-raspberry-pi-4.html
https://discuss.ardupilot.org/t/how-to-cross-compile-qgroundcontrol-for-raspberry-pi3/26790
https://wapel.de/?cat=13
https://www.tal.org/tutorials/building-qt-512-raspberry-pi

#On RPi
sudo nano /etc/apt/sources.list
sudo apt-get update
sudo apt-get dist-upgrade
sudo reboot
sudo rpi-update
sudo reboot
sudo apt-get -y build-dep qt5-qmake
sudo apt-get -y build-dep libqt5texttospeech5-dev
sudo apt-get -y build-dep libqt5multimedia5-plugins
sudo apt-get -y build-dep libqt5waylandclient5-dev
sudo apt-get -y install libudev-dev libinput-dev libts-dev libxcb-xinerama0-dev libxcb-xinerama0 gdbserver bluez libbluetooth-dev build-essential libfontconfig1-dev libdbus-1-dev libfreetype6-dev libicu-dev libinput-dev libxkbcommon-dev libsqlite3-dev libssl-dev libpng-dev libjpeg-dev libglib2.0-dev libraspberrypi-dev
sudo apt-get -y install libegl1-mesa libegl1-mesa-dev libgles2-mesa libgles2-mesa-dev libgbm-dev mesa-common-dev
sudo apt-get -y install libgstreamer1.0-dev libgstreamer-plugins-base1.0-dev gstreamer1.0-plugins-base gstreamer1.0-plugins-good gstreamer1.0-plugins-ugly gstreamer1.0-plugins-bad libgstreamer-plugins-bad1.0-dev gstreamer1.0-pulseaudio gstreamer1.0-tools gstreamer1.0-alsa
sudo apt-get -y install speech-dispatcher libudev-dev libsdl2-dev
sudo visudo
#pi ALL=NOPASSWD:/usr/bin/rsync

sudo apt-get -y build-dep libqt5webengine-data
sudo apt-get -y build-dep libqt5webkit5

#On VM
sudo apt-get update
sudo apt-get upgrade
sudo apt install build-essential dkms linux-headers-$(uname -r)
#install guest additions
sudo apt-get install gcc git bison python gperf pkg-config gdb-multiarch
ssh-keygen -t rsa -C pi@192.168.1.206 -P "" -f ~/.ssh/rpi_pi_id_rsa
cat ~/.ssh/rpi_pi_id_rsa.pub | ssh -o StrictHostKeyChecking=no pi@192.168.1.206 "mkdir -p .ssh && chmod 700 .ssh && cat >> .ssh/authorized_keys"

sudo mkdir /opt/qt5
sudo chown user:user /opt/qt5
cd /opt/qt5
wget http://download.qt.io/archive/qt/5.12/5.12.6/single/qt-everywhere-src-5.12.6.tar.xz
wget https://raw.githubusercontent.com/riscv/riscv-poky/master/scripts/sysroot-relativelinks.py
sudo chmod +x sysroot-relativelinks.py

tar xf qt-everywhere-src-5.12.6.tar.xz
cp -R qt-everywhere-src-5.12.6/qtbase/mkspecs/linux-arm-gnueabi-g++ qt-everywhere-src-5.12.6/qtbase/mkspecs/linux-arm-gnueabihf-g++
sed -i -e 's/arm-linux-gnueabi-/arm-linux-gnueabihf-/g' qt-everywhere-src-5.12.6/qtbase/mkspecs/linux-arm-gnueabihf-g++/qmake.conf

mkdir qt-everywhere-src-5.12.6/qtbase/mkspecs/devices/linux-rasp-pi4-v3d-g++
cd qt-everywhere-src-5.12.6/qtbase/mkspecs/devices/linux-rasp-pi4-v3d-g++
wget https://raw.githubusercontent.com/qt/qtbase/5.14.0/mkspecs/devices/linux-rasp-pi4-v3d-g%2B%2B/qmake.conf
wget https://raw.githubusercontent.com/qt/qtbase/5.14.0/mkspecs/devices/linux-rasp-pi4-v3d-g%2B%2B/qplatformdefs.h

cd /opt/qt5
wget https://releases.linaro.org/components/toolchain/binaries/latest-7/arm-linux-gnueabihf/gcc-linaro-7.5.0-2019.12-x86_64_arm-linux-gnueabihf.tar.xz
tar xf gcc-linaro-7.5.0-2019.12-x86_64_arm-linux-gnueabihf.tar.xz

rsync -avz --rsync-path="sudo rsync" --delete pi@192.168.1.206:/lib sysroot
rsync -avz --rsync-path="sudo rsync" --delete pi@192.168.1.206:/usr/include sysroot/usr
rsync -avz --rsync-path="sudo rsync" --delete pi@192.168.1.206:/usr/lib sysroot/usr
rsync -avz --rsync-path="sudo rsync" --delete pi@192.168.1.206:/opt/vc sysroot/opt
./sysroot-relativelinks.py sysroot

mkdir /opt/qt5/sysroot/etc
nano /opt/qt5/sysroot/etc/ld.so.conf
#add the following
/opt/vc/lib
/lib/arm-linux-gnueabihf
/usr/lib/arm-linux-gnueabihf
/usr/local/lib

mkdir build
cd build
../qt-everywhere-src-5.12.6/configure -release -opengl es2  -eglfs -device linux-rasp-pi4-v3d-g++ -device-option CROSS_COMPILE=/opt/RaspberryQt/gcc-linaro-7.5.0-2019.12-x86_64_arm-linux-gnueabihf/bin/arm-linux-gnueabihf- -sysroot /opt/qt5/sysroot -prefix /usr/local/qt5 -opensource -confirm-license -skip qtscript -skip qtwebengine -nomake tests -nomake examples -make libs -pkg-config -no-use-gold-linker -v -recheck
../qt5/configure -release -opengl es2  -eglfs -device linux-rasp-pi4-v3d-g++ -device-option CROSS_COMPILE=/opt/RaspberryQt/gcc-linaro-7.5.0-2019.12-x86_64_arm-linux-gnueabihf/bin/arm-linux-gnueabihf- -sysroot /opt/qt5/sysroot -prefix /usr/local/qt5 -opensource -confirm-license -skip qtscript -skip qtwebengine -nomake tests -nomake examples -make libs -pkg-config -no-use-gold-linker -v -recheck
make -j8
make install -j8

cd ../qt-everywhere-src-5.12.6/qtspeech/
../../sysroot/usr/local/qt5/bin/qmake
make


ln -s /opt/qt5/sysroot/usr/lib/arm-linux-gnueabihf/libicui18n.so /opt/qt5/sysroot/usr/local/qt5/lib/libicui18n.so
ln -s /opt/qt5/sysroot/usr/lib/arm-linux-gnueabihf/libicui18n.so.63 /opt/qt5/sysroot/usr/local/qt5/lib/libicui18n.so.63
ln -s /opt/qt5/sysroot/usr/lib/arm-linux-gnueabihf/libicui18n.so.63.1 /opt/qt5/sysroot/usr/local/qt5/lib/libicui18n.so.63.1

cd /opt/qt5
rsync -avz --rsync-path="sudo rsync" sysroot/usr/local/qt5 pi@192.168.1.206:/usr/local
!!Done till here!!
cd ~
rsync -avz ~/build-qgroundcontrol-QT_for_Pi-Release pi@192.168.1.206:~/

echo /usr/local/qt5/lib | sudo tee /etc/ld.so.conf.d/RaspberryQt.conf
sudo ldconfig
