### <a name="Partitions"></a>Partitions

The D-Link DCS-8000LH partitions are:
```
# cat /proc/mtd 
dev:    size   erasesize  name
mtd0: 00040000 00010000 "boot"
mtd1: 00020000 00010000 "pib"
mtd2: 00100000 00010000 "userdata"
mtd3: 00020000 00010000 "db"
mtd4: 00020000 00010000 "log"
mtd5: 00020000 00010000 "dbbackup"
mtd6: 00020000 00010000 "logbackup"
mtd7: 00300000 00010000 "kernel"
mtd8: 00b00000 00010000 "rootfs"
```
Or as seen by the driver with start and end addresses:

```
9 cmdlinepart partitions found on MTD device m25p80
Creating 9 MTD partitions on "m25p80":
0x000000000000-0x000000040000 : "boot"
0x000000040000-0x000000060000 : "pib"
0x000000060000-0x000000160000 : "userdata"
0x000000160000-0x000000180000 : "db"
0x000000180000-0x0000001a0000 : "log"
0x0000001a0000-0x0000001c0000 : "dbbackup"
0x0000001c0000-0x0000001e0000 : "logbackup"
0x0000001e0000-0x0000004e0000 : "kernel"
0x0000004e0000-0x000000fe0000 : "rootfs"
```

Partition usage:

 | number | name        | start    | end      | size     | fstype   | contents          |
 | ------ | ----------- | -------- | -------- | -------- | -------- | ---------------   |
 | 0      | "boot"      | 0x000000 | 0x040000 | 0x40000  | boot     | U-Boot            |
 | 1      | "pib"       | 0x040000 | 0x060000 | 0x20000  | raw      | device info       |
 | 2      | "userdata"  | 0x060000 | 0x160000 | 0x100000 | squashfs | mydlink (/opt)    |
 | 3      | "db"        | 0x160000 | 0x180000 | 0x20000  | tar.gz   | non-volatile data |
 | 4      | "log"       | 0x180000 | 0x1a0000 | 0x20000  | raw?     | empty             |
 | 5      | "dbbackup"  | 0x1a0000 | 0x1c0000 | 0x20000  | tar.gz   | copy of "db"      |
 | 6      | "logbackup" | 0x1c0000 | 0x1e0000 | 0x20000  | raw?     | empty             |
 | 7      | "kernel"    | 0x1e0000 | 0x4e0000 | 0x300000 | uImage   | Linux 3.10        |
 | 8      | "rootfs"    | 0x4e0000 | 0xfe0000 | 0xb00000 | squashfs | rootfs (/)        |


The D-Link firmware updates I have looked at will replace the
"userdata", "kernel" and "rootfs" partitions, but leave other
partitions unchanged. I imagine that the "boot" partition might be
upgraded too if deemed necessary by D-Link. But it was not touched
when going from 2.01.03 to 2.02.02.

The "log" and "logbackup" appear to be currently unused.  But I am
reluctant trusting this, given their names.  I guess they could be
cleaned and overwritten anytime.  They are too small to be very useful
anyway.  You can't put any writable file system om them with only two
erase blocks.