# til - Today I learned

### make - allow N jobs at once ###

This will use all but one CPU:

```make -j $(($(getconf _NPROCESSORS_ONLN) - 1))```

### Filtering USB communication in wireshark ###

When there is a transfer from host to device, the SUBMIT "packet" contains the actual USB data transmitted. When there is a transfer from device to host (initiated by the host, as always), the COMPLETE "packet" contains the actual USB data transmitted.

```!(usb.urb_type == URB_SUBMIT && usb.endpoint_number.direction == IN) && !(usb.urb_type == URB_COMPLETE && usb.endpoint_number.direction == OUT)```

Thanks to http://superuser.com/questions/873896/wireshark-usb-traces-explanations

### Calling xrandr when external monitor is plugged and unplugged ###

run ```udevadm monitor --environment```
then unplug device, wait till output changes, then plug it in
based on seen actions and variables, create an udev rule in /etc/udev/rules.d/XX-whatever.rules:
```
ACTION=="change", SUBSYSTEM=="drm", ENV{DISPLAY}=":0", ENV{XAUTHORITY}=" INSERT PATH TO YOUR X AUTHORITY FILE" RUN+="/etc/udev/external-monitor.sh"
```

and script reference in this rule:

```
#!/bin/bash
TAG="$0"
echo "Invoked ${0} --- $(env)" | logger -t "$TAG"
sleep 3
echo "invoking xrandr" | logger -t "$TAG"
/usr/bin/xrandr --verbose --display :0 --output HDMI1 --auto --primary --output LVDS1 --auto --right-of HDMI1 2>&1 | logger -t "$TAG"
echo "finished" | logger -t "$TAG"
```

Log messages will go to local syslog and most likely end up in /var/log/messages with tag equal to script filename.


### Continuous monitoring of memory, cpu, temperature, disks, network and other stats with export to CSV ###

https://github.com/dagwieers/dstat
Written in python, mostly reads data from files in /proc or /sys. Easily extensible via plugins.  


### Showing trace in bash ##

```
# + date function() file:line +
PS4='+$(date "+%F %T") ${FUNCNAME[0]}() $BASH_SOURCE:${BASH_LINENO[0]} + '
set -x
```


### Hardware information summary for Linux

```
wget --no-check-certificate https://github.com/smxi/inxi/blob/master/inxi
./inxi -c0 -v5
```


### Creating an empty large file

dd is relatively slow because it does real IO (it varies based on block size parameter and disk speed, etc.)
```
time -p dd if=/dev/zero bs=1M count=1024 of=1Gfile-dd
1024+0 records in
1024+0 records out
1073741824 bytes (1.1 GB) copied, 12.7845 s, 84.0 MB/s
real 13.00
user 0.00
sys 0.73
```

falocate is super quick - it allocates uninitialized blocks which does not need real IO to all blocks.

```
time -p fallocate -l 1G 1Gfile-fallocate
real 0.09
user 0.00
sys 0.00
```


### Using gdb to get full backtrace of all threads to a file 

Similar to 
```bash
pkill -3 java
```
```bash
executableName='/usr/lib64/firefox/firefox'
gdb "${executableName}" $(pidof "${executableName}") -batch -ex "thread apply all bt" &> stacktrace.txt 
```

### Linux kernel module parameters - how to show current values

```bash
module='snd_hda_intel'

# this should work nearly everywhere
for i in /sys/module/${module}/parameters/* ; do echo "$i = $(cat $i)"; done
# the following requires sysfsutils package
systool -vm $module
```

### Drop outgoing port unreachable ICMP messages

```bash
# filter port unreachable ICMP messages (Type:3 Code: 3) sent from this device 
# other ICMP packets work without change
# (assumes that iptables are empty)
# 
# list of known ICMP types and codes: iptables -p icmp -h
iptables -I OUTPUT -p icmp --icmp-type  port-unreachable -j DROP
```

#### Log and drop

```bash
iptables -N LOGGING
iptables -A OUTPUT -p icmp --icmp-type destination-unreachable -j LOGGING
iptables -A LOGGING -j LOG --log-prefix "IPTables-Dropped: " --log-level 4
```

### scapy - Programmable tcpdump and tcpreplay in Python

Very versatile tool for conditional capturing of network traffic or generating mangled packets. Good for various security related tests (e.g. MAC spoofing, DHCP exfaustion, ...). Nice tutorial at https://thepacketgeek.com/tag/scapy/


## About

I shamelessly stole this idea from [brenthaas/til](https://github.com/brenthaas/til).
