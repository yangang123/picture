stlink 安装在ubuntu中：
１．　sudo apt-get install libusb-dev
　　　　　sudo apt-get install libusb-1.0-0-dev
２．　
2.1：sudo add-apt-repository ppa:george-edison55/cmake-3.x
2.2：sudo apt-get update
2.3：sudo apt-get install cmake

3：　
git clone https://github.com/texane/stlink.git

４：　
cd stlink

make

cd build/Release && make install DESTDIR=_install

sudo cp st-flash /usr/bin

    lsusb查看usb是否存在

９．　
sudo st-flash write test.bin 0x8000000
