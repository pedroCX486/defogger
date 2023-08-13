### <a name="Backup"></a>Backup

Create a backup of everything *before* you mess up.  Restoring will be
hard anyway, so don't rely on that.  But you can forget about
restoring at all unless you have a backup, so make it anyway.

Note that the [**pib**](PARTITIONS.md) partition contains data which are
specific to **your** camera, and cannot be restored from any other
source!  This includes
 * model number
 * hardware revision
 * mac address
 * feature bits
 * private keys, pincode and passwords

Well, OK, we can restore most of the [**pib**](PARTITIONS.md)  using information from
the [camera label](https://www.mork.no/~bjorn/dcs8000lh/dcs8000lh-label.jpg), but
it's better to avoid having to do that...

A backup is also useful for analyzing the file systems offline.

Making a backup without networking is inconvenient, so setup
networking first. In theory, you could dump the flash to the serial
console. But this would be very time consuming and tiresome.

The D-Link firmware provides a selection of network file transfer
tools. Pick anyone you like:
 * tftp
 * wget
 * curl
 * ...and probably more

I've been using tftp for my backups because it is simple. You'll
obviously need a tftp server for this. Google for instructions on
setting that up.  You could alternatively set up a web server and use
wget or curl to post the files there, but this is more complx to set
up IMHO.

Here is one example of how to enable temporary telnet access and
copying all camera flash partitions to a tftp server (replace 
$CAMERA_PIN with the camera's pin if you haven't set the
environment variable):


```bash
./dcs8000lh-configure.py B0:C5:54:AA:BB:CC $CAMERA_PIN --telnetd
```

Example output:
```
Connecting to B0:C5:54:AA:BB:CC...
Verifying IPCam service
Connected to 'DCS-8000LH-BBCC'
Adding the 'admin' user as an alias for 'root'
Attempting to run 'grep -Eq ^admin: /etc/passwd||echo admin:x:0:0::/:/bin/sh >>/etc/passwd' on DCS-8000LH-BBCC by abusing the 'set admin password' request
Setting the 'admin' user password to '$CAMERA_PIN'
Attempting to run 'grep -Eq ^admin:x: /etc/passwd&&echo admin:$CAMERA_PIN|chpasswd' on DCS-8000LH-BBCC by abusing the 'set admin password' request
Starting telnetd
Attempting to run 'pidof telnetd||telnetd' on DCS-8000LH-BBCC by abusing the 'set admin password' request
 

Attempting to run '[ $(tdb get HTTPServer Enable_byte) -eq 1 ] || tdb set HTTPServer Enable_byte=1' on DCS-8000LH-BBCC by abusing the 'set admin password' request
Attempting to run '/etc/rc.d/init.d/extra_lighttpd.sh start' on DCS-8000LH-BBCC by abusing the 'set admin password' request
Done.


$ telnet 192.168.2.37
Trying 192.168.2.37...
Connected to 192.168.2.37.
Escape character is '^]'.
localhost login: admin
Password: 


BusyBox v1.22.1 (2019-02-14 17:06:35 CST) built-in shell (ash)
Enter 'help' for a list of built-in commands.


# for i in 0 1 2 3 4 5 6 7 8; do tftp -l /dev/mtd${i}ro -r mtd$i -p 192.168.2.1; done`
```

Change 192.168.2.37 to the address of your camera and 192.168.2.1 to
the address of your tftp server. Note that most tftp servers require
existing and writable destination files. Refer to your tftp server docs
for details.

### Backing up dynamic data

This is not necessary for system operation as any non-volatile data is
saved in the [**db**](PARTITIONS.md) partition anyway.  But it can still be useful to
have a copy of the system state for offline studying, so I also like
to save a working copy of /tmp:

``` bash
tar zcvf /tmp/tmp.tgz /tmp/
```

``` bash
tftp -l /tmp/tmp.tgz -r tmp.tgz -p 192.168.2.1
```