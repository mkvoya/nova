# NOVA: NOn-Volatile memory Accelerated log-structured file system

## Introduction
NOVA is a log-structured file system designed for byte-addressable, non-volatile memories. NOVA is fully POSIX compliant so that existing applications need not be modified to run on NOVA. NOVA bypasses the block layer and OS page cache, writes to NVM directly and reduces the software overhead. 

NOVA provides strong data consistency guanrantees:

* Atomic metadata update: each directory operation is atomic.
* Atomic data update; for each `write` operation, the file data and the inode are updated in a transactional way.
* Atomic `msync`: NOVA supports `mmap` operation, and modified data is committed to NVM atomically on each `msync` operation.

With atomicity guarantees, NOVA is able to recover from system failures and restore to a consistent state.

## Building NOVA
NOVA works on x86-64 Linux kernel 4.0 and 4.2.

To run on kernel version 4.0, you need to apply the patches under `kernel-4.0-patches` directory to your kernel source, rebuild the kernel, and then build NOVA with a simple

~~~
#make
~~~

command.

To run on kernel version 4.2, it is even simpler. Just checkout the 4.2 branch and compile.

~~~
#git checkout 4.2
#make
~~~

## Running NOVA
NOVA runs on a physically contiguous memory region that is not used by the Linux kernel. To reserve the memory space you can boot the kernel with `memmap` command line option. 

For instance, adding `memmap=4G$8G` (or `memmap=4G\$8G` if you are using GRUB 2) to the kernel boot parameters will reserve 4GB memory starting from 8GB address. 

After the OS has booted, you can initialize NOVA with the following command:


~~~
#insmod nova.ko
#mount -t NOVA -o physaddr=0x200000000,init=4G NOVA /mnt/ramdisk 
~~~

The above commands create a NOVA instance that manages memory from 8GB (0x200000000) to 12GB, and the mount point is /mnt/ramdisk.

To recover an existing NOVA instance, mount NOVA without the init option, for example:

~~~
#mount -t NOVA -o physaddr=0x200000000 NOVA /mnt/ramdisk 
~~~

There are two scripts provided in the source code, `setup-nova.sh` and `remount-nova.sh` to help setup NOVA.

## Current limitations

* NOVA does not currently support extended attributes.
* NOVA only works on x86-64 kernels.
