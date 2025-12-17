# Custom WSL2 Linux Kernel - CGroupsV2

This repository is forked from the excellent [Windows WSL2 Kernel Build Script repo by slyfox1186](https://github.com/slyfox1186/windows-wsl2-kernel-build-script).

This project aims to use GitHub Actions Workflows to produce and publish up-to-date, versioned custom builds of [WSL2-Linux-Kernel](https://github.com/microsoft/WSL2-Linux-Kernel) with [`HIDDEV`](https://docs.kernel.org/hid/hiddev.html) and [`HIDRAW`](https://docs.kernel.org/hid/hidraw.html) enabled, **plus full CGroupsV2 support and kernel IO functions like `io.weight`**.

These custom kernels builds can be used to enable full Yubikey passthrough to WSL2 using [`usbipd`](https://github.com/dorssel/usbipd-win), with full FIDO2 functionality, **and add the missing kernel IO functions from Microsoft's original kernel (like `weight`, min/max limits, and proportional I/O control in CGroupsV2)**.

The versioning scheme of this project matches the versioning scheme used by WSL2-Linux-Kernel.

## Added Features
- **HIDDEV and HIDRAW**: For full USB HID device support in WSL2.
- **Full CGroupsV2**: Enabled with `systemd.unified_cgroup_hierarchy=1` and all required options.
- **Kernel IO functions**: Includes `CONFIG_BLK_CGROUP_IOCOST=y`, `CONFIG_IO_WEIGHT=y`, and IO scheduling controllers with weights (`weight`), missing from Microsoft's base kernel to optimize containers and disk-intensive workloads.

## Usage

- Download the custom kernel from the releases page
- Make sure you have saved all your work in all WSL2 instances
- Shutdown all WSL2 instances with `wsl --shutdown`
- Edit (or create) the ~/.wslconfig file on Windows
- Specify the path to the custom kernel

```ini
# For example...
[wsl2]
kernel=C:\\Users\\YOUR_USERNAME\\Downloads\\vmlinux
```

- Start a WSL2 instance
- Check that the kernel is running with `uname -sr`

```
Linux 5.15.123.1-lgug2z-custom-WSL2
```

## CGroupsV2 IO.Weight

To make functional the IO.Weight functional to your Ubuntu Distro in WSL you need to run this commands

```
#!/bin/bash
set -e

CONFIG="DefaultIOAccounting=yes"

FILES=(
    "/etc/systemd/system.conf"
    "/etc/systemd/user.conf"
)

for FILE in "${FILES[@]}"; do
    if grep -q "^DefaultIOAccounting=" "$FILE"; then

        sudo sed -i "s/^DefaultIOAccounting=.*/$CONFIG/" "$FILE"
    else
        echo "$CONFIG" | sudo tee -a "$FILE" > /dev/null
    fi
    echo "Configuration applied in $FILE"
    sudo systemctl daemon-reexec
done
```

To test if properly works you can use this command

```
cat /sys/fs/cgroup/system.slice/cgroup.subtree_control
```

if you see IO on the cat it's working well


## Modification

If you want to build and publish your own custom WSL2 Linux Kernel, you can
fork this repository and make whatever configuration modifications in
[config-wsl](config-wsl). The [GitHub Actions
Workflow](.github/workflows/build.yml) will take care of the rest.

Please take care to update `CONFIG_LOCALVERSION` to distinguish your custom
kernel from this one.
