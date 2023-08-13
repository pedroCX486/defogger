### U-Boot

My DCS-8000LH came with this boot loader:

`U-Boot 2014.01-rc2-V1.1 (Jun 06 2018 - 03:44:37)`

But it is patched/configured to require a password for access to the
U-Boot prompt. Fortunately, D-Link makes the password readily
available in their GPL package. It is found in the file
`DCS-8000LH-GPL/configs/gpl_defconfig`:

`ALPHA_FEATURES_UBOOT_LOGIN_PASSWORD="alpha168"`

Quickly enter the **alpha168** password when you see:

`Press ESC to abort autoboot in 3 seconds`

And you'll get a `rlxboot#` prompt, with access to these U-Boot commands:

```
?       - alias for 'help'
base    - print or set address offset
bootm   - boot application image from memory
bootp   - boot image via network using BOOTP/TFTP protocol
cmp     - memory compare
coninfo - print console devices and information
cp      - memory copy
crc32   - checksum calculation
echo    - echo args to console
editenv - edit environment variable
efuse   - efuse readall | read addr
env     - environment handling commands
fephy   - fephy read/write
go      - start application at address 'addr'
help    - print command description/usage
imxtract- extract a part of a multi-image
loadb   - load binary file over serial line (kermit mode)
loadx   - load binary file over serial line (xmodem mode)
loady   - load binary file over serial line (ymodem mode)
loop    - infinite loop on address range
md      - memory display
mm      - memory modify (auto-incrementing address)
mw      - memory write (fill)
nm      - memory modify (constant address)
ping    - send ICMP ECHO_REQUEST to network host
printenv- print environment variables
reset   - Perform RESET of the CPU
setenv  - set environment variables
setethaddr- set eth address
setipaddr- set ip address
sf      - SPI flash sub-system
source  - run script from memory
tftpboot- boot image via network using TFTP protocol
tftpput - TFTP put command, for uploading files to a server
tftpsrv - act as a TFTP server and boot the first received file
update  - update image
version - print monitor, compiler and linker version
```

Using the boot loader for image manipulation will be hard though,
since the camera has no ethernet, USB or removable flash and the boot
loader has no WiFi driver.  It is probably possible to load an image
over serial, but I don't have the patience for that...

The environment is fixed and pretty clean:
```
rlxboot# printenv
=3
addmisc=setenv bootargs ${bootargs}console=ttyS0,${baudrate}panic=1
baudrate=57600
bootaddr=(0xBC000000 + 0x1e0000)
bootargs=console=ttyS1,57600 root=/dev/mtdblock8 rts_hconf.hconf_mtd_idx=0 mtdparts=m25p80:256k(boot),128k(pib),1024k(userdata),128k(db),128k(log),128k(dbbackup),128k(logbackup),3072k(kernel),11264k(rootfs)
bootcmd=bootm 0xbc1e0000
bootfile=/vmlinux.img
ethact=r8168#0
ethaddr=00:00:00:00:00:00
load=tftp 80500000 ${u-boot}
loadaddr=0x82000000
stderr=serial
stdin=serial
stdout=serial

Environment size: 533/131068 bytes
```

So we can get ourselves a root shell, type in order:

```bash
setenv bootargs ${bootargs} init=/bin/sh
```

```bash
${bootcmd}
```

Nothing is mounted or started since /sbin/init is skipped altogether
in this case.  Not even /sys and /proc.  We can emulate a semi-normal
system by running as the first command:

`/etc/rc.d/rcS` 

And then run for example:

`telnetd -l /bin/sh`

To enable temporary passwordless telnet into the camera instead of/in
addition to the serial console. This is futile unless you have
networking of course.  Use the much simpler Bluetooth procedure described above. 
Or the "mydlink" app if you prefer to establish a network connection
to your camera.

Then run the following commands:

`grep -Eq ^admin: /etc/passwd || echo admin:x:0:0::/:/bin/sh >>/etc/passwd`

`grep -Eq ^admin:x: /etc/passwd && echo "admin:$(pibinfo Pincode)" | chpasswd`

`tdb set SecureFW _TrustLevel_byte=0`

`tdb set HTTPServer Enable_byte=1`

`tdb set HTTPAccount AdminPasswd_ss="$(pibinfo Pincode)"`

`/etc/rc.d/init.d/extra_lighttpd.sh start`

Then on the local machine, run (replace CAMERA_PIN with the camera's pin if you haven't set the
environment variable):

`curl --http1.0 -u admin:$CAMERA_PIN --form upload=@DCS-8000LH_Ax_v2.02.02_3014.bin http://CAM.IP/config/firmwareupgrade.cgi`

It should be returned:

`curl: (52) Empty reply from server`

With this the camera will downgrade the firmware to 2.02.02. 

Then reboot the camera and using serial to re-enable lighttpd server on your camera, run the following command on your local machine:

`$ curl --http1.0 -u admin:CAMPIN --form upload=@fw.tar http://CAM.IP/config/firmwareupgrade.cgi`

About 1 min later, the camera will reboot.
After the reboot process, you will have a cam with all the goodies mentioned above with fw 2.02.02.

Note that the commands above don't setup the required wifi connection for you to access the camera remotely. You can try to read the [bluetooth script](../dcs8000lh-configure.py) to understand how to set up wifi, or use a Linux guide on how to connect to wifi using the terminal.