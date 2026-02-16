## USB fingerprint sensor driver

Supported device: FPC Sensor Controller L:0002 FW:022.26.2.x (10a5:a900)

## Environment

Ubuntu: `apt install -y --no-install-recommends libfprint2 fprintd pam_fprintd libusb-1.0-0-dev libevent-dev libdbus-1-dev libssl-dev libopencv-dev make cmake pkg-config gcc g++`


## Build

```bash
git clone https://github.com/elemantreum/fingerprint-ocv
cd fingerprint-ocv
git submodule init
git submodule update
cmake -S . -B build
cmake --build build
cp build/src/fingerprint-ocv /path/to/somewhere
```

## Install && Binary

[Link](./INSTALL.md)

## Warning!

Instruction don't be tested on any devices. It should work on Ubuntu, but i think i missed something, when was writing it. When i test this instruction and it will be working, this warning will be deleted. 
