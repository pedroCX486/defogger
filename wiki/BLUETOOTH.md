### Bluetooth LE GATT API

The Bluetooth service is in a "locked" mode by default. This is
controlled by the "Ble Mode" persistent setting stored in the **db**
partition. If true ("1"), then most of the Bluetooth commands are
rejected.  But changing the setting manually will not help much, since
the system automatically enter lock mode 180 seconds after the last
Bluetooth client disconnected.

The challenge -> response unlock method described below is much more
useful.

#### Converting the PIN Code to a Bluetooth unlock key

Most Bluetooth commands are rejected when locked.  Access to the full
Bluetooth API can be unlocked by using the PIN Code printed on the
camera label.  This code is not sent directly over the air
though. Instead it is combined with a random challenge.

Both the random challenge and the matching key are generated by the
application `sbin/gen_bt_config` on the camera side.  The key is
calculated by taking the first 16 bytes of the base64 encoded md5
digest of

 * model string + '-'  four last mac digits (or Bluetooth device name?)
 * PIN Code
 * challenge.

Note that this application depends on bluetooth libraries, which are
not in /lib. So we have to set LD\_LIBRARY\_PATH to run it manually:

```bash
LD_LIBRARY_PATH=/var/bluetooth/lib sbin/gen_bt_config update_key_only
```

Example output:
```
In main:182: modelStr = 'DCS-8000LH'
In main:183: mac = 'b0:c5:54:ab:cd:ef'
In update_ble_key:87: key data = 'DCS-8000LH-CDEF012345b2gaescrbldchnik'
```

I've slightly obfuscated my data here - the pincode in the above case
is `012345`, and the dynamically generated challenge is
`b2gaescrbldchnik`. The generated challenge and key are stored in
`/tmp/db/db.xml` and can be read directly from there:

```bash
grep Key /tmp/db/db.xml |tail -2
```

Example output:
```
<ChallengeKey type="3" content="b2gaescrbldchnik" />
<Key type="5" content="jrtY6nONQ5rV+2Ph" />
```

Or you can read them using the same tools the Bluetooth system uses:
``` bash
tdb get Ble ChallengeKey_ss
```

Example output:
```
b2gaescrbldchnik
```

```bash 
mdb get ble_key
```

Example output:
```
jrtY6nONQ5rV+2Ph
```

Yes, the D-Link code does actually use tdb for the first one and mdb
for the second.  I have absolutely no idea why,... It is possible to
read the key using tdb too:

```bash
tdb get Ble Key_ss
```

Example output:
```
jrtY6nONQ5rV+2Ph
```

Generating the same key by hand on a Linux system is simple:

```bash
echo -n 'DCS-8000LH-CDEF012345b2gaescrbldchnik' | md5sum | xxd -r -p | base64 | cut -c-16
```

Example output:
```
jrtY6nONQ5rV+2Ph
```

#### Characteristic UUIDs

D-Link is using the GATT BlueZ example plugin, patching it to add
their camera specific endpoints.  This means that we can find all the
API "documentation" in the
`DCS-8000LH-GPL/package/bluez_utils/feature-patch/5.28/customized-mydlink.patch`
file in the GPL archive.

This defines a number of 16bit UUIDs with mostly nonsense names:
```
+#define IPCAM_UUID		0xD001
+#define A000_UUID		0xA000
+#define A001_UUID		0xA001
+#define A100_UUID		0xA100
+#define A101_UUID		0xA101
+#define A102_UUID		0xA102
+#define A103_UUID		0xA103
+#define A104_UUID		0xA104
+#define A200_UUID		0xA200
+#define A201_UUID		0xA201
+#define A300_UUID		0xA300
+#define A301_UUID		0xA301
+#define A302_UUID		0xA302
+#define A303_UUID		0xA303
+#define A304_UUID		0xA304
```


`IPCAM_UUID` is registered as the `GATT_PRIM_SVC_UUID`, which means
that it shows up as a primary GATT service we can look for when
looking for a supported camera.

The rest of the UUIDs are characteristics of this primary service. The
API is based on reading or writing these characteristics.


#### Data formatting

Both input and output parameters are sent as ascii strings using
key=value pairs joined by `;`, with an exception for the nested KV
pairs in the WiFi survey results.  All keys are single upper case
characters. Key names are somewhat reused, so the exact meaning depend
on the characteristic.

Values are either integers, including boolean 0/1, or some set of
ascii text.

Three real examples, read from 0xA001, 0xA200 and 0xA104:
```
M=1;C=b2gaescrbldchnik
N=DCS-8000LH;P=1;T=1557349762;Z=CET-1CEST,M3.5.0,M10.5.0/3;F=2.01.03;H=A1;M=B0C554ABCDEF;V=3.0.0-b71
I=192.168.2.37;N=255.255.255.0;G=192.168.2.1;D=148.122.16.253
```

#### Listing characteristics


The **gattool** Linux command line tool is useful for exploring
Bluetooth LE devices.  You can look for primary services and list
associated characteristics of a service:
```
[B0:C5:54:AA:BB:CC][LE]> primary 
attr handle: 0x0001, end grp handle: 0x0008 uuid: 00001800-0000-1000-8000-00805f9b34fb
attr handle: 0x0010, end grp handle: 0x0010 uuid: 00001801-0000-1000-8000-00805f9b34fb
attr handle: 0x0011, end grp handle: 0x002e uuid: 0000d001-0000-1000-8000-00805f9b34fb
[B0:C5:54:AA:BB:CC][LE]> characteristics 0x0011
handle: 0x0012, char properties: 0x12, char value handle: 0x0013, uuid: 0000a000-0000-1000-8000-00805f9b34fb
handle: 0x0015, char properties: 0x0a, char value handle: 0x0016, uuid: 0000a001-0000-1000-8000-00805f9b34fb
handle: 0x0017, char properties: 0x02, char value handle: 0x0018, uuid: 0000a100-0000-1000-8000-00805f9b34fb
handle: 0x0019, char properties: 0x0a, char value handle: 0x001a, uuid: 0000a101-0000-1000-8000-00805f9b34fb
handle: 0x001b, char properties: 0x08, char value handle: 0x001c, uuid: 0000a102-0000-1000-8000-00805f9b34fb
handle: 0x001d, char properties: 0x02, char value handle: 0x001e, uuid: 0000a103-0000-1000-8000-00805f9b34fb
handle: 0x001f, char properties: 0x02, char value handle: 0x0020, uuid: 0000a104-0000-1000-8000-00805f9b34fb
handle: 0x0021, char properties: 0x0a, char value handle: 0x0022, uuid: 0000a200-0000-1000-8000-00805f9b34fb
handle: 0x0023, char properties: 0x08, char value handle: 0x0024, uuid: 0000a201-0000-1000-8000-00805f9b34fb
handle: 0x0025, char properties: 0x0a, char value handle: 0x0026, uuid: 0000a300-0000-1000-8000-00805f9b34fb
handle: 0x0027, char properties: 0x02, char value handle: 0x0028, uuid: 0000a301-0000-1000-8000-00805f9b34fb
handle: 0x0029, char properties: 0x08, char value handle: 0x002a, uuid: 0000a302-0000-1000-8000-00805f9b34fb
handle: 0x002b, char properties: 0x08, char value handle: 0x002c, uuid: 0000a303-0000-1000-8000-00805f9b34fb
handle: 0x002d, char properties: 0x02, char value handle: 0x002e, uuid: 0000a304-0000-1000-8000-00805f9b34fb
```

It is also possible to read and write characteristics using this tool,
but this can be a bit cumbersome unless you are fluent in ASCII coding
;-)