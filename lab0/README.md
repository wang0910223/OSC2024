# Lab0: Environment Setup

## Cross-Platform Development

### Cross Compiler 
Rpi3 uses ARM Cortex-A53 CPU. To compile the source code to 64-bit ARM machine code, we need a cross compiler to develop on a non-ARM64 environment.

```bash
# using this command to install the tools we need
sudo apt install gcc make gcc-aarch64-linux-gnu binutils-aarch64-linux-gnu
```

### Linker
Linker is used to place the sections from different input files(object file) to the appropriate output sections in the single output file(executable).

Here is an incomplete linker script.
- `.` is location counter, indicate the starting position of the section plcement. The location counter is initialized to zero by default.
- `.text` is defined as an output section.
    - `.text : { *(.text) }` will place all .text input sections from all input files at the address that was just assigned to the location counter, i.e., 0x80000.
```
# linker.ld
SECTIONS
{
  . = 0x80000;
  .text : { *(.text) }
}
```

### QEMU
We can test our code first on the QEMU platform before running on the actual board.

``` bash
# The `-arm` flag is added to specify the packages for installation.
sudo apt install qemu-system-arm
```


## From Source Code to Kernel Image

### From Source Code to Object Files
We use the cross compiler to convert the source code to the object file. The file `start.s` is our source code, we can convert it by `aarch64-linux-gnu-gcc -c start.S`.
``` asm
# start.s
.section ".text"
_start:
  wfe
  b _start
```

### From Object Files to ELF
A linker links object files to an ELF file. An ELF file can be loaded and executed by program loaders. Program loaders are usually provided by the operating system in a regular development environment. In bare-metal programming, ELF can be loaded by some bootloaders. 
We can use the following command to convert `aarch64-linux-gnu-ld -T linker.ld -o kernel8.elf start.o`.  Here, we use the provided linker script `linker.ld`, the `-T` option means to use the specific linker, rather than the default one.

### From ELF to Kernel Image
Rpi3’s bootloader can’t load ELF files. Hence, we need to convert the ELF file to a raw binary image. We can use objcopy to convert ELF files to raw binary.
``` bash
aarch64-linux-gnu-objcopy -O binary kernel8.elf kernel8.img
```


### Check on QEMU
After building, we can use QEMU to see the dumped assembly.
``` bash
qemu-system-aarch64 -M raspi3b -kernel kernel8.img -display none -d in_asm
```


## Deploy to REAL Rpi3

## Debugging
### Debug on QEMU
### Debug on Real Rpi3
