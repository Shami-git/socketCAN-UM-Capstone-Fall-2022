No Longer Totally Stolen from: A gentle introduction to Linux Kernel fuzzing
=============================================

Requirements
-------------
qemu-kvm, kernel development tools, static busybox, Linux kernel tar - this project uses 5.19.7.


Compiling
--------------

Download kernel tar from: https://www.kernel.org/pub/linux/kernel/v5.x/linux-5.19.7.tar.xz

Untar, run this script to add KCOV instrument for code coverage before building new kernel:

	find net -name Makefile | xargs -L1 -I {} bash -c 'echo "KCOV_INSTRUMENT := y" >> {}'

Before compiling, replace the .config file in the Linux source dir with the included config
file from the confs directory, be sure it is renamed correctly to ".config".
 
Compile the new Linux kernel with this config file.

To create the initird

	make rootfs

should build fuzzcan, place it in the initramfs and 
then create a new initramfs at the current directory.


Running
---------------

Launch qemu:

	qemu-system-x86_64 -kernel /path/to/bzImage -initrd /path/tofuzzcan-initramfs.cpio.gz \ 
	-append "root=/dev/ram0 rootfstype=ramfs init=/init console=ttyS0" -net nic,model=rtl8139 \
 	-net user -m 2048M --nographic

Run fuzzcan with: 

	fuzzcan -r 

You may have to press "enter" again after executing. This should yield a stream of hex pointers. 
To view these in user format: create a text file, copy the data stream and paste into this file.
Save it and run this file through addr2line:

	cat output_file | addr2line -e /path/to/vmlinux 

(^^^ careful! .not the bzImage that is in kernelsrc/arch/x86_64/boot.
The vmlinux (the uncompressed pure kernel binary. is just under kernelsrc/)
