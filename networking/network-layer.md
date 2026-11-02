# Network Layer
- *host-to-host* communication
- Encapsulates transport layer *segments* to network layer **datagrams**
>Application Layer and Transport Layer only run on *end systems*, but Network Layer protocols work on **every host and router** in the network.
- Has 2 major functions
    - **Forwarding** - **router local action**: Moving datagram from input link to required *output link*
    - **Routing** - **network wide action**: Figuring out **routing algorithms** i.e. apt pathway for *datagram*
### Forwarding Information Base (FIB) / IP Forwarding Table (IPF)
- *data structure* used by networking devices (routers, switches) to determine next *hop* (next network device where packet is to be sent) from *input link* to required *output link*
    - *routers*: at *network layer* - use *IP address* to figure apt output link for datagram
    - *switches*: at *data link layer*
- **RIB** is *router information base*, a collection of *all possible paths* a datagram can follow, to reach destion IP address.
    - *RIB* handles the complex algorithm that determines the best path for datagram to follow.
    - It opperates in the *control panal* where complex calculations are made.
- **FIB** i.e. *forwarding information base* is dataset of *winning* pathway.
    - Avoids the *control panal* entirely, allowing datagram to be forwarded at wire speed

#### Lookup:
- When datagram arrives, IP address of destination is looked up in the *forwarding table*, to figure out the next *hop* i.e. next network device (switch, router etc) to go to.
