+++
title = "Notes on UDP"
date = 2020-10-26
math = false
highlight = false
categories = ["Networks"]
tags = ["Networks", "Protocols", "UDP"]

# Optional featured image (relative to `static/img/` folder).
[header]
image = ""
caption = ""

+++


UDP is commonly the target of jokes about how unreliable it is a transport protocol. 

> **"I could tell you a UDP joke. But I'm not sure you'll get it."**

Indeed, the protocol is very simple and does not provide guarantees like TCP. Applications on top of UDP must implement functionalities such as error correction, flow and congestion control, sequencing, and duplication if they require a reliable transfer. So that is all?

Not really. After revisiting the UDP chapter in the first volume of [TCP/IP Illustrated](https://www.pearson.com/us/higher-education/program/Fall-TCP-IP-Illustrated-Volume-1-The-Protocols-2nd-Edition/PGM69698.html), I have found some details worth pinning in a post.

## UDP basics

> IP protocol number: **17**
> 
> UDP header size is always: **8 bytes**


All fields of the UDP header are **2 bytes** long.  

```
0                   1                   2                   3
 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
-----------------------------------------------------------------
|          Source Port          |        Destination Port       |
-----------------------------------------------------------------
|             Length            |            Checksum           |
-----------------------------------------------------------------
```
 
###  Some facts about the header
 
*  The source port is optional. I can be **set to 0** if the sender never requires a reply.
*  The length field is the size of the header plus the payload. **This field is redundant**, because the IPv4 header already contains the total datagram size.
*  A datagram with **0 bytes of data** is acceptable (but rare)
*  The checksum field is **optional with IPv4**, but mandatory by [RFC1122](https://tools.ietf.org/html/rfc1122).


## Checksum

UDP is capable of providing error detection using the checksum field. 

### Some facts about the checksum 

* **UDP pads odd-lenght datagrams with a 0 byte** just for the computation and verifications
* Similarly to TCP, UDP **uses a pseudo-header** derived with fields from the IPv4 or IPv6 headers.
	* The pseudo-header is **40 bytes** long
	* The pseudo-header includes the **IP source and destination plus the protocol/ nextheader field**. 
	* There is a **0 padding before the protocol number** in the pseudo-header
	* Using IP layer fields lets the receiver **verify if the datagram arrived at the correct destination** (in a theoretical case that IP accepts wrongly accepts a packet to a different destination.)
	* The pseudo-header is completed with the **full UDP header and the data**.
* If the checksum is **0x0000 on the receiver side**, it indicates that the checksum was not computed. 
* The UDP datagram is **discarded silently** in the case of errors

When the packet goes through a **NAT**, the **checksum is recalculated** because the pseudo-header will have a different source IP.

## UDP and IPV6

*  The checksum is **mandatory for IPv6**, because it does not calculate a checksum like IPv4.
*  IPv6 supports **jumbograms** that are larger than **65535 bytes**. This value is larger than the Length field. When sending a jumbogram using UDP, the **Length field is 0**
*  The size of the UDP datagram for **jumbograms** is calculated using the **Jumbo Payload Option** (gives the length of the IPv6 payload)













 




