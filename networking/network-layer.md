# Network Layer
- *host-to-host* communication
- Encapsulates transport layer *segments* to network layer **datagrams**
>Application Layer and Transport Layer only run on *end systems*, but Network Layer protocols work on **every host and router** in the network.
- Has 2 major functions
    - **Forwarding** - **router local action**: Moving datagram from input link to required *output link*
    - **Routing** - **network wide action**: Figuring out **routing algorithms** i.e. apt pathway for *datagram*
### Forwarding Information Base (FIB)
- *data structure* used by networking devices (routers, switches) to determine next *hops* from *input link* to required *output link*
    - *routers*: at *network layer* - use *IP address* to figure apt output link for datagram
    - *switches*: at *data link layer*