# Analyze Traffic

## Exam Objectives

Demonstrate the ability to decipher the contents of packet capture headers.

### Five Tips for Decoding Packets

1. Offsets from the beginning of the packet start with 0
2. Four bits = 1 hex character
3. One byte = 2 hex characters
4. Fields can be any length (one bit, many bytes, fixed length or variable length)
5. Fields in one protocol identify the length and contents of others

## Analyze Traffic w/ Wireshark

### Loading Wireshark

```
wireshark
wireshark [pcap file]
```

### Profiling a pcap

Wireshark's "Statistics" menu items serve as a great starting point when jumping into a pcap...

```
Statistics > Protocol Hierarchy
```

![Protocol Hierarchy](../screenshots/analyze-traffic-wireshark-statistics.PNG?raw=true "Protocol Hierarchy")

Any of the traffic here could be suspicious, but the 'Data' classification under protocol hierarchy means Wireshark was unable to apply a dissector because it does not a) recognize the port number or b) no heuristic dissector matched the packets. You can right-click on the packets in question and 'apply as filter' for quick and easy analysis.

![Protocol Hierarchy](../screenshots/protocol_hiearchy_data_class.png?raw=true "Protocol Hierarchy - Data")

### Finding a Packet

```
Edit > Find Packet ... (or Ctrl + F)
```

It's a good idea to select the radio button "Packet bytes" in order to search for the string or hex value within the payload.

![Find Packet](../screenshots/analyze-traffic-find-packet.PNG?raw=true "Find Packet")

Tip: Use the CTRL+N keyboard shortcut to cycle through relevant matches (CTRL+N = "Next packet").

### Following Streams

Right-click -> Follow TCP/UDP/HTTP Stream is extremely useful for seeing the entire relationship and flow of packets pertaining to conversations of interest.

### Useful Display Filters and other Tips

Looking at the start of all TCP conversations:

```
(tcp.flags.syn == 1) and (tcp.flags.ack == 0)
```

Pro tip: Resulting conversations from the above filter can be color-coded with Wireshark's color feature, to allow for quicker identification of distinct conversations and streams.

Looking for ARP and gratuitous ARP:

```
arp
arp.isgratuitous
```

Looking for ICMP replies only:

```
icmp.type == 0
```

Looking for IP options:

```
ip.hdr_len > 20
```

### Carving Files Manually

1. Follow the stream in Wireshark
2. Filter the conversation to represent the specific side of the conversation of interest
3. Save the file as raw output
4. Use a file editing tool, like vi on Linux, or WinHex on Windows to carve out any unnecessary bytes
5. Save the modified file and perform anyh additional analysis

### Wireshark Tips and Resources

http://www.malware-traffic-analysis.net/tutorials/wireshark/index.html

## Analyze Traffic w/ Tshark

Tshark (terminal Wireshark) is a great tool for command-line analysis of packets.

Reading packet captures without name resolutions:

```
tshark -n -r [capture.pcap]
```

Using a display filter (-Y) to narrow down results to a specific IP and protocol:

```
tshark -n -r [capture.pcap] -Y 'ip.addr == 192.168.60.22 and smtp'
```

Specifying certain fields to display (-T fields -e [field]):

```
tshark -n -r [capture.pcap] -Y 'ip.addr == 192.168.60.22 and smtp' -T fields -e tcp.steeam
```

Following streams with tshark:

```
tshark -n -r [capture.pcap] -Y 'tcp.stream == [stream id]' -z follow,tcp,ascii,60
```

## Analyze Traffic w/ tcpdump

### tcpdump Filters w/ Examples

Example 1: Showing only packets with the source IP of 127.0.0.1 with only the ACK bit set:

```
tcpdump -nt -r tcpdump.pcap 'src host 127.0.0.1 and tcp[13] = 0x10'
```

Example 2: Showing only packets with the destination IP of 127.0.0.1 with either the ACK or RST flags set:

```
tcpdump -nt -r tcpdump.pcap 'src host 127.0.0.1 and tcp[13] = 0x10'
tcpdump -nt -r tcpdump.pcap 'dst host 127.0.0.1 and tcp[13] & 0x14 != 0
```

## Traffic Analysis / SiLK

Counting the number of records:

```
rwfilter suspicious.silk --proto=0-255 --print-stat
```

Looking for top (5) senders of data (bytes):

```
rwstats suspicious.silk --fields sIP --bytes --count=5
```

Looking for all Unique destination ports used by the UDP Protocol:
```
rwfilter suspicious.silk --protocol-17 --pass-stdout | rwuniq --fields dport
```

Looking for all unique IP address within a target Netrange that had the RST Flag set:
```
rwfilter suspicious.silk --cidr=10.0.0.0/8 --rst-flag=1 --pass=stdout | rwuniq --fields sIP
```

Looking for traffic that is from an IP Address, but NOT from specific protocols (TCP, UDP ICMP):
```
rwfilter <file.silk> --protocol=1,6,17 --fail=stdout | rwfilter --inut-pipe=stdin --saddress=<ip address> --pass=stdout
```
*Note - The method to filter out specific traffic, is created by directly filtering for this traffic and to print out everything that DID NOT match "--fail==stdout', then pipe all of that traffic into a new rwfilter using "--input-pipe=stdin" and specifying the new filter you wish to select.


Print the flows which match a filter to the screen:
```
rwfilter file.silk --any-address=192.168.1.1 --aport=80 --pass=stdout | rwcut
```

*NOTE - The 'rwcut' filter at the end will translate the Binary SiLK data to ASCII for printing.

rwfilter requires at least one input filter and one output filter.
Basic examples which could be helpful are:

INPUT FILTERS:
```
--saddress=IP_WILDCARD
--daddress=IP_WILDCARD
--any-address=IP_WILDCARD
--sport=INTEGER_LIST
--dport=INTEGER_LIST
--aport=INTEGER_LIST
--protocol=INTEGER_LIST
--icmp-type=INTEGER_LIST
--icmp-code=INTEGER_LIST
```
OUPUT FILTERS:
```
--all-destination=ALL_PATH
--fail-destination=FAIL_PATH
--pass-destination=PASS_PATH
--print-statistics
--print-statistics=STATS_PATH
--max-pass-records
--max-fail-records
--print-volume-statistics
--print-volume-statistics=STATS_PATH
```

## Scapy

### Creating a frame with ICMP

![Scapy - Create Frame](../screenshots/scapy-create-frame.PNG?raw=true "Scapy - Create Frame")

Saving a created packet to disk:

```
wrpcap("/tmp/icmp.pcap", frame)
```

Send crafted packet:

```
send(packet_object)
```

### Analyzing User Agent Stings

It's important to correlate host operating systems (OS) versions based on the user-agent strings observed in web traffic.

An example user agent string:
Mozilla/5.0 (Windows NT 6.1; Trident/7.0; rv:11.0) like Gecko

Windows NT 6.1 correlates to Windows 7.

List of Windows NT versions to OS:

6.0	Windows Vista

6.1 Windows

6.2	Windows 8

6.3	Windows 8.1

10.0 Windows 10 / Server 2016






