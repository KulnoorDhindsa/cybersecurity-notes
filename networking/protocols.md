# Protocols
*Resource: Networking a Top Down Approach by Kudose and Ross*

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
TCP is called *connection-oriented* because before the sender sends data to the receiver, a *handshake* takes place.

- The *handshake* consists of *segments* carrying control information about both devices, including their ports.
> TCP only runs on end-systems (not routers), routers only look at the IP header, not the TCP header.
- **Duplex service**: if data is being transferred from A to B, data can simultaneously be transferred from B to A.
- **Point-to-point**: TCP establishes a direct connection between two singular endpoints (sender and receiver).
  - *Multicasting*, a single sender sending to multiple receivers; is **not possible** over TCP.
- TCP uses a **timeout-retransmit** mechanism for lost packets:
  - After sending a packet, a *timer* starts. If an **ACK** isn't received before it expires, the packet is assumed lost and retransmitted.
  - The timer is *dynamic* and adjusts based on measured **RTT (Round-Trip Time)**.
  - If a retransmitted packet times out again, the interval is doubled (exponential backoff) to avoid congesting the network further.

---

## Three-Way Handshake

Used by TCP to establish a *reliable connection* between two end systems. It's an exchange of *synchronized sequence numbers*; client and server each share the **Initial Sequence Number (ISN)** their segments will carry.

- Called a *three-way handshake* because 3 segments are exchanged between the **client process** (initiator) and the **server process**.
- It occurs between processes running on end systems.
- **Correction of a common misconception**: port numbers *are* present in every segment of the handshake, they're part of the TCP header, which is included in all three segments. What differs by layer is this: port numbers live in the **transport-layer (TCP) header**; IP addresses live in the **network-layer (IP) header**, which encapsulates the TCP segment as its payload.
> **Send buffer**: memory an end system reserves to hold segments of transport-layer data during the handshake and subsequent transfer.
- **MSS (Maximum Segment Size)**: the maximum amount of application-layer data a single segment can carry. A TCP segment = application data + TCP header.

### Steps

1. **SYN**: client sends a segment with the SYN flag set, synchronizing sequence numbers and signaling intent to start communication.
2. **SYN-ACK**: server responds with ACK (acknowledging the client's SYN) *and* its own SYN (synchronizing the server's ISN).
3. **ACK**: client acknowledges the server's SYN, completing the handshake and establishing a reliable connection.

### Important notes

1. **Sequence number consumption**: SYN and FIN flags each consume 1 sequence number even though they carry no application data, this is why the ACK is N+1.
2. **ISNs are randomized** as a security measure (see *Security: ISN Prediction* below).
3. **Half-open state**: after step 1, the server sits in **SYN-RECEIVED** state waiting for the final ACK. This state is what **SYN flood attacks** exploit.
> `SYN flood attack`: a DoS attack where the attacker sends many SYNs and never completes the handshake with an ACK, exhausting the server's half-open connection table so legitimate clients can't connect.
4. **RTT cost**: the handshake adds a full RTT before any data can flow; a real cost when traffic is heavy, which is one reason UDP is preferred when speed matters more than reliability (e.g. DNS, VoIP, QUIC building its own reliability on top of UDP).

---

## Security: TCP Attacks and Defenses

This is the part of TCP that actually matters for a cybersecurity engineer; the protocol mechanics above exist to be *exploited or defended*, and most offensive networking tooling (nmap, hping3, scapy-based tools) is built directly on top of TCP flag and sequence-number behavior.

### 1. SYN Flood (DoS)
- **Attack**: flood the server with SYNs (often spoofed source IPs) and never send the final ACK. The server allocates state for each half-open connection; the backlog queue fills up and legitimate SYNs get dropped.
- **Defense - SYN cookies**: instead of storing per-connection state after receiving a SYN, the server encodes the connection info into the ISN it sends back (a cryptographic hash of source/dest IP, port, and a secret, plus a timestamp). No state is stored until the final ACK arrives and the server can *verify* the cookie. This means half-open connections cost effectively nothing to maintain, defeating the flood.
- Also mitigated with SYN backlog tuning, rate limiting, and firewalls/load balancers that proxy the handshake (SYN proxying).

### 2. ISN Prediction / TCP Sequence Number Spoofing (Session Hijacking)
- **Why ISNs are randomized**: if an attacker can *predict* the ISN a victim server will use next, they can perform a **blind spoofing attack** - inject forged segments into a TCP connection between two other hosts *without ever seeing the actual traffic* (e.g. from off-path, spoofing the source IP of a trusted host). This can be used to hijack an authenticated session or inject malicious data.
- **Historical case**: this is the mechanism behind the famous 1994 **Kevin Mitnick attack on Shimomura's systems**; early TCP/IP stacks used predictable, incrementing ISNs, making blind spoofing practical. This attack is the canonical reason modern stacks use cryptographically random ISNs (RFC 6528).
- **Takeaway for you**: randomness in protocol fields that look "just for uniqueness" is very often a security control in disguise. Always ask what an attacker could do if that field were predictable.

### 3. TCP RST Injection / Connection Reset Attacks
- Because RST segments (see flag table below) instantly tear down a connection and require no application-layer authentication, just a matching sequence number in-window, an attacker (on-path, or off-path with correctly guessed sequence numbers) can forge an RST to kill someone else's TCP connection.
- **Real-world example**: this is a documented technique used by the **Great Firewall of China** to censor connections, it injects forged RST packets to sever TCP sessions to blocked sites.
- Also used defensively/offensively in the wild for things like "connection reset" middleboxes and some IDS/IPS evasion and disruption techniques.

### 4. Port Scanning via Flag Manipulation
TCP's flag-based state machine is exactly what tools like **nmap** exploit to fingerprint open/closed/filtered ports, often *without* completing a full handshake, for speed and stealth. This is the direct payoff of understanding flags (see table below):

| Scan type | What it sends | Open port response | Closed port response | Why attackers use it |
|---|---|---|---|---|
| **Full connect scan** | SYN → SYN-ACK → ACK (completes handshake) | Connection completes | RST | Reliable, but logged everywhere (full connection established) |
| **SYN scan ("half-open")** | SYN, then RST instead of final ACK | SYN-ACK | RST | Faster, avoids completing the handshake — historically stealthier, since some older systems didn't log unfinished connections |
| **FIN scan** | Just a FIN, no prior handshake | No response (silently dropped, per RFC 793) | RST | Can slip past stateless firewalls/filters configured to only watch for SYN |
| **NULL scan** | Segment with *no flags set* | No response | RST | Same evasion logic as FIN scan — exploits ambiguous corners of the RFC |
| **XMAS scan** | FIN + PSH + URG all set ("lit up like a Christmas tree") | No response | RST | Same evasion logic |
| **ACK scan** | Just an ACK | RST (regardless of open/closed) | RST | Doesn't detect open/closed — used to map firewall rulesets (filtered vs unfiltered) |

The behavioral asymmetry in the FIN/NULL/XMAS rows exists because RFC 793 says a closed port *must* respond RST to any segment without SYN/ACK/RST set, but says nothing about what an *open* port should do with a segment that isn't part of an existing connection — most stacks just drop it silently. That RFC ambiguity is the entire scan.

### 5. TCP Options and Evasion
- **Window Scaling and SACK (Selective ACK)**, both carried in the Options field you glossed over, aren't just performance features, they've been used in real evasion and DoS techniques against IDS/IPS systems that don't correctly track scaled windows or reassemble segments the way the end host will, causing the IDS to "see" different data than what the victim actually receives (a classic **TCP/IP stack evasion** class of attack, documented extensively by researchers like Ptacek & Newsham).

---

## TCP Segment Structure

A TCP segment consists of a *header* (20–60 bytes) and an *application-layer payload*, using sequence numbers and acknowledgements for reliable, ordered delivery.
=======
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

```
 0                   1                   2                   3
 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|          Source Port         |       Destination Port       |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|                        Sequence Number                       |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|                    Acknowledgement Number                    |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
| Data  |Reser-|U|A|P|R|S|F|                               |
| Offset| ved  |R|C|S|S|Y|I|            Window Size          |
|       |      |G|K|H|T|N|N|                               |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|            Checksum          |         Urgent Pointer       |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|                    Options (if any)                          |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|                             Data                             |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
```
- **Source-Port (16 bits)**: Identifies sending application's *port number* 
- **Destination Port (16 bits)**: Identifies reciever applicaiton's port number
- **Sequence Number (32 bits)**: Marks position of *data byts*; holds sequence number of *first data byte* in single segment
- **Acknowledgement Number (32 bits)**: *Cumulative acknowledgement* Has the sequence number of **next** byte incoming, thus confirming all the previous bytes
- **Data Offset (4 bits)**: Specifies length of TCP header
- **Reserved**: Set to 0, reserved for future use
- **Window Size (16 bits)**: *flow control*, tells the other device how many bytes the current device can accept
- **CheckSum (16 bits)**: For *error detection* for corrupted bits
- **Source Port (16 bits)**: identifies the sending application's port number.
- **Destination Port (16 bits)**: identifies the receiving application's port number.
- **Sequence Number (32 bits)**: marks the position of data bytes; holds the sequence number of the *first* data byte in this segment.
- **Acknowledgement Number (32 bits)**: *cumulative* acknowledgement — holds the sequence number of the **next** byte expected, implying all prior bytes were received.
- **Data Offset (4 bits)**: length of the TCP header (needed because Options is variable-length).
- **Reserved (3 bits)**: set to 0, reserved for future use.
- **Flags (9 bits)**: see the flag table below, this is the field that all the security techniques above are built on.
- **Window Size (16 bits)**: *flow control*, tells the other side how many bytes it can currently accept.
- **Checksum (16 bits)**: error detection for corrupted bits.
- **Urgent Pointer (16 bits)**: used with the URG flag to mark urgent data.
- **Options**: variable length; includes MSS negotiation, **Window Scaling**, **SACK permitted**, and timestamps; see *Security: TCP Options and Evasion* above.

### Flags

| Flag | Full name | Function |
|---|---|---|
| **SYN** | Synchronize | Initiates a connection, synchronizes ISN |
| **ACK** | Acknowledge | Marks the Acknowledgement Number field as valid; used in almost every segment after the handshake |
| **FIN** | Finish | Requests graceful connection termination (sender has no more data) |
| **RST** | Reset | Abruptly terminates a connection — sent for segments to a closed port, or to force-kill an existing connection |
| **PSH** | Push | Tells the receiver to pass buffered data to the application immediately rather than waiting to fill the buffer |
| **URG** | Urgent | Marks data as urgent; used with the Urgent Pointer (rarely used today) |

### Sequence Numbers
Every byte in the data stream is numbered. The **Sequence Number** in a segment header is the number assigned to that segment's **first** data byte.
- Example: a segment starting at byte `0` and running to byte `99` carries Sequence Number `0`. The next segment, starting at byte `100`, carries Sequence Number `100`.

### Acknowledgements
An Acknowledgement Number of `n` means the receiver is now expecting byte `n`, implicitly confirming all bytes up to `n-1` were received.

### Connection Teardown
The handshake gets covered constantly; teardown is just as important and is where a lot of real attacks (RST injection above) live.
- **Graceful close (four-way)**: either side can initiate. A → B: FIN. B → A: ACK (B's side may still have data to send). B → A: FIN (when B is done too). A → B: ACK, connection fully closed.
  - After the final ACK, the closing side enters **TIME_WAIT**; it holds the connection's state for a period (typically 2×MSL) to catch any stray/delayed segments. This state is itself relevant to security: too many connections stuck in TIME_WAIT (or an attacker deliberately inducing many) can exhaust a server's resources.
- **Abrupt close**: either side can send a bare **RST** to kill the connection immediately, discarding any unsent/unacknowledged data. This is the mechanism attackers abuse in RST injection.

---

## RTT and Timeout

- The *timeout* timer must be **larger** than the RTT (to avoid unnecessary retransmits), but not so large that a genuinely lost packet takes forever to be noticed (which also has security relevance, attacks that induce excessive retransmission, like some low-rate DoS techniques, exploit this tension).
- RTT is **not** measured for retransmitted segments (Karn's Algorithm); since you can't tell if the ACK you got corresponds to the original send or the retransmission, measuring it would corrupt the RTT estimate.

---

## Congestion Control 
End-to-end mechanism used to regulate the rate of packages being sent across the network based on the rate at which the packages are being recieved by the reciever to *prevent congestion* and *over-flow* in the network.

**Congestion Window (cwnd)** is a *state variable* on the sender side.

- TCP is **self-clocking** as ACKs are used to trigger / clock / adjust size of `cwnd` (congestion window) which adjusts rate of data being snet in the connection.
- *Ideal* rate to send data across TCP connection **without congesting the connection** and **utilizing full potential** are:
    1. *Decrease when segment lost*: To re-send the ACK and the lost segment which may get stuck in the congestion if speed of sending segments is too fast
    2. *Increase when ACK of previously lost segment is recieed*: To maintain earlier speed as *connection is smooth* (assumption made whEN ACK of lost packet is recieved
    3. 

---
#### Acknowledgements:
An acknowledgement number of `n` means the receiver is waiting for the `n`th byte, *automatically* implying that bytes up to `n-1` have been received.

*These are ongoing notes, thus not finished yet :)*



