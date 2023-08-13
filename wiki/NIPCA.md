### <a name="NIPCA"></a>Using NIPCA to manage the camera

The local web server provides a direct camera management API, but not
a web GUI application. All API requests require authentication. We
have added a single admin user, using the pincode from the camera
label as passord.  More users can be adding if necessary, even by
using the API itself.

Read the NIPCA reference spec for usage, or look at the script names
under **/var/www** in the [**rootfs**](#Partitions) and simply try
them out. Most API endpoints return a list of current settings. Some
of the settings can be set by GET requests by providing the new values
as URL parameters.

A few NIPCA references of different age:
 * http://gurau-audibert.hd.free.fr/josdblog/wp-content/uploads/2013/09/CGI_2121.pdf
 * https://docplayer.net/33354138-Network-ip-camera-application-programming-interface-nipca.html
 * ftp://ftp.dlink.net.pl/dcs/dcs-2132L/documentation/DCS-2132L_NIPCA_support table_1-9-5_20131211.pdf
 * https://www.airlivecam.eu/data/IP%20Camera%20Open%20API.doc

Google for more. Be aware that a most of these settings depend on the
hardware.  There is obviously no point in trying to manage an SD card
slot of the DCS-8000LH...

A few of examples, using curl to read and set configuration variables 
(replace $CAMERA_PIN with the camera's pin if you haven't set the
environment variable):

```bash
curl -u admin:$CAMERA_PIN http://192.168.2.37/common/info.cgi
```

Example output:
```
model=DCS-8000LH
product=Wireless Internet Camera
brand=D-Link
version=2.02
build=02
hw_version=A
nipca=1.9.7
name=DCS-8000LH
location=
macaddr=B0:C5:54:AA:BB:CC
ipaddr=192.168.2.37
netmask=255.255.255.0
gateway=192.168.2.1
wireless=yes
inputs=0
outputs=0
speaker=no
videoout=no
pir=no
icr=yes
ir=yes
mic=yes
led=no
td=no
playing_music=no
whitelightled=no
```

```bash
curl -u admin:$CAMERA_PIN 'http://192.168.2.37/config/datetime.cgi' 
```

Example output:
```
method=1
timeserver=ntp1.dlink.com
timezone=1
utcdate=2019-05-09
utctime=13:25:14
date=2019-05-09
time=15:25:14
dstenable=yes
dstauto=yes
offset=01:00
starttime=3.2.0/02:00:00
stoptime=11.1.0/02:00:00
```

```bash
curl -u admin:$CAMERA_PIN http://192.168.2.37/config/led.cgi?led=off
```

Example output:
```
led=off
```

Most camera settings can be controlled using this API and e.g curl for
the command line.  There are also packages implementing API clients,
like for example this nodejs one: https://www.npmjs.com/package/nipca