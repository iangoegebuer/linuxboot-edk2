# Building LinuxBoot with OVMF
This file provides instructions for building LinuxBoot with OVMF (x86_64 architecture),or
booting to the UEFI Shell instead (06/25/2023).\
The "Directory Structure" blocks are there for visualization in case you run into issues.
## Initial setup
**Run:**
```bash
# all essential packages
$ sudo apt update
$ sudo apt install -y git nasm iasl build-essential uuid-dev python
$ git clone https://github.com/tianocore/edk2.git
$ cd ./edk2
$ git submodule update --init --recursive
$ make -C BaseTools
```
If there's update for submodules, use following git commands to get the latest submodules code.\
**Run:**
```bash
$ git pull
$ git submodule update --recursive
```
## Building your own kernel.efi Image
For in-depth instructions and explanations: https://github.com/linuxboot/book/blob/master/coreboot.u-root.systemboot/README.md.\
If you build the linux kernel following the instructions above you also need to enable EFI_STUB in below order:
```bash
1. Power managment and ACPI options -> ACPI(Advanced Configuration and Power Interface) Support
2. Processor type and features -> EFI runtime service support
3. Processor type and features -> EFI runtime service support -> EFI stub support
```
### 1. Clone the Linux Github repo
This section assumes that you currently are in the edk2 directory.
```
Directory Structure:
├── edk2
```
**Run:**
```bash
$ git clone https://github.com/torvalds/linux.git
$ cd linux
$ git checkout v6.1
```
Note: The Linux Kernel Version used for this ReadMe is v6.1.
### 2. Generate the initramfs for your kernel
This section assumes that you are currently in the edk2/linux directory.
```
Directory Structure:
├── edk2
    ├── linux (branch v6.1)
```
**Run:**
```bash
$ git clone https://github.com/u-root/u-root
$ cd u-root
$ go build
```
```
Directory Structure:
├── edk2
    ├── linux (branch v6.1)
        ├── u-root
```
Note: The u-root command will end up in $GOPATH/bin/u-root, so you may need to add $GOPATH/bin to your $PATH.\
**Run:**
```bash
$ ./u-root
$ xz --check=crc32 --lzma2=dict=512KiB /tmp/initramfs.linux_amd64.cpio
$ cd ../
```
### 3. Build the kernel image
This section assumes that you are currently in the edk2/linux directory.
```
Directory Structure:
├── edk2
    ├── linux
```
**Run:**
```bash
$ cp ../LinuxBoot/sample.config .config
```
or you can also download a different sample config directly from the link below and save it as .config in the linux directory:\
https://github.com/linuxboot/book/blob/master/coreboot.u-root.systemboot/linux-4.19.6-linuxboot.config\
**Run:**
```bash
$ make menuconfig # (OPTIONAL) Further configure linux kernel config with a Graphical User Interface
$ make -j$(nproc --ignore=1) # press Enter key until command runs or modify configs
```
```
Directory Structure:
├── edk2
    ├── linux
        ├── arch
            ├── x86
                ├── boot
                └── bzImage
```
**Run:**
```bash
$ mv arch/x86/boot/bzImage ../LinuxBoot/kernel.efi
```
```
Directory Structure:
├── edk2
    ├── LinuxBoot
    └── kernel.efi # used to be named bzImage
```
You have sucessfully built and placed a kernel.efi Image in the correct location.\
**Run:**
```bash
$ cd ../
```
## Building and running the OVMF Image on QEMU with or without LinuxBoot.
This section assumes that you currently are in the edk2 directory.
```
Directory Structure:
├── edk2
```
**Run:**
```bash
$ source edksetup.sh
```
### Build OVMF
*with default EDK2 UEFI shell*\
**Run:**
```bash
$ build -p OvmfPkg/OvmfPkgX64.dsc -a X64 -b DEBUG -t GCC5
```
*with LinuxBoot*\
**Run:**
```bash
$ build -p OvmfPkg/OvmfPkgX64.dsc -a X64 -b DEBUG -t GCC5 -D LINUXBOOT_ENABLE=TRUE
```
### Run on QEMU
**Run:**
```bash
$ mkdir hda-contents
$ qemu-system-x86_64 -L . --bios Build/OvmfX64/DEBUG_GCC5/FV/OVMF.fd -net none -hda fat:rw:hda-contents -nographic

```