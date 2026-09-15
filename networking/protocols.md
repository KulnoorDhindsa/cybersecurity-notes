# Protocols

# UDP
Takes *messages* from application processes, attaches the required *metadata* like *source and destination IP addresses and port numbers* as *8 byte headers* (no trailers) to the front of the message.

It makes **best-effort attempt** to ensure that all the packages are sent safely from sending end-system to recieving end-system, with **NO guarantee**.

>All the small packets or *messages* have the same metadata i.e. the same source and destination IP address and port numbers

>**NO handshake** happens here (inlike in TCP), thus its *connectionless*.

UDP is still used as:
- **NO** handshake or transmission *delay*
- Sends data *instantly* (after encasing data into a *UDP segment*), preventing older data from blocking or lagging incoming data
    - TCP has **congestion control mechanism**, if a packet *drops*, TCP stops the message and waits for that data to be re-transmitted and *acknowledged* by destination host/port
- UDP has **smaller Headers** (8 byte headers) compared to TCP (20 byte headers)
- UDP is used in:
    - **Live-Videos**: Dropping some part of the videos/voice and continuing the rest is preffered over stopping the entire stream
    - **Online Gaming**: Faster updates and more coordination with inputs and outputs
    - **DNS**: Quick lookups prefer faster single-response and reply methods rather than maintaining long sessions over the Network.
    - **RIP (Routing Information Protocol) routing table updates**: Digital maps or database stored inside a router (used as GPS) using RIP which lists all known *network destinations*, IP address of next router (NEXT HOP), total number of routers (hops) a packet passes to get to its destination (Metric / Next HOP)
>Servers responsible for a single application supports many active clients when application runs on UDP rather than TCP
- UDP is NOT used in:
    - **HTTP**: Reliability is more credible for web-pages rather than speed
    - **Few Streaming videos**: Where reliability and error-free delivery out-weigh the speed and cost of UDP services
        - TCP is **fire-wall friendly** 
### Segment Structure
- **Header**: UDP attaches *8 byte headers* to data from application layer, making it a **segment**. A UDP header includes the following:
    - **Source Port**: (2 bytes) Sending applicaiton's port number
    - **Destinaiton Port**: (2 bytes) Receiving application's port number
    - **Length**: (2 bytes) Total byte length of the UDP header and data payload
    - **Checksum**:  (2 bytes) Recieving host uses it to detect any *accidental* error in the segment like data corrupted due to **signal loss**
        - UDP *Checksum* **DOES NOT** check for malicious entries in segments, its only a maths formula, easy to fake
- **Data Payload**: The Application layer data being sent (e.g. a DNS query) of variable length (determined by `Length` in the header)
---
# TCP
Its called *connection-oriented* as before the actual sender sends data to reciever, a *handshake* takes place.
- The *handshake* consists of *segments* of information regarding both devices and their ports.
>TCP only runs on end-systems!
- **duplex-service**: If data is being transfered from A to B, then at same time, data can be transfered from B to A
- **point-to-point**: TCP establishes direct connection between singular end points (sender and reciever)
    - *Multicasting* that is, singular sender sending to multiple recievers is NOT POSSIBLE !
## Three-way Handshake
Used by TCP to establish *reliable connection* between sending end system and recieving end system. Its an *exchange of Synchronised Sequence Numbers* between client and server, where they share the Initials Sequence Numbers (ISNs) that their segments will cary.
- *Three-way Handshake* as 3 segments are sent between 2 processes, *client process* (the one initiating the connection) and *server process*.
- It occurs between processes running on end systems.
- Port numbers, socket numbers of end systems are NOT in Three-way Handshakes, but in TCP headers. IP address are in IP header, in the *network layer packet*, that wraps the *Transport layer segment* as its *payload*.
>**Send Buffer**: When an end-sytem reserves a memory location for the segments of Transport Layer sent in the TCP handshake.
- **MSS** (Maximum Segment Size) is the maximum amount of application layer data that can be stored in the segment. TCP segment is chunks of *client-data* with TCP headers.
### Steps for 3 way Handshake
1. **SYN**: Client sends segment with **SYN** flag to synchronise *sequence numbers*, informing server that communication is likely to start with the client.
2. **SYN + ACK**: Server sends **SYN - ACK** where, **ACK** for *acknowledgement* of recieving initial SYN flag from client, and *another SYN* flag to synchronize *sequence numbers* of the server, that is the Initial Sequence Numbers (ISNs) of the segments sent by server.
3. **ACK**: Client sends final ACK as it recieves the SYN-ACK sent by server, thus establishing a reliable connection for data transfer.
### Important:
1. **Sequence number consumption**: SYN and FIN flag consume 1 sequence number even though they carry no data. Thats why, ACK is ISN+1
2. **ISN are RANDOMIZED**: ISNs are *randomised* and not shared as a *security measure*, where hackers can't predict the next ISN and prevents hijacking of connection.
3. **half-open state**: After #1, reciever is in *half-recieved state* with only **SYN-recieved** and is *exploited* by a **SYN flood attack** - never sending the ACK.
>`SYN flood attack` is a DOS attack (denial-of-service) by not sending the ACK and flooding the connection with multiple SYNs and haulting the connection.
4. **RTT**: Its well known that the handshake adds an RTT, thus slowing connection when traffic is heavy, which is why UDP is preferred when *speed* is a factor in the condition.

### TCP Segment Structure
TCP segment consists of *header* (20-60 bytes) annd *application-layer payload* using *sequence numbers* and *acknowledgements* for reliable data transfer.
```
| Source Port | Destination Port|
+-+-+-+-+-+-+-+-+-
| Sequence Number|
+-+-+-+-+-
| Acknowledgement number |
+-+-+-+-+-+-
| Data Offset | Reserved | Flag(s) | Window Size|
+-+-+-+-+-+-
|Checksum | Urgent Pointer|
|+-+-+-+-+-
| Options |
+-+-+-+-+-
| Data |
```
- **Source-Port (16 bits)**: Identifies sending application's *port number* 
- **Destination Port (16 bits)**: Identifies reciever applicaiton's port number
- **Sequence Number (32 bits)**: Marks position of *data byts*; holds sequence number of *first data byte* in single segment
- **Acknowledgement Number (32 bits)**: *Cumulative acknowledgement* Has the sequence number of **next** byte incoming, thus confirming all the previous bytes
- **Data Offset (4 bits)**: Specifies length of TCP header
- **Reserved**: Set to 0, reserved for future use
- **Window Size (16 bits)**: *flow ctrls*, tells other device, how many bytes the current device can accept
- **CheckSum (16 bits)**: For *error detection* for corrupted bits

#### Sequence Numbers: 
Every byte in a segment is *numbered* (to track them). **Sequence Number** in a *segmemt header* is number assigned to **first** data byte.
- e.g: Segment started with data byte `0` (so Sequence Number for that segment is `0`) and went till data byte `99`. The next segment starts with data byte `100`, so Sequence Number for that segment is `100`.

#### Acknowledgements:
Acknowledgement number if `n` means, the reciever is waiting for `n`th byte, *automatically* implying that bytes upto `n-1` have been recieved.