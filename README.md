# CS 118 Notes

<!-- vim: set spell: -->

<!-- Lecture 1 -->

## What is the internet?

- Three components:
  - Hosts (end systems)
  - Communication links
    - Fiber optics cables, radio, etc.
    - **NOTE:** The transition rate is called _bandwidth_
  - Routers and switches
    - **NOTE:** _Packet switching_ means forwarding packets of data
- Network structure:
  - Network edge
    - Internet boundaries, client and server hosts
      - Access networks like home Wi-Fi, DSL, cable network, Ethernet, cellular,
        etc.
  - Network core
    - Internet backbone of interconnected routers

## How are access networks constructed in the first place?

- Signals carried by EM waves (radio) or within wires (Ethernet)
- Network core will move bits around access networks to the destination, using
  routers
- Router functions:
  - Routing: determines source-destination route taken by _data routing
    algorithms_
  - Forwarding: moves data from router's input to appropriate router output

## How can the internet grow so big?

- Connecting all access ISPs to each other does not scale ($O(n^{2})$
  connections)
- Connect all to a global ISP is bad because if the main fails, everything fails
  - Also bad for business as one company will control all traffic
- Connecting to multiple global ISPs reduces work load and is more reliable
- Creating a hierarchical structure is even better as it reduces load and allows
  for regional ISPs

## What is the internet software architecture?

- It is complex:
  - Many difference pieces running in different places
- But simple:
  - All pieces adhere to simple rules
- Rules:
  - Enabling rules
    - Packet switching:
      - Breaking up load into smaller packets
      - Packets are labeled with a number to denote order
      - Packets are transmitted at _full link speed_

<!-- Discussion 1 -->

## Client-server model

- Asymmetric communication model
  - Client requests data:
    - Initiates communication
    - Waits for server response
  - Server (Daemon) responds to requests:
    - Discoverable by clients (e.g. IP address & port)
    - Waits for client connection
    - Processes requests, sends replies
- Clients and servers are programs at the **application layer**

## TCP: Transmission Control Protocol

- A connection is set up between client and server
- Reliable data transfer
  - Guaranteed deliveries of all data
  - No duplicate data will be delivered
- Data will always be received in the order it is sent
- Full-duplex byte stream (in two directions simultaneously)
- Regulated data flow with flow control and congestion control

## UDP: User Data Protocol

- Basic transmission service
  - No connection needed
  - Unit of data transfer: datagram (variable length)
- No reliability guarantee
- No ordered deliver guarantee
- No flow control / congestion control

## Socket APIs

- Socket:
  - An endpoint in inter-process communication across a computer network
  - TCP sockets have tuples of `<ip_addr:port>`
  - On Linux and MacOS, sockets are like files (socket handles are file
    descriptors)
  - TCP sockets provide `write()` and `read()` for sending and receiving data
- Socket port numbers
  - Smaller numbers are reserved and only accessible by super-users

## Caveat: Byte ordering matters

- Data that is sent in little-endian must be read as such, and vice versa
- The API defines methods to convert to proper network ordering
  - This only works for 16- and 32-bit data, more complex data types must be
    taken into account differently

<!-- Lecture 2 -->

## Packet switching

- **Store-and-forward** operations inside the network
- Sending host
  - Takes application message

## Circuit switching

- End-end resources are allocated and reserved for a voice call between source
  and destination
- `FDM`
  - `FDM` does one user at a time
  - `TDM` alternates users over time

## Packet switching vs. Circuit switching

- Packet switching allows more users in the network
- Example: 1 Mb/s link, where each user needs 100 Kb/s when active, and users
  are active 10% of the time.
  - Max supported users:
    - Circuit switching: 10
    - Packet switching: with `N` users, probability that more than 10 users are
      active at a time is less than 0.004. `N` must be:
      - Each user is active with probability 10%
      - $P(N, x) = \begin{pmatrix} N\\x \end{pmatrix} p^{x}(1-p)^{N-x}$
      - $\sum_{x=0}^{10} P(N, x) \ge 1 - 0.04\%$
      - $N = 35$

## What is the Internet Service Model?

- Internet best effort service model
  - Arguably the simplest service model
    - Only promise is to try its best
  - Does not guarantee anything, not even:
    - Successful packet delivery
    - Timing or in-order delivery
    - Minimum throughput/speed to deliver data

## Internet software appears in the form of protocols

- Protocols control sending and receiving messages
  - E.g. HTTP, Skype, TCP, etc.
- Protocols are software pieces
  - There are 100s\~100s of them
- Organized in layers which cannot only talk to the immediate upper or lower
  layer

## Layering

- Modularization allows for easier maintenance
- Different layers have their own protocols

### Layers

- **Application:** Supports network applications
  - FTP, SMTP, HTTP
- **Transport:** Process-to-process data transfer
  - TCP, UDP
- **Network:** Routing of datagrams from source to destination
  - IP, routing protocols
- **Link:** Data transfer between neighboring network elements
  - Ethernet, Wi-Fi, PPP
- **Physical:** Bits "on the wire"

## How to evaluate the internet

Performance is gauged with three metrics:

### Throughput

- Throughput is the rate (bits/time) at which units are transferred
- End-to-end throughput is equivalent to the bottleneck throughput

### Packet Loss

- Queue preceding link in buffer has finite capacity
- Packet arriving to full queue dropped
- Lost packet may be retransmitted by previous node, by source end system, or
  not at all

### Delay

- Host sending function:
  - Takes application message
  - Breaks it into smaller chunks, called packets, of length `L` bits
  - Transmits packets across network at rate `R`
  - Packet transmission delay $= \frac{L}{R}$

## Packet Delay

If arrival rate (in bits) to link exceeds transmission rate for a period of
time:

- Packets will queue and wait to be transmitted on the link
- Packets can be dropped (lost) if memory (buffer) fills up

### Four Sources of Packet Delay

- $d_{proc}$: Nodal processing
  - Check bit errors
  - Determine output link
  - Typically < milliseconds
- $d_{queue}$: Queuing delay
  - Time waiting at output link for transmission
  - Depends on congestion of router
- $d_{trans}$: Transmission delay
  - L: Packet length (bits)
  - R: Link bandwidth (bps)
  - $d_{trans} = \frac{L}{R}$
- $d_{prop}$: Propagation delay
  - d: Length of physical link
  - s: Propagation speed
  - $d_{prop} = \frac{d}{s}$

<!-- Lecture 3 -->

## Application Layer

### Creating a network app

- We need to incorporate different machines communicating across the internet
- We can do this several ways:
  - Client-server paradigm
    - Server
      - Always-on host
      - Has a permanent IP address
      - Often used in data centers for scaling
    - Clients
      - Communicate with the server
      - May be intermittently connected
      - Can have dynamic IP addresses
      - Do _not_ communicate directly with each other
    - Examples: HTTP, FTP
  - Peer-to-peer architecture
    - Each peer is a client _and_ server
      - Peers request services from each other in exchange for providing service
        to one another
    - Self-scalable architecture; more users means more demand but also more
      supply
    - Peers are intermittently connected and change IP addresses
      - No always-on server

### Communication across network

#### Processes

- A program running within a host
  - Client process initiates communication
  - Server process awaits communication
- To receive messages, processes must have an _identifier_
  - Identifier includes IP address and port number
    - IP address is not enough! Many processes can run on one host.

#### Sockets

- Sockets are where processes send and receive messages to and from
- Analogous to doors
- There will be one socket on each side of communication
- Sockets can use different transport services

#### Transport services

- UDP
  - Connectionless: no connection setup
  - Unreliable data transfer
  - No rate control
    - You control the sending rate
  - No guarantees on:
    - Reliability, flow control, congestion control, timing, throughput,
      security
- TCP
  - Connection oriented: setup required between client and server processes
  - Reliable transfer between client and server
  - Automated rate control:
    - Flow control: sender won't overwhelm receiver
    - Congestion control: throttle sender when the network is overloaded
  - No guarantees on:
    - Timing, minimum throughput, security

#### Transport service choices across applications

| Application            | Application Layer Protocol | Transport Protocol |
| ---------------------- | -------------------------- | ------------------ |
| File transfer/download | FTP                        | TCP                |
| E-mail                 | SMTP                       | TCP                |
| Web documents          | HTTP 1.1                   | TCP                |
| Internet telephony     | SIP, RTP, or proprietary   | TCP or UDP         |
| Streaming audio/video  | HTTP, DASH                 | TCP                |
| Interactive games      | WOW, FPS (proprietary)     | UDP or TCP         |

### Web applications

#### What paradigm?

Client-server

#### What content?

- Web pages
  - Organized as many objects which can be stored on different web servers
  - One base-HTML file, referencing other objects by URL

#### How to transfer?

HTTP connections

##### HTTP Overview

- Hypertext Transfer Protocol
- The Web's application layer protocol
- Uses client/server model
  - Stateless: server maintains no information about past client requests
- Uses TCP
- Two types of connection
  - Non-persistent HTTP
    - TCP connection opened
    - At most _one_ object sent over the connection
    - TCP connection closed
  - Persistent HTTP
    - TCP connection opened to a server
    - _Multiple_ objects can be sent over the single connection between the
      client and that server
    - TCP connection closed

###### Non-persistent HTTP

Requires two `RTT`s (round trip times) per object, one to establish connection
and another to send the object over it.

```mermaid
sequenceDiagram

Client ->> Server: Initiate TCP connection
Server ->> Client: RTT
Client ->> Server: Request object
Note over Server: + Some time to transmit file
Server ->> Client: RTT
```

> **NOTE:** There is a variation on this called **Parallel Non-persistent
> HTTP**, which uses one TCP connection and then multiple parallel requests to
> fetch referenced objects. The downside is that it uses more server (OS)
> resources.

###### Persistent HTTP

Main idea is to reuse the same TCP connection for multiple objects. Only
requires one `RTT` for each referenced object.

###### Non-persistent vs. Persistent

Example: 10 small, referenced objects

| Non-persistent HTTP | Persistent HTTP | Non-persistent HTTP (5 objects in parallel) |
| :-----------------: | :-------------: | :-----------------------------------------: |
|  2 + 2*10 = 22 RTT  | 2 + 10 = 12 RTT |              2 + 2 + 2 = 6 RTT              |
