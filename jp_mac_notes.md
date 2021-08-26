# JP Mac and iOS Notes

## Disk Operations
### Reformatting External Drives
When attempting to reformat an external drive I encountered this error:
> Mediakit reports not enough space on device for requested operation.  

Apparently it is very common.  Once it happens, using the Disk Utility app fails to access the disk in question.  The solution is to use the following commands in terminal.

1. `diskutil list`
    + lists the mounted disks.  Find the one in question (`disk0` and `disk1` tend to be the internal hard drive / SSD of your Mac.  My external was `disk2`).  You can verify this by checking the sizes, but also by checking in the Disk Utility app under the listing for "Device".  (Even if Disk Utility cannot access the disk to perform operations, it can read its basic info such as device name).  For clarity and to prevent accidental copy-paste erasures, I will use `disk99` as the pretend disk name for these instructions.  Note when using the `diskutil list` command you might see multiple listings for a single drive with names such as `disk99s2` -- these refer to different sectors on the disk; the device name itself is only the core name of `disk99`  

2. `diskutil unmountDisk force disk99`  
    + Forces the unmounting of the disk, freeing it for reformatting.  

3. `sudo dd if=/dev/zero of=/dev/rdisk99 bs=1024 count=1024`  
    + We are overwriting the boot sector (`bs`) as all zeroes.  Note we used 1024 to specify the first MiB of the disk, the boot sector.  If we wanted the first 4 MiB, we'd use 1024 * 4 = 4096, etc.  By using only the first MiB we can address the likely problem area without having to rewrite the entire disk in zeroes (which could take a long time).
    + `dd` stands for "data definition" in Unix, and it requires `sudo` access (must enter your admin password).
    + `rdisk99` vs. `disk99`: the `r` stands for "raw device" and is supposed to communicate directly with the disk.  Omitting the `r` means to use data transit via the buffer.  In practice either should work, but when using `dd` or any duplication program, it is apparently best practice to use `r`.

4. `diskutil partitionDisk disk99 GPT JHFS+ "New Disk Name" 0g`
    + partition the disk using journaled HFS+ and name it whatever is in the quotes.  APFS filesystem only has a purpose on an SSD; traditional hard drives do not benefit from it.  You can format it in whatever format desired, however.
