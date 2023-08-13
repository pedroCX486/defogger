#### RTSP

Direct RTSP access is also supported, using the same **admin** user.

The RTSP URLs are configurable, so the proper way to use RTSP is to
first check the URL of the wanted profile using the NIPCA API (replace 
$CAMERA_PIN with the camera's pin if you haven't set the
environment variable):

```bash
curl -u admin:$CAMERA_PIN --insecure 'https://192.168.2.37/config/rtspurl.cgi?profileid=1'
```

Example output:
```
profileid=1
urlentry=live/profile.0
video_codec=H264
audio_codec=OPUS
```

And then connect to this RTSP URL:

```
vlc rtsp://192.168.2.37/live/profile.0
```

Note that persistent RTSP access can be enabled with original
unmodified D-Link firmware, using the Bluetooth **--rtsp** option.
This modifies the necessary settings.  The **rtspd** service is
already started by default in the original firmware.

So there is no need to mess with the firmware at all if all you want
is RTSP.