# OpenWrt for Fritz!Box 7490

Custom OpenWrt firmware build for the AVM Fritz!Box 7490 with WiFi support. This repository only includes automated builds via GitHub Actions and supports both OpenWrt 24.10 and 25.12.4 versions.

## Overview

This build uses a dual-architecture approach:
- **Lantiq (XRX200)**: Main system that boots from NAND
- **ATH79 WASP**: Secondary processor loaded from RAM to handle WiFi radios (2.4GHz and 5GHz)

The build includes custom patches, pre-configured network settings, WiFi mesh support, and automated firmware extraction/integration.

## Features

- ✅ Full WiFi support (2.4GHz and 5GHz)
- ✅ 802.11r Fast Roaming (FT)
- ✅ 802.11k/v (RRM, BSS Transition)
- ✅ WiFi mesh support (802.11s with SAE encryption)
- ✅ Pre-configured network, firewall, and system settings
- ✅ Automated build pipeline (GitHub Actions)
- ✅ Support for OpenWrt 24.10 and 25.12.4

## Repository Structure

```
.
├── patches/                   # Custom patches for Fritz!Box 7490 support
│   ├── 0001-*.patch           # Lantiq port 5 configuration for WASP
│   ├── 0002-0004-*.patch      # AVM WASP kernel module
│   ├── 0005-*.patch           # ATH79 support for Fritz!Box 3490/5490/7490
│   ├── 0008-*.patch           # ath9k EEPROM extraction
│   ├── 0009-*.patch           # ath9k/ath10k calibration placeholders
│   └── 0010-0011-*.patch      # USB and WASP firmware placeholders
├── firmware/                  # Required firmware files (YOU NEED TO GET SOME)
│   ├── ath9k-eeprom-ahb-18100000.wmac.bin   # Extract from running Lantiq
│   ├── ath_tgt_fw1.fw                       # Provided in the latest tarballs from AVM
│   ├── cal-pci-0000:00:00.0.bin             # Extract from running Lantiq
│   ├── lantiq-vrx200-b.bin                  # From OpenWrt repository
│   └── xhcifw.mem                           # From GitHub
├── files/                     # Custom configuration files
│   └── etc/
│       ├── rc.local.template  # Startup script for WASP initialization
│       ├── config/            # UCI configuration files
│       │   ├── network.template
│       │   ├── firewall
│       │   ├── dhcp
│       │   ├── ddns.template
│       │   ├── dropbear
│       │   ├── sqm
│       │   └── system
│       ├── dropbear/
│       │   └── authorized_keys.template
│       └── hotplug.d/iface/
│           └── 99-dslite-mtu
├── .config-lantiq             # Lantiq build configuration
├── .config-lantiq-24.10       # Lantiq config for OpenWrt 24.10
├── .config-lantiq-25.12.4     # Lantiq config for OpenWrt 25.12.4
├── .config-wasp               # WASP build configuration
├── .config-wasp-24.10         # WASP config for OpenWrt 24.10
├── .config-wasp-25.12.4       # WASP config for OpenWrt 25.12.4
├── .github/workflows/
│   └── build.yaml             # Automated build workflow
└── secret-sync.sh             # Push secrets to GitHub repository
```

## Prerequisites

### Required Firmware Files

**⚠️ Legal Notice**: Due to licensing restrictions, proprietary firmware files cannot be distributed in this repository.

You need the following firmware files:
- `ath9k-eeprom-ahb-18100000.wmac.bin` - WiFi calibration data (extracted from running system)
- `cal-pci-0000:00:00.0.bin` - WiFi calibration data (extracted from running system)

## Build Method

### GitHub Actions

1. **Fork this repository** and set up secrets:
   ```bash
   # Create .secrets directory with required files:
   # - SSH_PRIVATE_KEY: SSH key for deployment
   # - WIFI_PASSWORD: Your WiFi password
   # - WIFI_SSID: Your WiFi SSID
   # - MESH_PASSWORD: WiFi mesh password
   
   # Push secrets to GitHub
   ./secret-sync.sh
   ```

2. **Trigger the build**:
   - Go to Actions → "Build OpenWrt Fritz!Box 7490" → Run workflow
   - Select OpenWrt version (24.10 or 25.12.4)
   - Choose build options (clean build, update sources, build target)

3. **Download artifacts**:
   - Firmware images will be available as workflow artifacts
   - Both Lantiq and WASP images are built and integrated automatically

## Extracting Required Firmware Files

### Overview

The Fritz!Box 7490 requires several proprietary firmware files for full functionality:

1. **WiFi Calibration Data** - Device-specific calibration files for 2.4GHz and 5GHz radios

WiFi calibration data is **device-specific** and must be extracted from a running Lantiq OpenWrt system on your Fritz!Box 7490.

#### Step 1: Build and Flash Initial Lantiq Image

First, build and flash the Lantiq image **without** WiFi support (WASP not yet configured). This initial image allows you to extract the calibration data.

#### Step 2: Boot and Extract Calibration Files

After flashing and booting into the Lantiq OpenWrt:

```bash
# Connect to the device (default IP: 192.168.1.1)
ssh root@192.168.1.1

# Check if calibration files exist
ls -la /lib/firmware/ath9k-eeprom-ahb-18100000.wmac.bin
ls -la /lib/firmware/ath10k/cal-pci-0000:00:00.0.bin
```

If the files exist, copy them to your build machine:

```bash
# From your build machine
scp root@192.168.1.1:/lib/firmware/ath9k-eeprom-ahb-18100000.wmac.bin firmware/
scp root@192.168.1.1:/lib/firmware/ath10k/cal-pci-0000:00:00.0.bin firmware/
```

**Note**: These calibration files are generated by the Lantiq OpenWrt patches included in this repository. They are extracted from the hardware EEPROM on first boot.

### Firmware Directory Structure

After extraction, your `firmware/` directory should contain:

```
firmware/
├── ath9k-eeprom-ahb-18100000.wmac.bin    # Extract from running Lantiq
├── ath_tgt_fw1.fw                        # Provided in the latest tarballs from AVM
├── cal-pci-0000:00:00.0.bin              # Extract from running Lantiq
├── lantiq-vrx200-b.bin                   # From OpenWrt repository
└── xhcifw.mem                            # From GitHub
```

### Troubleshooting Firmware Extraction

#### unsquashfs4-avm-be: Command not found

Install dependencies for building Freetz-NG tools:
```bash
sudo apt-get update
sudo apt-get install build-essential git wget curl bzip2 \
  libtool automake autoconf m4 pkg-config
```

#### 7z: Command not found

Install p7zip:
```bash
# Debian/Ubuntu
sudo apt-get install p7zip-full

# macOS
brew install p7zip
```

#### Firmware extraction fails

- Ensure you're using the correct AVM firmware version for Fritz!Box 7490
- Check that `unsquashfs4-avm-be` is executable: `chmod +x unsquashfs4-avm-be`

### References

- [OpenWrt PR #5075 - Fritz!Box 7490 Support](https://github.com/openwrt/openwrt/pull/5075)
- [OpenWrt Forum - Support Fritzbox 7490](https://forum.openwrt.org/t/support-fritzbox-7490/4112/)
- [AVM Source Code](https://osp.avm.de/fritzbox/fritzbox-7490/)
- [Armbian Firmware GitHub Archive](https://github.com/armbian/firmware/tree/master)
- [d-patch GitHub repo](https://github.com/d-patch/openwrt-fritz7490)

## Flashing Instructions

### Initial Flash (from AVM Stock Firmware)

1. Download your Fritz!Box 7490 to AVM stock firmware
2. Use the AVM recovery mode or web interface to flash the Lantiq image
3. The WASP WiFi processor will be initialized automatically on boot

### Upgrading from Previous OpenWrt

```bash
scp openwrt-lantiq-xrx200-avm_fritz7490-squashfs-sysupgrade.bin root@192.168.1.1:/tmp/
ssh root@192.168.1.1 'sysupgrade /tmp/openwrt-lantiq-xrx200-avm_fritz7490-squashfs-sysupgrade.bin'
```

## Configuration

### Network Setup

Default configuration (defined in `files/etc/config/network.template`):
- **LAN**: 192.168.1.1/24
- **WASP**: 192.168.1.2 (WiFi processor, connected to main system)
- **WAN**: DHCP (configurable for PPPoE, DS-Lite, etc.)

### WiFi Configuration

WiFi is automatically configured via `rc.local` script on boot:
- **5GHz (radio0)**: Channel 44, HT40, 802.11ac
  - AP mode with your SSID
  - Mesh mode (802.11s) for multi-AP setups
- **2.4GHz (radio1)**: Channel 1, HT40, 802.11n
  - AP mode with your SSID

Both radios support:
- Fast Roaming (802.11r)
- RRM (802.11k)
- BSS Transition (802.11v)
- Protected Management Frames (802.11w)
- WPS push button

### Customizing WiFi Settings

Edit `files/etc/rc.local.template` before building, or modify directly on the device:

```bash
ssh root@192.168.1.1
vi /etc/rc.local
# Make changes, then:
/etc/rc.local  # Re-run the configuration script
```

## Troubleshooting

### WASP Not Starting

Check if the WASP module is loaded:
```bash
lsmod | grep avm_wasp
dmesg | grep -i wasp
```

Manually reload:
```bash
rmmod avm_wasp
modprobe avm_wasp
```

### WiFi Not Working

Check WASP connectivity:
```bash
ping 192.168.1.2
ssh root@192.168.1.2
# On WASP:
wifi status
logread | grep -i wifi
```

Re-run WiFi configuration:
```bash
/etc/rc.local
```

## Advanced Configuration

### Custom Packages

Edit `.config-lantiq` or `.config-wasp` files to add packages:
```bash
make menuconfig
# Navigate to desired packages
# Save configuration
cp .config .config-lantiq-25.12.4  # or appropriate config file
```

### Network Customization

Modify `files/etc/config/network.template`:
- Change LAN IP range
- Configure VLANs
- Set up additional interfaces

### Firewall Rules

Edit `files/etc/config/firewall` to add custom rules.

### SQM QoS

The build includes SQM (Smart Queue Management) for bufferbloat control. Configure via:
```bash
ssh root@192.168.1.1
vi /etc/config/sqm
# Or via LuCI: Network → SQM QoS
```

## Credits

This build is based on work by:
- [Zappception/openwrt](https://github.com/Zappception/openwrt)
- kestrel1974, jschwartzenberg, timocapa for Fritz!Box 7490 patches and WASP support
- OpenWrt community

## Resources

- [OpenWrt Documentation](https://openwrt.org/docs)
- [OpenWrt Forum](https://forum.openwrt.org)
- [Fritz!Box 7490 Device Page](https://openwrt.org/toh/avm/avm_fritz_box_7490)
- [Bug Reports](https://bugs.openwrt.org)

## License

This project inherits licenses from OpenWrt (GPL-2.0) and included components. See individual files for specific license information.

---

**⚠️ Disclaimer**: Flashing custom firmware may void your warranty and carries risk. Proceed at your own risk. Always have a backup plan to recover your device.
