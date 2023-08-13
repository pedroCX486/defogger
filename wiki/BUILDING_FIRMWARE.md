#### <a name="BuildFirmware"></a>Bulding the example firmware update in this repo

Rebuilding the example is as easy as just typing **make**. 
The Makefile is a noisy one, so you can see all that's going on:
```
$ make
echo "WARNING: keys/DCS-8000LH-sign.pem is missing - using a new abitrary key instead"
WARNING: keys/DCS-8000LH-sign.pem is missing - using a new abitrary key instead
[ -f random-signkey.pem ] || openssl genrsa -out random-signkey.pem
Generating RSA private key, 2048 bit long modulus (2 primes)
..............................................................................................+++++
e is 65537 (0x010001)
openssl rsa -pubout -in random-signkey.pem -out verify.key
writing RSA key
echo "Publisher:DMdssdFW1" >certificate.info
echo "Supported Models:DCS-8000LH,DCS-8000LH" >>certificate.info
echo "Firmware Version:1.0.0" >>certificate.info
echo "Target:update.bin" >>certificate.info
echo "Build No:9999" >>certificate.info
echo "Contents:update" >>certificate.info
openssl rand 16 > aes.key
openssl rsautl -encrypt -in aes.key -inkey keys/DCS-8000LH-PriKey.pem -out aes.key.rsa
sed -ne 's/"//g' -e 's/^VERSION *= *//p' dcs8000lh-configure.py >version
mksquashfs version opt.local opt.squashfs -all-root -comp xz
Parallel mksquashfs: Using 4 processors
Creating 4.0 filesystem on opt.squashfs, block size 131072.
[==============================================================================|] 2/2 100%

Exportable Squashfs 4.0 filesystem, xz compressed, data block size 131072
        compressed data, compressed metadata, compressed fragments, compressed xattrs
        duplicates are removed
Filesystem size 1.08 Kbytes (0.00 Mbytes)
        60.69% of uncompressed filesystem size (1.79 Kbytes)
Inode table size 98 bytes (0.10 Kbytes)
        100.00% of uncompressed inode table size (98 bytes)
Directory table size 46 bytes (0.04 Kbytes)
        100.00% of uncompressed directory table size (46 bytes)
Number of duplicate files found 0
Number of inodes 3
Number of files 2
Number of fragments 1
Number of symbolic links  0
Number of device nodes 0
Number of fifo nodes 0
Number of socket nodes 0
Number of directories 1
Number of ids (unique uids + gids) 1
Number of uids 1
        root (0)
Number of gids 1
        root (0)
openssl aes-128-cbc -md md5 -kfile aes.key -nosalt -e -out update.aes -in opt.squashfs
*** WARNING : deprecated key derivation used.
Using -iter or -pbkdf2 would be better.
*** WARNING : deprecated key derivation used.
Using -iter or -pbkdf2 would be better.
sed  -e "s/@@MODEL@@/\"DCS-8000LH\"/" -e "s/@@MD5SUM@@/\"f1a1d3952c1630e5adb53e7f93b59d5e\"/" -e "s/@@VERSION@@/\"1.0.0-9999\"/" update.sh >update.bin
openssl aes-128-cbc -md md5 -kfile aes.key -nosalt -e -out update.bin.aes -in update.bin
*** WARNING : deprecated key derivation used.
Using -iter or -pbkdf2 would be better.
openssl dgst -sha1 update.aes | cut -d' ' -f2 > update.sha1
cat update.bin.aes aes.key.rsa certificate.info update.sha1 | openssl dgst -sha1 | cut -d' ' -f2 > sign.sha1
openssl rsautl -sign -inkey random-signkey.pem -out sign.sha1.rsa -in sign.sha1
tar cvf fw.tar certificate.info aes.key.rsa sign.sha1.rsa update.aes update.bin.aes verify.key
certificate.info
aes.key.rsa
sign.sha1.rsa
update.aes
update.bin.aes
verify.key

```

This will produce a new **fw.tar** firmware update image in the root directory.