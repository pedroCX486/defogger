### Why can we run the NIPCA webserver before we modify the firmware?

D-Link left all the webserver parts in the firmware, including all the
NIPCA CGI tools. The only change they made was disabling the startup
script.

The webserver can be enabled and started manually from the shell by
running:

```
tdb set HTTPServer Enable_byte=1
/etc/rc.d/init.d/extra_lighttpd.sh start
```

This is precisely what our Bluetooth tool does when it is called with
the **--lighttpd** option.

The `HTTPServer Enable_byte` is persistent, so setting is only
necessary once. Unless you do a factory reset.


### What's the problem with the RTSP server in the unmodified firmware?

The original D-Link firmware is already running **rtspd**, but it is only
listening on the loopback address 127.0.0.1.  It is probably intended
as a backend server for the **mydlink** services.

We can make rtspd listen on all addresses by clearing the **RTPServer
RejectExtIP** setting.  Both rtspd and the firewall need a restart for
this to have an effect.  Enabling **RTPServer Authenticate** is
probably a good idea when doing this, to prevent the camera from
streaming to anyone who can connect.

```
tdb set RTPServer RejectExtIP_byte=0
tdb set RTPServer Authenticate_byte=1
/etc/rc.d/init.d/firewall.sh reload
/etc/rc.d/init.d/rtspd.sh restart
```

These settings are persistent as usual, so they only need to be
modified after factory resets. Changing the settings and then
rebooting the camera will therefore enable remote RTSP access, since
both services are running by default in the D-Link firmware.


### Can I unpack the "userdata" file system?

The [**userdata**](#Partitions) you backed up as **mtd2** contains a xz compressed
squasfs file system, with most of the mydlink cloud tools. The file
system can be unpacked on a Linux system using unsquashfs:
```
$ unsquashfs mtd2
Parallel unsquashfs: Using 4 processors
15 inodes (22 blocks) to write

[=============================================================================================================================================================================================================|] 22/22 100%

created 12 files
created 1 directories
created 3 symlinks
created 0 devices
created 0 fifos
$ ls -la squashfs-root/
total 1156
drwxr-xr-x  2 bjorn bjorn    340 Feb 14 10:58 .
drwxrwxrwt 41 root  root    2280 May 13 15:13 ..
-rwxr-xr-x  1 bjorn bjorn  13184 Feb 14 10:58 ca-refresh
-rwxr-xr-x  1 bjorn bjorn 273692 Feb 14 10:58 cda
lrwxrwxrwx  1 bjorn bjorn      9 May 13 15:13 cert -> /tmp/cert
-rwxr-xr-x  1 bjorn bjorn   5991 Feb 14 10:58 client-ca.crt.pem
lrwxrwxrwx  1 bjorn bjorn      7 May 13 15:13 config -> /tmp/db
-rwxr-xr-x  1 bjorn bjorn 436428 Feb 14 10:58 da_adaptor
-rwxr-xr-x  1 bjorn bjorn      4 Feb 14 10:58 dcp_version
-rwxr-xr-x  1 bjorn bjorn    814 Feb 14 10:58 device.cfg
lrwxrwxrwx  1 bjorn bjorn     17 May 13 15:13 lib -> /var/libevent/lib
-rwxr-xr-x  1 bjorn bjorn      5 Feb 14 10:58 m2m
-rwxr-xr-x  1 bjorn bjorn   6220 Feb 14 10:58 mydlink_watchdog.sh
-rwxr-xr-x  1 bjorn bjorn   1034 Feb 14 10:58 opt.local
-rwxr-xr-x  1 bjorn bjorn 171828 Feb 14 10:58 sa
-rwxr-xr-x  1 bjorn bjorn 242028 Feb 14 10:58 strmsvr
-rwxr-xr-x  1 bjorn bjorn     10 Feb 14 10:58 version
```

The primary entry point here is the **opt.local** init-script.  This
is also the only required file.  The **version** file is read by the
Bluetooth API, and reported as the mydlink version, which makes it
useful for verifying a modified camera.  Our alternate
[**userdata**](#Partitions) file system contains only these two
files. But one could imagine including a number of other useful tools,
like tcpdump, a ssh server etc.

It is also possible to keep all the D-Link files, if that's
wanted. The original **opt.local** script can be modified to leave
mydlink support running while still starting other features.  We could
even add our own non-volatile setting to choose one or the other, or
both, and making it a configuration thing. Fantasy is the only
limiting factor.

Repacking the files into a camera compatible squashfs file system:
```
mksquashfs squashfs-root mtd2.new -all-root -comp xz
```

Note that **xz** compression is required.  No other compression is
supported AFAIK.

There are simpler ways to write the new file system to the camera than
creating a firmware update package, if you just want to test it. One
example:

```
tftp -r mtd2.new -l /tmp/mtd2.new -g 192.168.2.1
cat /tmp/mtd2.new >/dev/mtdblock2 
```

But DON'T do that unless you both have a backup and know what you are
doing...

You should reboot the camera after doing this, unless you make sure
you stop any process running from the previous /opt system and remount
it properly.