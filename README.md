# til

Today I learned

### Using gdb to get full backtrace of all threads to file 

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
