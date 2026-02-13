# Fingerprint Driver Installation Guide

## Supported Devices

- FPC Sensor Controller (USB ID: 10a5:a900)
- Honor Magicbook Art 14 and compatible laptops

## Requirements

- Ubuntu
- CMake, GCC/G++
- libusb-1.0, libevent, dbus, openssl, opencv

## Installation Steps

### 1. Install Dependencies

```bash
sudo apt install libusbx-devel libevent-devel dbus-devel openssl-devel opencv-devel cmake gcc gcc-c++
```

### 2. Clone and Build

```bash
git clone https://github.com/vrolife/fingerprint-ocv
cd fingerprint-ocv
git submodule init
git submodule update
cmake -S . -B build
cmake --build build
sudo cp build/src/fingerprint-ocv /usr/local/bin/
```

### 3. Disable System fprintd

```bash
# Rename the D-Bus service file to prevent auto-start
sudo mv /usr/share/dbus-1/system-services/net.reactivated.Fprint.service \
       /usr/share/dbus-1/system-services/net.reactivated.Fprint.service.disabled
```

### 4. Configure D-Bus Permissions

```bash
sudo cat > /etc/dbus-1/system.d/net.reactivated.Fprint.conf << 'EOF'
<!DOCTYPE busconfig PUBLIC
 "-//freedesktop//DTD D-BUS Bus Configuration Version 1.0"
 "http://www.freedesktop.org/standards/dbus/1.0/busconfig.dtd">
<busconfig>
  <policy user="root">
    <allow own="net.reactivated.Fprint"/>
    <allow send_destination="net.reactivated.Fprint"/>
  </policy>
  <policy context="default">
    <allow own="net.reactivated.Fprint"/>
    <allow send_destination="net.reactivated.Fprint"/>
  </policy>
</busconfig>
EOF
```

### 5. Set Up systemd Service

```bash
sudo cat > /etc/systemd/system/fingerprint-ocv.service << 'EOF'
[Unit]
Description=Fingerprint Driver
After=dbus.service

[Service]
Type=simple
ExecStart=/usr/local/bin/fingerprint-ocv --bus=system
Environment=OPENSSL_CONF=/dev/null
Restart=on-failure

[Install]
WantedBy=multi-user.target
EOF

sudo systemctl daemon-reload
sudo systemctl enable --now fingerprint-ocv
```

### 6. Verify Installation

```bash
systemctl status fingerprint-ocv
fprintd-verify
```

## Troubleshooting

### "dbus_bus_request_name failed: Request to own name refused by policy"

- Run step 3 to disable system fprintd
- Run step 4 to configure D-Bus permissions
- Restart D-Bus: `sudo systemctl restart dbus-broker-service`

### "usbi_mutex_lock: Assertion failed"

- Run driver with `OPENSSL_CONF=/dev/null` environment variable

### TLS/SSL errors during connection

- Set `OPENSSL_CONF=/dev/null` in systemd service environment

## Commands Reference

```bash
# Check device
lsusb | grep 10a5

# Verify driver is running
systemctl status fingerprint-ocv

# Enroll fingerprint
fprintd-enroll

# Verify fingerprint
fprintd-verify

# Restart driver
sudo systemctl restart fingerprint-ocv

# Stop driver
sudo systemctl stop fingerprint-ocv
```

## Restore System fprintd (if needed)

```bash
sudo mv /usr/share/dbus-1/system-services/net.reactivated.Fprint.service.disabled \
       /usr/share/dbus-1/system-services/net.reactivated.Fprint.service
sudo systemctl disable fingerprint-ocv
sudo systemctl restart fprintd
```
