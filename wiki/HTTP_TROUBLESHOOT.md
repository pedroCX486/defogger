#### <a name="Errors"></a>Errors during firmware update via HTTP

The **firmwareupgrade.cgi** script running in the camera isn't much
smarter than the rest of the system, so there are a few important
things keep in mind.  These are found by trial-and-error:

 * HTTP/1.1 might not work - the firmwareupgrade.cgi script does not support **100 Continue** AFAICS
 * The firmware update image should be provided as a **file** input field from a form
 * The field name must be **upload**.

Use the exact curl command provided above, replacing only the PIN
Code, IP address and firmware filename.  This should work.  Anything
else might not.

The camera must be manually rebooted by removing power or pressing
reset if the firmware upgrade fails for any reason. The
**firmwareupgrade.cgi** script stops most processes, inluding the
Bluetooth handler, and fails to restart them on errors. 

There will be no permanent harm if the upload fails.  But note that
you have to repeat the **--lighttpd** step after rebooting the camera,
before you can retry. It does not start automatically until we've
installed our modified "mydlink" alternative.

The contents of the fw.tar file must obviously be a valid, encrypted,
firmware update intended for the specified hardware.  It must also be
signed.  But the signing key can be unknown to the camera provided the
previous **--unsignedfw** request above was successful.

The [**Makefile**](../Makefile) provided here shows how to [build](BUILDING_FIRMWARE.md) a valid firmware
update, but for the DCS-8000LH only!  It does not support any other
model. It will create a new throwaway signing key if it canæt find a
real one, and include the associated public key in the archive in case
you want to verify the signature manually.

Note that the encryption key might be model specific.  I do not know
this as I have no other model to look at.  Please let me know if you
have any information on this topic.

The encryption key is part ot the [**pib**](PARTITIONS.md) partition, and can be
read from a shell using
```
pibinfo PriKey
```

Or you can simply look at your partition backup.  The key is stored as
a plain text *RSA PRIVATE KEY* PEM blob, so it is easy to spot. This
repo includes a copy of my [key](../keys/DCS-8000LH-PriKey.pem) as I see
no point in attempting to keep a well known shared key like this one
"secret".