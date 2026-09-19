# Protocols

# UDP
Takes *messages* from application processes, attaches the required *metadata* like *source and destination IP addresses and port numbers* as* 8-byte headers* (no trailers) to the front of the message.

It makes a **best-effort attempt** to ensure that all the packets are sent safely from the sending end-system to the receiving end-system, with **NO guarantee**.

>All the small packets or *messages* have the same metadata, i.e. the same source and destination IP address and port numbers

>**NO handshake** happens here (unlike in TCP), thus it's *connectionless*.

UDP is still used as:
- **NO** handshake or transmission *delay*
- Sends data *instantly* (after encasing data into a *UDP segment*), preventing older data from blocking or lagging incoming data
    - TCP has a **congestion control mechanism**; if a packet *drops*, TCP stops the message and waits for that data to be retransmitted and *acknowledged* by the destination host/port
- UDP has **smaller Headers** (8-byte headers) compared to TCP (20-byte headers)
- UDP is used in:
    - **Live-Videos**: Dropping some part of the videos/voice and continuing the rest is preferred over stopping the entire stream
    - **Online Gaming**: Faster updates and more coordination with inputs and outputs
    - **DNS**: Quick lookups prefer faster single-response and reply methods rather than maintaining long sessions over the Network.
    - **RIP (Routing Information Protocol) routing table updates**: Digital maps or databases stored inside a router (used as GPS) using RIP, which lists all known *network destinations*, the IP address of the next router (NEXT HOP), and the total number of routers (hops) a packet passes to get to its destination (Metric / Next HOP)
>Servers responsible for a single application support many active clients when the application runs on UDP rather than TCP
- UDP is NOT used in:
    - **HTTP**: Reliability is more important for web-pages rather than speed
    - **Few Streaming videos**: Where reliability and error-free delivery outweigh the speed and cost of UDP services
        - TCP is **fire-wall friendly** 
### Segment Structure
- **Header**: UDP attaches *8 byte headers* to data from application layer, making it a **segment**. A UDP header includes the following:
    - **Source Port**: (2 bytes) Sending application's port number
    - **Destination Port**: (2 bytes) Receiving application's port number
    - **Length**: (2 bytes) Total byte length of the UDP header and data payload
    - **Checksum**:  (2 bytes) Receiving host uses it to detect any *accidental* error in the segment, like data corrupted due to **signal loss**
        - UDP *Checksum* **DOES NOT** check for malicious entries in segments; it's only a maths formula, easy to fake
- **Data Payload**: The Application layer data being sent (e.g. a DNS query) of variable length (determined by `Length` in the header)
---
# TCP
Its called *connection-oriented* because before the actual sender sends data to the receiver, a *handshake* takes place.
- The *handshake* consists of *segments* of information regarding both devices and their ports.
>TCP only runs on end-systems!
- **duplex-service**: If data is being transferred from A to B, then at the same time, data can be transferred from B to A
- **point-to-point**: TCP establishes a direct connection between singular end points (sender and receiver)
    - *Multicasting*, that is, a single sender sending to multiple receivers, is NOT POSSIBLE!
- TCP uses **timeout - retransmit** mechanism for lost packets
    - After sending a packet, a *timer* is started; if, by the end, an **Ack** isn't received by the receiver, the packet is assumed to be lost and is retransmitted
    - The *timer* is *dynamic* and adjusts the **RTT (Round-Trip Time)** as well.
    - If a retransmitted packet times out, then the interval is doubled to prevent congestion of the network
## Three-way Handshake
Used by TCP to establish *reliable connection* between the sending end system and the receiving end system. It's an *exchange of Synchronised Sequence Numbers* between client and server, where they share the Initial Sequence Numbers (ISNs) that their segments will carry.
- *Three-way Handshake* as 3 segments are sent between 2 processes, *client process* (the one initiating the connection) and *server process*.
- It occurs between processes running on end systems.
- Port numbers and socket numbers of end systems are NOT in Three-way Handshakes, but in TCP headers. IP addresses are in the IP header, in the *network layer packet*, that wraps the *Transport layer segment* as its *payload*.
>**Send Buffer**: When an end system reserves a memory location for the segments of the Transport Layer sent in the TCP handshake.
- **MSS** (Maximum Segment Size) is the maximum amount of application layer data that can be stored in the segment. TCP segment is a chunk of *client data* with TCP headers.
### Steps for 3-way Handshake
1. **SYN**: Client sends a segment with the **SYN** flag to synchronise *sequence numbers*, informing the server that communication is likely to start with the client.
2. **SYN + ACK**: Server sends **SYN - ACK**, where **ACK** is for *acknowledgement* of receiving the initial SYN flag from the client, and *another SYN* flag to synchronise *sequence numbers* of the server; that is, the Initial Sequence Numbers (ISNs) of the segments sent by the server.
3. **ACK**: The client sends the final ACK as it receives the SYN-ACK sent by the server, thus establishing a reliable connection for data transfer.
### Important:
1. **Sequence number consumption**: SYN and FIN flags consume 1 sequence number even though they carry no data. That's why ACK is N+1
2. **ISNs are RANDOMIZED**: ISNs are *randomised* and not shared as a *security measure*, where hackers can't predict the next ISN, which prevents hijacking of the connection.
3. **half-open state**: After #1, reciever is in *half-recieved state* with only **SYN-recieved** and is *exploited* by a **SYN flood attack** - never sending the ACK.
>`SYN flood attack` is a DoS (denial-of-service) attack that doesn't send the ACK and floods the connection with multiple SYNs, halting the connection.
4. **RTT**: It's well known that the handshake adds an RTT, thus slowing the connection when traffic is heavy, which is why UDP is preferred when *speed* is a factor in the condition.

### TCP Segment Structure
TCP segment consists of a *header* (20-60 bytes) and an *application-layer payload*, using *sequence numbers* and *acknowledgements* for reliable data transfer.
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
- **Window Size (16 bits)**: *flow control*, tells the other device how many bytes the current device can accept
- **CheckSum (16 bits)**: For *error detection* for corrupted bits

#### Sequence Numbers: 
Every byte in a segment is *numbered* (to track them). **Sequence Number** in a *segmemt header* is number assigned to **first** data byte.
- e.g: Segment started with data byte `0` (so Sequence Number for that segment is `0`) and went till data byte `99`. The next segment starts with data byte `100`, so the Sequence Number for that segment is `100`.

#### Acknowledgements:
An acknowledgement number of `n` means the receiver is waiting for the `n`th byte, *automatically* implying that bytes up to `n-1` have been received.

### RTT and Timeout-time
- The *timeout* timer has to be **larger** than the RTT, but not large enough for exploitation.
- RTT is **NOT** measured for *re-transmitted* segment.

## Congestion Ctrl Mechanism
*Packet loss* happens when *router buffers* OVERFLOW and cause **network congestion**.
