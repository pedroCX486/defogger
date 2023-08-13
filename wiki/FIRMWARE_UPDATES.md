### Firmware updates

There are at least two shell scripts providing a firmware update
service in the D-Link firmware:

 * /var/www/config/firmwareupgrade.cgi
 * /sbin/fwupdate

They are both pretty similar and obviously come from the same source.
The main difference is that firmwareupgrade.cgi provides the NIPCA
firmwareupgrade service, while fwupdate is a command line tool.

The web service is most interesting for us, providing both the upload
and upgrade in one simple tool.  The fwupdate tool is used by the
mydlink cloud tool **da_adaptor** , via an fw_upgrade symlink.


#### Signed and encrypted

Looking at the contents of a firmware update from D-Link can be
demotivating at the beginning:

```
$ tar xvf DCS-8000LH_Ax_v2.02.02_3014.bin 
update.bin.aes
update.aes
aes.key.rsa
certificate.info
sign.sha1.rsa

$ file *
aes.key.rsa:      data
certificate.info: ASCII text
sign.sha1.rsa:    data
update.aes:       data
update.bin.aes:   data

$ ls -l
total 10956
-rw-r--r-- 1 bjorn bjorn      128 Feb 14 10:58 aes.key.rsa
-rw-r--r-- 1 bjorn bjorn      130 Feb 14 10:58 certificate.info
-rw-r--r-- 1 bjorn bjorn      128 Feb 14 10:58 sign.sha1.rsa
-rw-r--r-- 1 bjorn bjorn 10268368 Feb 14 10:58 update.aes
-rw-r--r-- 1 bjorn bjorn   936464 Feb 14 10:58 update.bin.aes
```

So all the interesting stuff is AES encrypted, and the AES key is RSA
encrypted.  The only directly readable file is this one, and it
doesn't tell us much:

```
$ cat certificate.info 
Publisher:DMdssdFW1
Supported Models:DCS-8000LH,DCS-8000LH
Firmware Version:1.0.0
Target:update.bin
Build No:3014
Contents:update
```

Not much we can do about this then.  Or so it seems...  Until we look
at **firmwareupgrade.cgi**, or **fwupdate** which has almost the same
code:

```sh
verifyFirmware() {
	result=uploadSign
	#tar tf "$UPLOADBIN" > /dev/null 2> /dev/null || return 1
	fw_sign_verify.sh "$UPLOADBIN" /etc/db/verify.key > /dev/null 2> /dev/null || return 1
	return 0
}

decryptFirmware() {
	result=uploadDecrypt
	pibinfo PriKey > $dir/decrypt.key 2> /dev/null
	fw_decrypt.sh $dir/decrypt.key $out > /dev/null 2> /dev/null || return 1
	return 0
}
```

Can it be that simple?  Yes, it is.

Looking further at the **fw_sign_verify.sh** and **fw_decrypt.sh**,
used by both update tools, confirms it.  The firmware is verified by
using the RSA public key in **/etc/db/verify.key** to decrypt the hash
in **sign.sha1.rsa**.  Then it is decrypted using a key from the
factory data **pib** partition.



#### Further unpacking the firmware update

So we have the keys and the hashing algorithms we need to both verify
and decrypt this firmware.  We can run the commands found in
**fw_decrypt.sh** to get the real contents (slightly adapted to modern
openssl versions):

```
$ openssl rsautl -decrypt -in aes.key.rsa -inkey decrypt.key -out aes.key

$ openssl aes-128-cbc -v -md md5 -kfile aes.key -nosalt -d -in update.bin.aes -out update.bin
bufsize=8192
*** WARNING : deprecated key derivation used.
Using -iter or -pbkdf2 would be better.
bytes read   :   936464
bytes written:   936454

$ openssl aes-128-cbc -v -md md5 -kfile aes.key -nosalt -d -in update.aes -out update
bufsize=8192
*** WARNING : deprecated key derivation used.
Using -iter or -pbkdf2 would be better.
bytes read   : 10268368
bytes written: 10268355

$ file update.bin update
update.bin: POSIX shell script, ASCII text executable
update:     data
```

OK, the **update** file is still in an unknown format, but at least
we have the tool used to write it to the system. And it is a shell
script, so we have the source to look at too!  But 936454 bytes is a
hell of a shell script, and this is of course because most of it is an
uuencoded binary.  So we don't know exactly what that does.  But it is
named ddPack so a fair guess is that it is a tool for dd'ing multiple
file systems or other images packed as a single file.  That's really
enough info.

binwalk shows that the **update** file is just two squashfs systems
and a kernel, with a 1024 header of some sort.  The header presumably
tells ddPack how it should apply these three images:

```
$ binwalk update

DECIMAL       HEXADECIMAL     DESCRIPTION
--------------------------------------------------------------------------------
1024          0x400           Squashfs filesystem, little endian, version 4.0, compression:xz, size: 338755 bytes, 16 inodes, blocksize: 131072 bytes, created: 2019-02-14 09:58:28
340992        0x53400         uImage header, header size: 64 bytes, header CRC: 0x675F081D, created: 2019-02-14 09:31:53, image size: 1661571 bytes, Data Address: 0x804D4960, Entry Point: 0x804D4960, data CRC: 0x73083021, OS: Linux, CPU: MIPS, image type: OS Kernel Image, compression type: none, image name: "linux_3.10"
2002627       0x1E8EC3        Squashfs filesystem, little endian, version 4.0, compression:xz, size: 8265620 bytes, 2145 inodes, blocksize: 131072 bytes, created: 2019-02-14 09:58:45
```

But we can easily guess that without knowing anything about the
header.  There is only one alternative:
 * The kernel goes into the **kernel** partition
 * The 8265620 bytes squasfs system goes into the **rootfs** partition
 * The remaining squasfs system goes into the **userdata** partition

So there is no need to analyze ddPack.  We have the necessary entry
points for **fwupdate** or **firmwareupgrade.cgi** in the
**update.bin** script, and that's what we needed to know for the next
step:


#### Creating our own firmware updates

We do have shell access, so we can simply write the file systems we
want to flash as shown earlier.  We don't need to use the D-Link
scripts.  But where's the fun in that?

There is one challenge here: The D-Link tools are expecting signed and
encrypted firmware updates.  They will run their verifyFirmware() and
decryptFirmware() functions, and fail the update if any of the returns
an error.

But bailing out on verification errors is only the default setting, as
illustrated by this code from **fwupdate** (there is code with similar
functionality in **firmwareupgrade.cgi**):


```sh
        TrustLevel=`tdb get SecureFW _TrustLevel_byte`
        verifyFirmware
        ret=$?
        case $ret in
                2)
                        sign="not_signed"
                ;;
                0)
                        sign="trust"
                ;;
                *)
                        sign="untrust"
                ;;
        esac    
        if [ "$do_up" = "1" -a "$ret" != "0" -a "$TrustLevel" = "1" ]; then
                echo "3"
                return 1
        fi
```

So we don't need to sign the firmware if we change the **SecureFW
_TrustLevel** setting.  Or we can even sign it with a key unknown to
the camera if we like.  Which can be useful if we ever replace the
[**rootfs**]](#Partitions), since it will allow us to install our own verification
key and use it with D-Links tools.

But what about the encryption?  This cannot be disabled.  This gets
even better: The decrypting key so graciously provided to us in the
[**pib**](PARTITIONS.md) partition is an RSA private key. So not only can we decrypt
the firmware with it, but we can also encrypt! Nice.


The [**Makefile**](../Makefile) in this repo has examples of how to use this to
create firmware update images which are accepted by the **fwupdate**
and **firmwareupgrade.cgi** tools.  It uses an alternative
[**update.bin**](../update.sh) made to modify only the [**userdata**](PARTITIONS.md) partition.  This
way we can install our own code in the camera, but still leave the
D-Link camera OS unmodified.