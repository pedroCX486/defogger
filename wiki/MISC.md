#### The IPCam Characteristics

Guessed meanings of each characteristic, based on the source code and
some trial and error. Not necessarily how D-Link would describe them:


| UUID | op     | description     | format                                  | keys                                                                                                            |
| ---- | ------ | --------------- | --------------------------------------- | --------------------------------------------------------------------------------------------------------------- |
| A000 | read   | last status     | C=%d;A=%d;R=%d                          | C: uuid, A: mode, R: state                                                                                      |
| A000 | notify | last status     | C=%d;A=%d;R=%d                          | C: uuid, A: mode, R: state                                                                                      |
| A001 | read   | challenge       | M=%d;C=%s                               | M: opmode, C: challenge                                                                                         |
| A001 | write  | auth            | M=%d;K=%s                               | M: opmode, K: key                                                                                               |
| A100 | read   | wifi survey     | N=%d;P=%d;...                           |                                                                                                                 |
| A101 | read   | wifi config     | M=%s;I=%s;S=%s;E=%s                     | M: opmode, I: essid, S: 4 , E: 2                                                                                |
| A101 | write  | wifi config     | M=%s;I=%s;S=%s;E=%s;K=%s                | M: opmode, I: essid, S: 4 , E: 2, K: password                                                                   |
| A102 | write  | wifi connect    | C=%d                                    | C: connect (0/1)                                                                                                |
| A103 | read   | wifi status     | S=%d                                    | S: wifi link status (0,1,?)                                                                                     |
| A104 | read   | ip config       | I=%s;N=%s;G=%s;D=%s                     | I: address, N: netmask, G: gateway, D: DNS-server                                                               |
| A200 | read   | system info     | N=%s;P=%d;T=%d;Z=%s;F=%s;H=%s;M=%s;V=%s | N: devicename, P: haspin (0/1), T: time (unix epoch), Z: timezone, F: fwver, H: hwver, M: macaddr, V:mydlinkver |
| A200 | write  | name and time   | N=%s;T=%d;Z=%s                          | N: devicename, T: time (unix epoch), Z: timezone                                                                |
| A201 | write  | admin password  | P=%s;N=%s                               | P: current password, N: new password                                                                            |
| A300 | read   | reg state       | G=%d                                    | G: registration state (0/1)                                                                                     |
| A300 | write  | reg state       | G=%d                                    | G: registration state (0/1)                                                                                     |
| A301 | read   | provisioning    | N=%s;T=%s;U=%s                          | N: username, T: footprint, U: portal                                                                            |
| A302 | write  | restart mydlink | C=%d                                    | C: restart (0/1)                                                                                                |
| A303 | write  | register        | S=%s;M=%s                               | S: , M:  (written to /tmp/mydlink/reg_info, and then kill -USR1 `pidof da_adaptor`)                             |
| A304 | read   | register        | S=%d;E=%d                               | S: , E:  (cat /tmp/mydlink/reg_st)                                                                              |


The UUIDs from 0xA300 to 0xA304 are all related to the mydlink cloud
service, and therefore not of much use to us.  I haven't bothered
trying to figure out exactly how they are used.

We could in theory use the 0xA303 request which simply calls
**/opt/opt.local restart**.  But with the gaping 0xA201 hole,
allowing **any** command, there isn't much need for this one...

A few more details on the more complex characteristics:


##### A000

The only characteristic sent as notifications.  But it can also be
read directly for syncronous operations.

The value is the state to the last Bluetooth action:

	"C=%d;A=%d;R=%d", last_action_status.uuid, last_action_status.mode, last_action_status.state


##### A100

The wifi survey scan results are split in 128 byte "pages", where each
page starts with the total number of pages and the current page
number.  The characteristic value must be read as many times as the
given total.

For example, reading 3 pages:
```
[B0:C5:54:AA:BB:CC][LE]> char-read-hnd 0x0018
Characteristic value/descriptor: 4e 3d 33 3b 50 3d 31 3b 4c 3d 49 3d 41 6e 74 69 62 6f 6b 73 2c 4d 3d 30 2c 43 3d 36 2c 53 3d 34 2c 45 3d 32 2c 50 3d 36 32 26 4c 3d 49 3d 41 53 56 31 37 2c 4d 3d 30 2c 43 3d 31 31 2c 53 3d 34 2c 45 3d 32 2c 50 3d 34 36 26 4c 3d 49 3d 41 53 56 31 37 2d 64 6c 69 6e 6b 2c 4d 3d 30 2c 43 3d 36 2c 53 3d 34 2c 45 3d 32 2c 50 3d 36 38 26 4c 3d 49 3d 66 6a 6f 72 64 65 31 32 33 2c 4d 3d 30 
[B0:C5:54:AA:BB:CC][LE]> char-read-hnd 0x0018
Characteristic value/descriptor: 4e 3d 33 3b 50 3d 32 3b 2c 43 3d 31 2c 53 3d 34 2c 45 3d 32 2c 50 3d 35 38 26 4c 3d 49 3d 4a 4f 4a 2c 4d 3d 30 2c 43 3d 31 31 2c 53 3d 34 2c 45 3d 32 2c 50 3d 34 37 26 4c 3d 49 3d 4b 6a 65 6c 6c 65 72 62 6f 64 2c 4d 3d 30 2c 43 3d 36 2c 53 3d 34 2c 45 3d 32 2c 50 3d 36 32 26 4c 3d 49 3d 6d 67 6d 74 2c 4d 3d 30 2c 43 3d 31 2c 53 3d 34 2c 45 3d 32 2c 50 3d 37 34 26 4c 3d 49 3d 52 69 
[B0:C5:54:AA:BB:CC][LE]> char-read-hnd 0x0018
Characteristic value/descriptor: 4e 3d 33 3b 50 3d 33 3b 6e 64 65 64 61 6c 2c 4d 3d 30 2c 43 3d 31 31 2c 53 3d 34 2c 45 3d 32 2c 50 3d 36 32 
```

These strings are decoded as:
```
N=3;P=1;L=I=Antiboks,M=0,C=6,S=4,E=2,P=62&L=I=ASV17,M=0,C=11,S=4,E=2,P=46&L=I=ASV17-dlink,M=0,C=6,S=4,E=2,P=68&L=I=fjorde123,M=0
N=3;P=2;,C=1,S=4,E=2,P=58&L=I=JOJ,M=0,C=11,S=4,E=2,P=47&L=I=Kjellerbod,M=0,C=6,S=4,E=2,P=62&L=I=mgmt,M=0,C=1,S=4,E=2,P=74&L=I=Ri
N=3;P=3;ndedal,M=0,C=11,S=4,E=2,P=62
```

Which, when joined after removing the N/P paging info, becomes::
```
L=I=Antiboks,M=0,C=6,S=4,E=2,P=62&L=I=ASV17,M=0,C=11,S=4,E=2,P=46&L=I=ASV17-dlink,M=0,C=6,S=4,E=2,P=68&L=I=fjorde123,M=0,C=1,S=4,E=2,P=58&L=I=JOJ,M=0,C=11,S=4,E=2,P=47&L=I=Kjellerbod,M=0,C=6,S=4,E=2,P=62&L=I=mgmt,M=0,C=1,S=4,E=2,P=74&L=I=Rindedal,M=0,C=11,S=4,E=2,P=62
```

And after splitting this on & we get the final result:
```
L=I=Antiboks,M=0,C=6,S=4,E=2,P=62
L=I=ASV17,M=0,C=11,S=4,E=2,P=46
L=I=ASV17-dlink,M=0,C=6,S=4,E=2,P=68
L=I=fjorde123,M=0,C=1,S=4,E=2,P=58
L=I=JOJ,M=0,C=11,S=4,E=2,P=47
L=I=Kjellerbod,M=0,C=6,S=4,E=2,P=62
L=I=mgmt,M=0,C=1,S=4,E=2,P=74
L=I=Rindedal,M=0,C=11,S=4,E=2,P=62
```

So each L entry is made up of the same set of keys:

 * I: essid
 * M: opmode? or authalg? (always 0 in the sample)
 * C: channel (2.4 GHz only)
 * S: key_mgmt/auth_alg/proto?
 * E: key_mgmt/auth_alg/proto?
 * P: relative signal. Higher is better. dBm + 100?

Still need to figure out the mapping of the M,S,E keys to
wpa_supplicant config settings. I assume they represent enums.  But we
can simply treat them as opaque values since we only use the survey
data to help setup WiFi anyway. We copy these to the setup request,
and do not need to know what they mean.


FWIW, my example setting `M=0;I=Kjellerbod;S=4;E=2`
is mapped to this wpa_supplicant configuration:
```
# cat /tmp/wpa_supplicant.conf 
ctrl_interface=/var/run/wpa_supplicant
device_type=4-0050F204-3
model_name=DCS-8000LH
manufacturer=D-Link
os_version=01020300
config_methods=push_button virtual_push_button
eapol_version=1
network={
        scan_ssid=1
        ssid="Kjellerbod"
        key_mgmt=WPA-PSK
        auth_alg=OPEN
        proto=RSN
        psk="redeacted"
}
```

##### A201

This write request allows setting an admin password, used for example
by the webserver. It takes the old and new passwords as unencoded
input, verifies that the old password matches, and then change the
admin password to the provided new one.

The initial password is empty, which prevents webserver
authentication. Simply provide an empty string for the old password in
the first request: **P=;N=newpassword**

But this request is much more useful in other ways.... The new passord
(N_str) is processed like this (after slight compression of the
interesting code lines):

```C
	snprintf(cmd, sizeof(cmd), "mdb set admin_passwd %s", N_str);
	snprintf(cmdbuf, sizeof(cmdbuf), "%s > %s 2>&1", cmd, p_name);
	fp = popen(cmdbuf, "r");
```

You don't have to be a security expert to see the problem here. But
one mans bug is another mans feature :-)


##### A303

The two strings S and M are url decoded and checked for special
characters.  Then the **orginal** url encoded strings are written to
**/tmp/mydlink/reg_info** and SIGUSR1 is sent to the **da_adaptor**
process.  Presumably triggering it to reread the reg_info file.

It is pretty safe to assume that this provides some registration info
to the mydlink system, allowing it to connect to the cloud service.

The set of allowed characters is rather interesting:
```
 "ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789 !\"#$%&'()*+,-./:;<=>?@[\\]^_`{|}~"
```

Which initially made me think that this was an obvious security hole,
since I missed the point that it's the url encoded strings that are
used on the command line.

But given the quality of the rest of the code here, I would be very
surprised if there isn't an issue or ten in the da_adaptor code
allowing this to be abused. It's just a bit harder to figure out
without the source code.
