---
layout: post
title: "SEC503 Commandline Tools"
categories: Commandline
---

## tcpdump
> Capturing, viewing and filtering PCAPs
_Read file with various fields_ 
```bash
# Basic usage
# -r read from file
# -n do NOT resolve IPs to hostnames
# -t do NOT print timestamps
tcpdump -ntr file.pcap

# -e Show MAC Addresses/Link Layer details
# -c print x number of packets
tcpdump -ntr file.pcap -e -c 10
```

_Alt Timestamps_
* `-tt` print UNIX epoch time
* `-ttt` print delta-time between each line

_Hex & Verbose Output_
```bash
# -x print the data of each packet in hex
# -xx also prints link-layer header
tcpdump -ntr file.pcap -s
```

```bash
# -X print in hex AND ASCII
# -XX also prints link-layer leader
tcpdump -ntr file.pcap -X
```

```bash
# -v print verbose including TTL, ID, total length, IP options
# -vv includes additional app-layer fields, -vvv even more verbose
tcpdump -ntr file.pcap -v
```

_BPF_
```bash
# The BPF is placed in quotes after other arguments
# E.g. TCP SYN&ACK set:
tcpdump -ntr file.pcap 'tcp[13]&0x12 = 0x12'
```

## snort
> Writing and executing alerting on traffic

_Run config file over a pcap, and display alerts_
```bash
# Logs will output to current dir, unless -l ./logdir is specified
snort -c ./config_file.lua -r ./mypcap.pcap -A alert_full
```

## zeek
> Distributed, event-based signaturing and analysing of traffic 

> formerly known as _Bro_

_test a zeek script_
```bash
zeek script.zeek
```

_injest logs from pcap_
```bash
# This will create the various .log files in the current directory
# Pass - as filename to read from STDIN
zeek -r ./mypcap.pcap
```
_signatures identify particular events and flag them (**publish**)_
* A zeek script can define a script that that **subscribes** to an event triggered by a signature match 
```bash
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
> Netflow analysis

_Convert PCAP to SiLK Format_
```bash
rwp2yaf2silk --in=file.pcap --out=file.silk
# or use --in=- to read from stdin
```

_Manipulate Netflow data from the SiLK Database that's running_
```bash
rwfilter --type=all --proto=0- --start-date=yyyy/mm/dd --end-date=yyyy/mm/dd --pass=stdout | # pipe to another rw command
```
* Chain muiltiple `rwfilter` commands to do more complex filtering
* `--flags-all` to specify TCP flags
* `--flags-initial` to search flags recorded by the first packet in a flow. `--flags-initial=S/SA` is the same as `tcp[13]&0x12=0x02` (i.e. SYN flag only)
* `--print-stat` prints flow statistics including filter pass/fail count
* Filter by various flow properties:
    * `--proto` (e.g. =6 for TCP, 0- for all, or specify range)
    * `--sport`, `--dport`, `--aport` (either port)
    * `--saddress`, `--daddress`
        * `--any-address=1.2.3.4` match source or destination IP
    * `--scidr`, `--dcidr`, `--anycidr`

* Pipe to another rw command such as _(generally use `--fields` with these)_:
    * `rwstats` groups into bins by the specified fields, and produces stats
        * `--count=0` prints all bins
        * `--bytes` count by bytes
    * `rwcut` Display tabluated output of specified fieldsA
    * `rwcount` Summarise records across time, sum(bytes,packets,records)
    * `rwuniq` group/count by a specified column
    * `rwcombine` to combine all flows into single statistic (e.g. summing total bytes across multiple flows)

```bash
# Get tabulated netflow data
rwfilter ... | rwcut --fields sip,sport,dip,dport,flags,packets,protocol,dura
# Which source sends the greatest number of bytes
rwfilter ... | rwstats --fields 
```