### Restoring original D-Link firmware

Why you would do something this dumb after D-Link shutdown 
the cloud service is beyond me, but...

The D-Link firmware, including the mydlink tools in the
[**userdata**](#Partitions) partition, can be restored by doing a
manual firmware upgrade providing a firmware update from D-Link.  Real
example, going back to v2.02.02 (replace $CAMERA_PIN with the camera's pin
if you haven't set the environment variable):

```bash
curl --http1.0 -u admin:$CAMERA_PIN --form upload=@DCS-8000LH_Ax_v2.02.02_3014.bin http://192.168.2.37/config/firmwareupgrade.cgi
```

Example output:
```
curl: (52) Empty reply from server
```

I don't know why I got that **Empty reply** warning instead of the
expected **upgrade=ok**, but update went fine so I guess it can safely
be ignored. Might be a side effect of rewriting the root file system,
which the firmwareupgrade.cgi script is running from.