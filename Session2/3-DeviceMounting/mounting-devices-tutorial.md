# Docker Device Mounting Tutorial

This guide will teach you how to mount devices into Docker containers, enabling containers to interact with host hardware like USB devices, GPUs, storage devices, and more.

## Understanding Device Mounting

In Docker, containers are isolated from the host system by default. To allow a container to access hardware devices, you need to explicitly mount them using the `--device` flag or by mounting device files from `/dev`.

### Why Mount Devices?

- Access USB devices (cameras, serial devices, etc.)
- Use GPUs for machine learning workloads
- Access storage devices directly
- Interact with hardware sensors

## Prerequisites

Before starting, ensure you have:

- Docker installed and running (use `./setup.sh docker` if not installed)
- Appropriate permissions to access devices (may need sudo or docker group membership)
- Knowledge of which device you want to mount (e.g., `/dev/video0`, `/dev/ttyUSB0`)

### Check Available Devices

List available devices on your system:

```bash
# List all devices
ls -la /dev/

# List USB devices
lsusb

# List block devices (storage)
lsblk

# List serial devices
ls -la /dev/tty*
```

## Basic Device Mounting

### Method 1: Using --device Flag (Recommended)

The `--device` flag is the safest and most controlled way to mount devices.

**Syntax:**
```bash
docker run --device=<host-device>:<container-device>:<permissions> <image>
```

**Example: Mount a USB device**
```bash
docker run -it --device=/dev/ttyUSB0:/dev/ttyUSB0:rwm ubuntu:latest bash
```

**Permissions:**
- `r` - read
- `w` - write
- `m` - mknod (create device files)
- `rwm` - all permissions (most common)

### Method 2: Volume Mount (Less Common)

You can also mount devices as volumes, though this is less secure:

```bash
docker run -it -v /dev/ttyUSB0:/dev/ttyUSB0 ubuntu:latest bash
```

⚠️ **Warning:** Volume mounting devices can be risky and may not work for all device types.

## Common Use Cases

### 1. USB Serial Device (Arduino, Serial Ports)

Mount a serial device for communication:

```bash
# Find your device
ls -la /dev/ttyUSB* /dev/ttyACM*

# Run container with device access
docker run -it \
  --device=/dev/ttyUSB0:/dev/ttyUSB0:rwm \
  ubuntu:latest bash

# Inside container, verify access
ls -la /dev/ttyUSB0
```

## Advanced Mounting Options

### Mount Multiple Serial Devices

```bash
docker run -it \
  --device=/dev/ttyUSB0:/dev/ttyUSB0:rwm \
  --device=/dev/ttyUSB1:/dev/ttyUSB1:rwm \
  --device=/dev/ttyACM0:/dev/ttyACM0:rwm \
  ubuntu:latest bash
```

### Mount All Serial Devices with Wildcard (Privileged Mode)

⚠️ **Security Risk:** Only use for testing, never in production!

```bash
docker run -it --privileged ubuntu:latest bash
```

This gives the container access to all host devices and removes most security restrictions.

### Device Cgroup Rules for Serial Devices

Control serial device access with cgroup rules:

```bash
docker run -it \
  --device-cgroup-rule='c 188:* rmw' \
  ubuntu:latest bash
```

Format: `type major:minor permissions`
- `c` = character device
- `188` = major number for USB serial devices (ttyUSB*)
- `166` = major number for ACM devices (ttyACM*)
- `*` = all minor numbers
- `rmw` = read, mknod, write

**Multiple serial device types:**
```bash
docker run -it \
  --device-cgroup-rule='c 188:* rmw' \
  --device-cgroup-rule='c 166:* rmw' \
  ubuntu:latest bash
```

## Troubleshooting

### Device Not Found

**Problem:** Device doesn't appear in container

**Solutions:**

1. Verify device exists on host:
```bash
ls -la /dev/ttyUSB0
```

2. Check if device was unplugged/replugged (device number may change)

3. Use udev rules to create persistent device names

4. Mount parent device directory:
```bash
docker run -it --device=/dev/bus/usb ubuntu:latest bash
```

### Device Busy

**Problem:** Device is already in use

**Solutions:**

1. Check what's using the device:
```bash
lsof /dev/ttyUSB0
fuser /dev/ttyUSB0
```

2. Stop other processes accessing the device

3. Some devices can be shared (video), others cannot (serial)

## Security Considerations

### Risk Levels for Serial Devices

1. **Low Risk:** Read-only access to serial devices (monitoring only)
2. **Medium Risk:** Read-write access to serial sensors
3. **High Risk:** Read-write access to control devices (can affect physical equipment)
4. **Critical Risk:** Using `--privileged` mode

## Additional Resources

### Docker Documentation
- [Docker device documentation](https://docs.docker.com/engine/reference/commandline/run/#device)
- [Docker security best practices](https://docs.docker.com/engine/security/)
- [Docker Compose devices](https://docs.docker.com/compose/compose-file/#devices)

### Serial Communication
- [PySerial documentation](https://pyserial.readthedocs.io/)
- [Arduino Serial communication](https://www.arduino.cc/reference/en/language/functions/communication/serial/)
- [Serial Programming Guide for POSIX Systems](https://tldp.org/HOWTO/Serial-Programming-HOWTO/)

### Linux Resources
- [Linux devices documentation](https://www.kernel.org/doc/html/latest/admin-guide/devices.html)
- [udev rules guide](https://wiki.archlinux.org/title/Udev)
- [Serial port configuration](https://linux.die.net/man/1/stty)

### Testing Tools
- [minicom](https://salsa.debian.org/minicom-team/minicom) - Serial terminal emulator
- [screen](https://www.gnu.org/software/screen/) - Terminal multiplexer with serial support
- [picocom](https://github.com/npat-efault/picocom) - Minimal serial terminal
- [CuteCom](https://gitlab.com/cutecom/cutecom) - Graphical serial terminal

## Summary

Key takeaways for serial device mounting:
- Use `--device` flag to mount serial devices safely
- Always use minimal required permissions (rw vs rwm)
- Add container user to dialout group for permissions
- Use persistent device paths (by-id) in production
- Never use `--privileged` mode in production
- Implement reconnection logic for hotplug scenarios
- Test serial communication before deploying
- Document baud rate and serial parameters
- Handle serial errors and timeouts gracefully
- Log all serial communication in production environments

