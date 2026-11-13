# Wireshark-Traffic-Analysis
Task 5: Capture and Analyze Network Traffic Using Wireshark

Cyber Security Internship | Date: 13 November 2026

# Overview

This task covers basic packet capture and protocol analysis with Wireshark. Live traffic was captured on the active network interface of a Kali Linux virtual machine while ping, DNS and web requests were generated. The capture was then inspected to identify the protocols in use and to read the details of individual packets.

# Objective

Capture live network packets and identify basic protocols and traffic types.

# Environment
Item	Details
Operating system	Kali Linux 2026.3 (VMware virtual machine), Linux 7.1.5
Capture tool	Wireshark 4.6.6
Capture interface	eth0
Machine address	192.168.10.131/24
DNS server	192.168.10.2
Traffic generators	ping, nslookup, curl
# Procedure
Step 1: Check Wireshark and the active interface
bash
wireshark --version
ip a

Wireshark 4.6.6 was already installed (Figure 1). ip a showed that eth0 was up with the address 192.168.10.131/24, so eth0 was chosen as the capture interface (Figure 2).

Step 2: Start the capture

Wireshark was started with sudo wireshark, eth0 was selected, and the capture was started. The title bar reads "Capturing from eth0" in the screenshots.

Step 3: Generate traffic

While the capture was running, the following commands were run in a terminal:

bash
ping -c 5 8.8.8.8
nslookup example.com
curl http://example.com
curl https://www.google.com

These produce ICMP, DNS, TCP and HTTP traffic. The plain HTTP request to example.com was used because its content is readable, unlike HTTPS.

Step 4: Stop the capture and inspect packets

The capture was stopped after the commands had finished. Packets were examined in the packet list and in the details and bytes panes (Figures 3 and 4).

Step 5: Filter and export

The following display filters isolate each protocol seen in this capture:

Filter	Shows
icmp	Ping echo requests and replies
dns	DNS queries and responses
tcp	All TCP segments
tcp.flags.syn == 1	The start of each TCP connection
http	Plain-text HTTP requests and responses

The capture was saved through File > Save As using the pcap format, as capture.pcap.

5. Evidence
Figure 1: Wireshark version

Wireshark version output

wireshark --version confirms Wireshark 4.6.6 on Linux 7.1.5 (Kali).

Figure 2: Interface check and traffic generation

ip a output and the commands used to generate traffic

ip a shows eth0 with address 192.168.10.131. The terminal also shows ping -c 5 8.8.8.8 (5 packets sent, 5 received, 0% loss, round-trip min/avg/max of 69.004 / 249.034 / 380.529 ms) and the nslookup example.com result, which came from the DNS server at 192.168.10.2.

Figure 3: Capture start, SSDP, ICMP and DNS

Wireshark packet list showing TCP, SSDP, UDP, ICMP and DNS

Packets 1 to 21 of the capture. Frame 8, the first ICMP echo request, is selected, and its details show the Ethernet II, IPv4 and ICMP layers.

Figure 4: DNS, TCP handshake and HTTP

Wireshark packet list showing DNS, the TCP handshake and HTTP

Packets 13 to 33. Frame 28, the final ACK of the TCP three-way handshake, is selected, and its details show the Ethernet II, IPv4 and TCP layers.

# Protocols Identified
Protocol	Packets (example)	Source and destination	What it shows
ICMP	8 and 9	192.168.10.131 and 8.8.8.8	Echo request and echo reply from ping
DNS	18 and 19	192.168.10.131 and 192.168.10.2	Query for the A record of example.com, and the response
TCP	26, 27, 28	192.168.10.131:52582 and 104.20.23.154:80	Three-way handshake before the web request
HTTP	29 and 33	192.168.10.131 and 104.20.23.154	GET / HTTP/1.1 and the 200 OK (text/html) reply
SSDP	3 to 6	192.168.10.1 and 239.255.255.250	M-SEARCH * HTTP/1.1 multicast discovery messages
UDP	7	192.168.10.1:57621 and 192.168.10.255:57621	A 44-byte broadcast datagram

Six protocols were identified, which is more than the three required.

# Packet Analysis

# 7.1 ICMP (ping)

Packet 8 is the first echo request, sent from 192.168.10.131 to 8.8.8.8, with sequence number 1 and TTL 64. Packet 9 is the reply, which arrived about 69 ms later and has a TTL of 128. This matches the 69.0 ms reported by ping for the first packet. Five request and reply pairs appear in the capture (sequence numbers 1 to 5), matching the five packets sent. The request and reply TTLs differ because each is set by the host that created the packet.

# 7.2 DNS

Packet 18 is a standard query (ID 0x9994) for the A record of example.com, sent to the DNS server at 192.168.10.2. Packet 19 is the response. It returns two IPv4 addresses, 172.66.147.243 and 104.20.23.154, and lists the Cloudflare name servers for the domain.

Queries for the AAAA (IPv6) record were also sent, and the response lists 2606:4700:10::ac42:93f3 and 2606:4700:10::6814:179a. The lookups appear twice in the capture, which fits the separate nslookup and curl runs. The addresses match the nslookup output in Figure 2.

# 7.3 TCP three-way handshake

Before the web request, the machine opened a connection to 104.20.23.154 (one of the addresses returned by DNS):

Packet	Time (s)	Direction	Flags	Notes
26	35.590	client to server	SYN	Seq=0, Win=64240, MSS=1460, SACK permitted, window scale 256
27	35.636	server to client	SYN, ACK	Seq=0, Ack=1, MSS=1460
28	35.637	client to server	ACK	Seq=1, Ack=1

The connection was established in about 46 ms. Sequence numbers are shown as relative values by Wireshark. Frame 28 is 54 bytes, with source port 52582 and destination port 80.

# 7.4 HTTP

Once the handshake completed, packet 29 carried the request GET / HTTP/1.1. The server acknowledged it (packet 30), then sent the page in a segment of 932 bytes (packet 31), which Wireshark reassembled and displayed as HTTP/1.1 200 OK (text/html) in packet 33. The client acknowledged the data in packet 32. Because this exchange is unencrypted, the request and response content can be read directly in Wireshark.

# 7.5 Background traffic

Not every packet came from the commands that were run:

SSDP (packets 3 to 6): the device at 192.168.10.1 sent M-SEARCH messages to the multicast address 239.255.255.250 about once per second. SSDP is used to discover devices on a local network.
UDP (packet 7): a 44-byte broadcast from 192.168.10.1 to 192.168.10.255 on port 57621. Its purpose was not investigated.
TCP (packets 1 and 2): an already-open connection to 34.107.243.93 on port 443. Wireshark marked the reply TCP ACKed unseen segment because the segment being acknowledged was sent before the capture began.
# Findings Summary
The capture shows ICMP, DNS, TCP, HTTP, SSDP and UDP traffic.
A complete web request can be followed from start to finish: DNS lookup, TCP handshake, HTTP request and HTTP response.
Ping results in the capture agree with the ping command output, and DNS results agree with nslookup.
Plain HTTP is readable in a capture. Wireshark shows the full request and response, which is why sensitive traffic should be encrypted.
Background protocols appear alongside the generated traffic, so a display filter is needed to isolate the traffic being studied.
# Limitations

The screenshots show the unfiltered packet list. Filtering is described in Section 4 but is not shown in a screenshot.
The curl https://www.google.com request was run, but no TLS packets are shown in the screenshots, so TLS is not included in the table of identified protocols.
The capture was taken on a virtual machine behind NAT, so the addresses are private lab addresses. Results from another network would differ.
# Key Learning

Starting a capture on the correct interface and generating controlled traffic.
Reading Wireshark's packet list, details and bytes panes.
Following a web request across DNS, TCP and HTTP.
Recognising the TCP three-way handshake in the packet list.
Using display filters to separate one protocol from the rest.
Saving a capture as .pcap for later analysis.

# Repository Contents
README.md
capture.pcap
screenshots/
    01_wireshark_version.png
    02_interface_and_traffic_generation.png
    03_capture_icmp_dns.png
    04_capture_dns_tcp_http.png
 # Conclusion

Live traffic was captured on eth0 and analysed in Wireshark. The capture contains ICMP, DNS, TCP, HTTP, SSDP and UDP packets, and one complete web request was traced from the DNS lookup through the TCP handshake to the HTTP response. The task gave hands-on practice with packet capture and protocol analysis.

# Disclaimer

The capture was made on a personal lab virtual machine, for educational purposes, from traffic generated by the author. No other users' traffic was intentionally captured.
