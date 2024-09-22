---
layout: post
title: "SEC503 Commandline Tools"
categories: Commandline
---

## tcpdump
_Read file with various fields_ 
```bash
# Basic usage
# -r read from file
# -n do NOT resolve IPs to hostnames
# -t do NOT print timestamps
tcpdump -ntr file.pcap

# -e Show MAC Addresses
# -c print x number of packets
tcpdump -ntr file.pcap -e -c 10
```

_Hex Output_
```bash

```


## snort
_Run config file over a pcap, and display alerts_
```bash
snort -c ./config_file.lua -r ./mypcap.pcap -A alert_full
```

## zeek
_injest logs from pcap_
```bash
# This will create the various .log files in the current directory
# Pass - as filename to read from STDIN
zeek -r ./mypcap.pcap
```
_signatures identify particular events and flag them (**publish**)_
* A zeek script can define a script that that **subscribes** to an event triggered by a signature match 
```
# Pass signature file with -s
zeek -r ./mypcap.pcap -s my-sig.sig
```
_print statements in an event will print to STDOUT when running `zeek`_
```bash
# Provide the custom .zeek script file
zeek -r ./mypcap.pcap ./my.script.zeek
```

### zeek-cut
_choose which columns to use_
```bash
# Reads from STDIN, you usually want to pipe log files to it
# Select columns, -d displays time in human-readable format
# Select columns, -u displays UTC time in human-readable format
cat *.log | zeek-cut -u ts fuid id.orig_h id.orig_p id.resp_h id.resp_p sig_id event_msg sub_msg
```

## SiLK
_Convert PCAP to SiLK Format_
```bash
rwp2yaf2silk --in=file.pcap --out=file.silk
# or use --in=- to read from stdin
```

_Manipulate Netflow data from the SiLK Database that's running_
```bash
rwfilter --type=all --proto=0- --start-date=yyyy/mm/dd --end-date=yyyy/mm/dd --pass=stdout | # pipe to another rw command
```
* `--flags-initial` to search flags recorded by the first packet in a flow. `--flags-initial=S/SA` is the same as `tcp[13]&0x12=0x02` (i.e. SYN flag only)

* Pipe to another rw command such as:
    * `rwstats` groups into bins by the specified fields, and produces stats
        * `count=0` prints all bins
    * `rwcount` 
    * `rwuniq`
    * `rwcut` Display tabluated output of specified fields