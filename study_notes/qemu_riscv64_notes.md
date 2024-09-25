**Question:** So, in xv6-riscv OS, the loader loads the xv6 kernel into memory at physical address 0x80000000. The reason it places the kernel at 0x80000000 rather than 0x0 is because the address range 0x0:0x80000000 contains I/O devices. Can you show me that? 

The result from terminal #1 says the firmware base starts at 0x80000000, so probably that is why. Between 0x00000000 and 0x80000000, is probably reserved for some other i/o device stuff.

#### With some googling:<br>

**Boot ROM**, an integrated circuit (chip) that is located on the motherboard and stores the firmware code responsible for booting the computer. This name is not standardized, so other developers often call it **Flash ROM**, **BIOS Flash**, **Boot Flash**, **SPI Flash**, etc but these terms are interchangeable. <br>


A **firmware** is a low level software that provides the necessary instructions for hardware initialization and control. It operates between the hardware and the operating system, often providing essential services such as hardware abstraction, device drivers, and system initilization routines. (The firmware code in the **Boot ROM** is executed first when the computer is powered on. Performing basic tests, initializas the hardware, and then loads the OS loader from a bootable device, such as a hard drive or a USB drive, into memory. The integrated circuit(chip) is made from **non-volatile memory (NVM)**)<br>


The firmware in our case, is loaded at a base address of 0x80000000, meaning that when the RISC-V processor starts, it looks at this memory location to begin executing instructions. The firmware itself is stored in memory and contains the necessary code to bootstrap the system. 


To give more context, in the initialization process, when the system is powered on or reset, the processor begins execution at a predefined address (firmware base). The firmware will:


1. Initialize hardware components i.e. timers, I/O devices, etc <br>
2. Set up memory management<br>
3. Configure CPU settings<br>
4. Load the operating system kernel into memory.<br>

Device detection:<br>
 
The firmware facilitates communication between the CPU and peripheral devices using drivers. In our example, the firmware communicates with devices like the console (uart8250) and timers (aclintmtimer).<br>

Runtime SBI (Supervisor Binary Interface): <br>

The firmware implements a Supervisor Binary Interface (SBI), which provides a set of services for the operating system to interact with the hardware.The version indicated (1.0) signifies the SBI specification that the firmware adheres to.

**Boot Process and Boot Time**<br>

1. Boot process<br>
Power-On Reset: The processor resets and starts executing instructions from the firmware base. <br>

Firmware Execution: The firmware initializes hardware and sets up memory.<br>

Loading the OS: The firmware loads the operating system kernel into memory, often using a specific bootloader.<br>

Transition to OS: Once the OS is loaded, control is handed over, and the OS takes over managing the system.<br>

2. Boot Tme<br>
Boot time refers to the duration from powering on the device to when the operating system is ready for user interaction. This includes:<br>
Time taken for firmware to initialize hardware.<br>
Time taken to load the OS into memory.<br>
Time for the OS to complete its initialization.<br>

3. HART (Hardware thread)<br>
In RISC-V, HART refers to a hardware thread. In our case (see example output from terminal below), there's one HART (HART Count: 1) that runs the firmware. The firmware can manage multiple HARTs for parallel processing, depending on the architecture.<br>

4. Memory Regions and Privilege Levels<br>
The firmware sets up memory regions that define access permissions (read, write, execute). This segmentation is critical for maintaining system security and stability. Privilege levels like S-mode for Supervisor mode determine what operations can be performed by the executing code.


 


(extra note: <br>
qemu and risc-v firmware:<br>

1. firmware image:<br>

qemu includes a firmware image (often based on OpenSBI or similar implementation) that emulates the necessary low-level initialization routines for riscv systems. This firmware is responsible for booting the system and providing the SBI interface for the guest operating system. <br>

2. booting process:<br>

when you start a riscv emulated system in qemu, it loads the firmware images from a specified location (or uses a default one if not specified) into memory at the designated firmware base address (i.e. 0x80000000).<br>
The firmware executes and initializes the emulated hardware environment, prepares the memory map, and loads the operating system kernel into RAM.<br>

3. OpenSBI: <br>
qemu uses OpenSBI (Open Supervisor Binary Interface) as the firmware implementation. OpenSBI provides the necessary SBI functions that the operating system can call to interact with the hardware, such as managing interrupts and handling power management. <br>

4. Configurable Options:<br>
When running a qemu emulation, you can speciy various options for the firmware, such as the path to a custom firmware image or particular configurations for the emulated environment. The default firmware location (linux): /usr/share/qemu, etc.
If you want to use a custom firmware image (e.g. a specific version of OpenSBI), you will need to know the path to that image. Specify path to custom firmware image using the **-bios** option. <br>
```
qemu-system-riscv64 -bios /path/to/your/firmware/image.elf -nographic -smp 1 -m 512M
``` 

If you want to build OpenSBI:

```
git clone https://github.com/riscv/opensbi.git

cd opensbi

make -C <path-to-your-riscv-toolchain> ARCH=riscv

//after building, the firmware image (default name opensbi.elf) will be in the build directory.

qemu-system-riscv64 -bios ./build/opensbi.elf -nographic -smp 1 -m 512M
```
)<br>

 

To monitor qemu via sockets (terminal #1):

```
jerry@linux:~$ qemu-system-riscv64 -nographic -machine virt -m 2G -monitor unix:/tmp/qemu-monitor-socket,server,nowait

OpenSBI v1.3
   ____                    _____ ____ _____
  / __ \                  / ____|  _ \_   _|
 | |  | |_ __   ___ _ __ | (___ | |_) || |
 | |  | | '_ \ / _ \ '_ \ \___ \|  _ < | |
 | |__| | |_) |  __/ | | |____) | |_) || |_
  \____/| .__/ \___|_| |_|_____/|___/_____|
        | |
        |_|

Platform Name             : riscv-virtio,qemu
Platform Features         : medeleg
Platform HART Count       : 1
Platform IPI Device       : aclint-mswi
Platform Timer Device     : aclint-mtimer @ 10000000Hz
Platform Console Device   : uart8250
Platform HSM Device       : ---
Platform PMU Device       : ---
Platform Reboot Device    : sifive_test
Platform Shutdown Device  : sifive_test
Platform Suspend Device   : ---
Platform CPPC Device      : ---
Firmware Base             : 0x80000000
Firmware Size             : 322 KB
Firmware RW Offset        : 0x40000
Firmware RW Size          : 66 KB
Firmware Heap Offset      : 0x48000
Firmware Heap Size        : 34 KB (total), 2 KB (reserved), 9 KB (used), 22 KB (free)
Firmware Scratch Size     : 4096 B (total), 760 B (used), 3336 B (free)
Runtime SBI Version       : 1.0

Domain0 Name              : root
Domain0 Boot HART         : 0
Domain0 HARTs             : 0*
Domain0 Region00          : 0x0000000002000000-0x000000000200ffff M: (I,R,W) S/U: ()
Domain0 Region01          : 0x0000000080040000-0x000000008005ffff M: (R,W) S/U: ()
Domain0 Region02          : 0x0000000080000000-0x000000008003ffff M: (R,X) S/U: ()
Domain0 Region03          : 0x0000000000000000-0xffffffffffffffff M: (R,W,X) S/U: (R,W,X)
Domain0 Next Address      : 0x0000000000000000
Domain0 Next Arg1         : 0x00000000bfe00000
Domain0 Next Mode         : S-mode
Domain0 SysReset          : yes
Domain0 SysSuspend        : yes

Boot HART ID              : 0
Boot HART Domain          : root
Boot HART Priv Version    : v1.12
Boot HART Base ISA        : rv64imafdch
Boot HART ISA Extensions  : time,sstc
Boot HART PMP Count       : 16
Boot HART PMP Granularity : 4
Boot HART PMP Address Bits: 54
Boot HART MHPM Count      : 16
Boot HART MIDELEG         : 0x0000000000001666
Boot HART MEDELEG         : 0x0000000000f0b509
qemu-system-riscv64: terminating on signal 2
jerry@linux:~$ ^C
jerry@linux:~$ ^C
jerry@linux:~$ 
```


when I run (on terminal #2) using netcat (nc):

```
jerry@linux:~$ nc -U /tmp/qemu-monitor-socket
QEMU 8.2.2 monitor - type 'help' for more information
(qemu) info devices
info devices
unknown command: 'info devices'
(qemu) info mtree
info mtree
address-space: I/O
  0000000000000000-000000000000ffff (prio 0, i/o): io

address-space: gpex-root
  0000000000000000-ffffffffffffffff (prio 0, i/o): bus master container

address-space: cpu-memory-0
address-space: memory
  0000000000000000-ffffffffffffffff (prio 0, i/o): system
    0000000000001000-000000000000ffff (prio 0, rom): riscv_virt_board.mrom
    0000000000100000-0000000000100fff (prio 0, i/o): riscv.sifive.test
    0000000000101000-0000000000101023 (prio 0, i/o): goldfish_rtc
    0000000002000000-0000000002003fff (prio 0, i/o): riscv.aclint.swi
    0000000002004000-000000000200bfff (prio 0, i/o): riscv.aclint.mtimer
    0000000003000000-000000000300ffff (prio 0, i/o): gpex_ioport_window
      0000000003000000-000000000300ffff (prio 0, i/o): gpex_ioport
    0000000004000000-0000000005ffffff (prio 0, i/o): platform bus
    000000000c000000-000000000c5fffff (prio 0, i/o): riscv.sifive.plic
    0000000010000000-0000000010000007 (prio 0, i/o): serial
    0000000010001000-00000000100011ff (prio 0, i/o): virtio-mmio
    0000000010002000-00000000100021ff (prio 0, i/o): virtio-mmio
    0000000010003000-00000000100031ff (prio 0, i/o): virtio-mmio
    0000000010004000-00000000100041ff (prio 0, i/o): virtio-mmio
    0000000010005000-00000000100051ff (prio 0, i/o): virtio-mmio
    0000000010006000-00000000100061ff (prio 0, i/o): virtio-mmio
    0000000010007000-00000000100071ff (prio 0, i/o): virtio-mmio
    0000000010008000-00000000100081ff (prio 0, i/o): virtio-mmio
    0000000010100000-0000000010100007 (prio 0, i/o): fwcfg.data
    0000000010100008-0000000010100009 (prio 0, i/o): fwcfg.ctl
    0000000010100010-0000000010100017 (prio 0, i/o): fwcfg.dma
    0000000020000000-0000000021ffffff (prio 0, romd): virt.flash0
    0000000022000000-0000000023ffffff (prio 0, romd): virt.flash1
    0000000030000000-000000003fffffff (prio 0, i/o): alias pcie-ecam @pcie-mmcfg-mmio 0000000000000000-000000000fffffff
    0000000040000000-000000007fffffff (prio 0, i/o): alias pcie-mmio @gpex_mmio_window 0000000040000000-000000007fffffff
    0000000080000000-00000000ffffffff (prio 0, ram): riscv_virt_board.ram
    0000000400000000-00000007ffffffff (prio 0, i/o): alias pcie-mmio-high @gpex_mmio_window 0000000400000000-00000007ffffffff

memory-region: system
  0000000000000000-ffffffffffffffff (prio 0, i/o): system
    0000000000001000-000000000000ffff (prio 0, rom): riscv_virt_board.mrom
    0000000000100000-0000000000100fff (prio 0, i/o): riscv.sifive.test
    0000000000101000-0000000000101023 (prio 0, i/o): goldfish_rtc
    0000000002000000-0000000002003fff (prio 0, i/o): riscv.aclint.swi
    0000000002004000-000000000200bfff (prio 0, i/o): riscv.aclint.mtimer
    0000000003000000-000000000300ffff (prio 0, i/o): gpex_ioport_window
      0000000003000000-000000000300ffff (prio 0, i/o): gpex_ioport
    0000000004000000-0000000005ffffff (prio 0, i/o): platform bus
    000000000c000000-000000000c5fffff (prio 0, i/o): riscv.sifive.plic
    0000000010000000-0000000010000007 (prio 0, i/o): serial
    0000000010001000-00000000100011ff (prio 0, i/o): virtio-mmio
    0000000010002000-00000000100021ff (prio 0, i/o): virtio-mmio
    0000000010003000-00000000100031ff (prio 0, i/o): virtio-mmio
    0000000010004000-00000000100041ff (prio 0, i/o): virtio-mmio
    0000000010005000-00000000100051ff (prio 0, i/o): virtio-mmio
    0000000010006000-00000000100061ff (prio 0, i/o): virtio-mmio
    0000000010007000-00000000100071ff (prio 0, i/o): virtio-mmio
    0000000010008000-00000000100081ff (prio 0, i/o): virtio-mmio
    0000000010100000-0000000010100007 (prio 0, i/o): fwcfg.data
    0000000010100008-0000000010100009 (prio 0, i/o): fwcfg.ctl
    0000000010100010-0000000010100017 (prio 0, i/o): fwcfg.dma
    0000000020000000-0000000021ffffff (prio 0, romd): virt.flash0
    0000000022000000-0000000023ffffff (prio 0, romd): virt.flash1
    0000000030000000-000000003fffffff (prio 0, i/o): alias pcie-ecam @pcie-mmcfg-mmio 0000000000000000-000000000fffffff
    0000000040000000-000000007fffffff (prio 0, i/o): alias pcie-mmio @gpex_mmio_window 0000000040000000-000000007fffffff
    0000000080000000-00000000ffffffff (prio 0, ram): riscv_virt_board.ram
    0000000400000000-00000007ffffffff (prio 0, i/o): alias pcie-mmio-high @gpex_mmio_window 0000000400000000-00000007ffffffff

memory-region: pcie-mmcfg-mmio
  0000000000000000-000000000fffffff (prio 0, i/o): pcie-mmcfg-mmio

memory-region: gpex_mmio_window
  0000000000000000-ffffffffffffffff (prio 0, i/o): gpex_mmio_window
    0000000000000000-ffffffffffffffff (prio 0, i/o): gpex_mmio
```

```
jerry@linux:/usr/share/qemu$ ls
bamboo.dtb         linuxboot.bin        openbios-sparc32                        petalogix-s3adsp1800.dtb  s390-ccw.img      u-boot-sam460-20100605.bin
canyonlands.dtb    linuxboot_dma.bin    openbios-sparc64                        pvh.bin                   s390-netboot.img  vhost-user
hppa-firmware.img  multiboot.bin        opensbi-riscv64-generic-fw_dynamic.bin  qboot.rom                 skiboot.lid       vof.bin
init               multiboot_dma.bin    opensbi-riscv64-generic-fw_dynamic.elf  QEMU,cgthree.bin          slof.bin
keymaps            npcm7xx_bootrom.bin  palcode-clipper                         QEMU,tcx.bin              trace-events-all
kvmvapic.bin       openbios-ppc         petalogix-ml605.dtb                     QEMU,VGA.bin              u-boot.e500

```

**Other notes**<br>

In the xv6-riscv makefile, you might see something like
``` 
qemu-system-riscv64 -machine virt -bios none -kernel kernel/kernel -m 128M -smp 3 -nographic -drive file=fs.img,if=none,format=raw,id=x0 -device virtio-blk-device,drive=x0,bus=virtio-mmio-bus.0 
```

```qemu-system-riscv64```:
    This specifies that you're using the QEMU emulator for the RISC-V 64-bit architecture. <br>

```-machine virt```:
    This option sets the machine type to virt, which is a generic virtual machine for RISC-V. It emulates various peripherals useful for development.

```-bios none```:
    This tells QEMU not to load any BIOS firmware. In the context of XV6, the operating system itself is often configured to handle system initialization.

```-kernel kernel/kernel```:
    This specifies the kernel binary to load. In this case, it's pointing to kernel/kernel, which is the compiled XV6 kernel.

```-m 128M```:
    This sets the amount of memory allocated to the VM to 128 MB.

```-smp 3```:
    This specifies the number of CPU cores (or threads) to emulate. Here, it is set to 3.

```-nographic```:
    This option disables graphical output and uses the terminal for input/output instead. This is common when running operating systems like XV6, which do not have a graphical interface.

```-drive file=fs.img,if=none,format=raw,id=x0```:
    This sets up a disk drive. file=fs.img specifies the image file to use as the disk. if=none indicates that this is not attached to a specific interface yet, and format=raw specifies that the image is in raw format. id=x0 assigns an identifier to this drive.

```-device virtio-blk-device,drive=x0,bus=virtio-mmio-bus.0```:
    This adds a VirtIO block device to the system. It connects the drive (identified by x0) to the virtio-mmio-bus.0. VirtIO is a virtualized device standard that allows for efficient communication between the guest operating system and the host.







