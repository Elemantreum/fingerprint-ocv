# Fingerprint Driver Installation Guide

## Supported Devices

- FPC Sensor Controller (USB ID: 10a5:a900)
- Honor Magicbook Art 14 and compatible laptops

## Requirements

- Ubuntu
- CMake, GCC/G++
- libusb-1.0, libevent, dbus, openssl, opencv, fprintd

## Installation Steps

### 1. Install Dependencies

```bash
sudo apt install fprintd libusbx-devel libevent-devel dbus-devel openssl-devel opencv-devel cmake gcc gcc-c++
```

### 2. Clone and Build

```bash
git clone https://github.com/elemantreum/fingerprint-ocv
cd fingerprint-ocv
git submodule init
git submodule update
cmake -S . -B build
cmake --build build
sudo cp build/src/fingerprint-ocv /usr/libexec/fprintd
```


### 3. Set Up systemd Service

```bash
sudo cat > /etc/systemd/system/fprintd.service << 'EOF'
[Unit]
Description=Fingerprint Authentication Daemon (OCV)
After=systemd-logind.service
Requires=systemd-logind.service

[Service]
Type=dbus
BusName=net.reactivated.Fprint
ExecStart=/usr/libexec/fprintd-ocv --bus=system
Restart=on-failure

[Install]
WantedBy=multi-user.target
~                              
EOF

sudo systemctl daemon-reload
sudo systemctl restart fprintd.service
```

### 4. Verify Installation

```bash
systemctl status fprintd.service
fprintd-verify
```

## Commands Reference

```bash
# Check device
lsusb | grep a900

# Verify driver is running
systemctl status fprintd.service

# Enroll fingerprint
fprintd-enroll

# Verify fingerprint
fprintd-verify

# Restart driver
sudo systemctl restart fprintd

# Stop driver
sudo systemctl stop fprintd
```
