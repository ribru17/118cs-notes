# CSS 118 Notes

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

### How are access networks constructed in the first place?

- Signals carried by EM waves (radio) or within wires (Ethernet)
- Network core will move bits around access networks to the destination, using
  routers
- Router functions:
  - Routing: determines source-destination route taken by _data routing
    algorithms_
  - Forwarding: moves data from router's input to appropriate router output

### How can the internet grow so big?

- Connecting all access ISPs to each other does not scale ($O(n^{2})$
  connections)
- Connect all to a global ISP is bad because if the main fails, everything fails
  - Also bad for business as one company will control all traffic
- Connecting to multiple global ISPs reduces work load and is more reliable
- Creating a hierarchical structure is even better as it reduces load and allows
  for regional ISPs

### What is the internet software architecture?

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

### TCP: Transmission Control Protocol

- A connection is set up between client and server
- Reliable data transfer
  - Guaranteed deliveries of all data
  - No duplicate data will be delivered
- Data will always be received in the order it is sent
- Full-duplex byte stream (in two directions simultaneously)
- Regulated data flow with flow control and congestion control

### UDP: User Data Protocol

- Basic transmission service
  - No connection needed
  - Unit of data transfer: datagram (variable length)
- No reliability guarantee
- No ordered deliver guarantee
- No flow control / congestion control

### Socket APIs

- Socket:
  - An endpoint in inter-process communication across a computer network
  - TCP sockets have tuples of `<ip_addr:port>`
  - On Linux and MacOS, sockets are like files (socket handles are file
    descriptors)
  - TCP sockets provide `write()` and `read()` for sending and receiving data
- Socket port numbers
  - Smaller numbers are reserved and only accessible by super-users

### Caveat: Byte ordering matters

- Data that is sent in little-endian must be read as such, and vice versa
- The API defines methods to convert to proper network ordering
  - This only works for 16- and 32-bit data, more complex data types must be
    taken into account differently

<!-- Lecture 2 -->

### Packet switching

- **Store-and-forward** operations inside the network
- Sending host
  - Takes application message

### Circuit switching

- End-end resources are allocated and reserved for a voice call between source
  and destination
- `FDM`
  - `FDM` does one user at a time
  - `TDM` alternates users over time

### Packet switching vs. Circuit switching

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

### Internet software appears in the form of protocols

- Protocols control sending and receiving messages
  - E.g. HTTP, Skype, TCP, etc.
- Protocols are software pieces
  - There are 100s\~100s of them
- Organized in layers which cannot only talk to the immediate upper or lower
  layer

### Layering

- Modularization allows for easier maintenance
- Different layers have their own protocols

#### Layers

- **Application:** Supports network applications
  - FTP, SMTP, HTTP
- **Transport:** Process-to-process data transfer
  - TCP, UDP
- **Network:** Routing of datagrams from source to destination
  - IP, routing protocols
- **Link:** Data transfer between neighboring network elements
  - Ethernet, Wi-Fi, PPP
- **Physical:** Bits "on the wire"

### How to evaluate the internet

Performance is gauged with three metrics:

#### Throughput

- Throughput is the rate (bits/time) at which units are transferred
- End-to-end throughput is equivalent to the bottleneck throughput

#### Packet Loss

- Queue preceding link in buffer has finite capacity
- Packet arriving to full queue dropped
- Lost packet may be retransmitted by previous node, by source end system, or
  not at all

#### Delay

- Host sending function:
  - Takes application message
  - Breaks it into smaller chunks, called packets, of length `L` bits
  - Transmits packets across network at rate `R`
  - Packet transmission delay $= \frac{L}{R}$

##### Packet Delay

If arrival rate (in bits) to link exceeds transmission rate for a period of
time:

- Packets will queue and wait to be transmitted on the link
- Packets can be dropped (lost) if memory (buffer) fills up

###### Four Sources of Packet Delay

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
| 2 + 2\*10 = 22 RTT  | 2 + 10 = 12 RTT |              2 + 2 + 2 = 6 RTT              |

> **NOTE:** Each requires an initial 2 `RTT` to get data from the DOM.

For the last case (five objects in parallel):

1. First two `RTT`s are for setting up connection and requesting DOM data.
2. Next two `RTT`s are for requesting the first five objects (limit determined
   by parallel requests).
3. Last two `RTT`s are for requesting the last five objects.

<!-- Discussion 2 -->

### Practice

#### a)

**Q:** What is the propagation delay of a packet of length 1000 bytes sent over
a link of distance 2500km, propagation speed $2.5x10^8$ m/s, and transmission
rate 2 Mbps?

**A:** `TBD`

#### b)

**Q:** Suppose host A wants to send a large file to host B. The path from Host A
to Host B has three hops on the link, the data rates are: $R_1$ = 500Kbps, $R_2$
= 2Mbps, and $R_3$ = 1Mbps.

- Assuming no other traffic in the network, what is the throughput for the file
  transfer?
  - **A:** 500 kbps
- Suppose the file is 4 million bytes. Roughly how long will it take to transfer
  the file to Host B?
  - **A:** (4 / 0.5) \* 8 = 64 seconds
    - We multiply by 8 due to the conversion from bytes to bits
- Suppose the file is 200 bytes and it is segmented into two 100 byte packets.
  Also $R_2$ transfer speed has been reduced to 100Kbps. What is the queuing
  delay for the second packet at host A, the first node, and the second node?
  - **A:** At the sender node, it is 1.6 ms (the time it takes for the first
    packet to be sent across $R_1$). The second packet will have queuing delay
    at the first node since $R_2$ is now the bottleneck (first packet will not
    finish transmitting by the time the second packet arrives). At the first
    node the queuing delay will be 6.4ms. The second node will have no queuing
    delay as the first packet will be long gone before the second packet arrives
    at the link.

#### c)

**Q:** When you need to retrieve a web document from an HTTP server with an
_unknown_ IP address, what protocol should you use?

**A:** You must use DNS for if you do not know the IP address (for the
application layer). For the transport layer, you use UDP with DNS and TCP for
HTTP.

### Application Layer Models

- Application architectures
  - Client-server
  - Peer-to-peer
  - Hybrid
    - Skype (TCP & UDP)
    - `GTalk` (TCP & UDP)

### Application Layer Protocols

- HTTP: stateless protocol on top of TCP
  - HTTP is fundamentally based on pull model
  - Persistent vs. Non-persistent
  - Method types: `GET`, `HEAD`, `POST`, `PUT`, `DELETE`, Conditional `GET`
  - What if we want a stateful service (e.g. a web cart that remembers your
    items)?
    - Web caches (proxy server) with cookies e.g.

<!-- Lecture 4 -->

### HTTP Messages

There are two types, both with headers terminated by an empty line of `\r\n`:

#### Request Message

- In ASCII (human-readable format)
- Take various forms
  - `POST`
    - Web pages often includes form input
    - User input sent from the client to the server in the body of the `POST`
      request
  - `GET`
    - Can include user data in the URL field of the message following a `?`
      - E.g. `www.somesite.com/animalsearch?monkeys&banana`
  - `HEAD`
    - Requests headers (only) that would be returned _if_ the specified URL were
      requested with a `GET` method
  - `PUT`
    - Uploads a new file (object) to the server
    - Completely replaces file that exists at the specified URL with content in
      the body of the `POST` request message

#### Response Message

- Similar looking body to request message
- Contains status codes
  - `200`: OK
  - `301`: Moved Permanently
  - `400`: Bad Request
  - `404`: Not Found
  - `505`: Unsupported HTTP Version

### HTTP Web Advanced Features

#### Web cookies

- **Recall:** HTTP `GET` / response interaction is _stateless_
  - No need for client / server to track state of multi-step exchange
  - No need for client / server to recover from partially completed transactions
- Cookies maintain user / server state

#### Web caches (proxy servers)

- **Goal:** satisfy client request without involving origin server
- User configures browser to point to a web cache (i.e., proxy server)
- Browser sends all HTTP requests to the cache:
  - _If_ the object is in the cache: Cache returns the object to the client
  - _Else_ the cache requests the object from the origin server, caches the
    received object, then returns the object to the client

#### Conditional `GET`

- **Goal:** Don't send object if cache has up-to-date cached version
  - No object transmission delay
  - Lower link utilization
- Cache: specify date of cached copy in HTTP request
  - `if-modified-since: <date>`
- Server: response contains no object if cached copy is up-to-date
  - `HTTP/1.0 304 Not Modified`

#### HTTP/2

- **Goal:** Decrease delay in multi-object HTTP requests
- HTTP/1.1 introduced multiple, pipelined `GET`s over a single TCP connection
- HTTP/2 migrates head-of-line blocking
  - Smaller objects will not have to wait behind previous, larger objects which
    are taking longer to download
- Allows for this by dividing objects into frames and interleaving the frame
  transmission
  - Smaller objects are delivered very quickly and initial, large objects are
    slightly delayed
- Recovery from packet loss still stalls all object transmissions
- No security via vanilla TCP connection

#### HTTP/3

- **Goal:** Same as HTTP/2
- Adds security, per-object error- and congestion-control (more pipe-lining)
  over UDP
  - More on this in the transport layer section

### Internet Application: E-mail

- Mail servers
  - Includes a mailbox containing incoming messages from the user
  - Also includes a message queue of outgoing (to be sent) mail messages
- App-layer protocol: SMTP (simple mail transfer protocol)
  - Used between mail servers to send email messages
  - Client: sending mail server
  - "Server": receiving mail server
- Uses TCP for reliable transfer
- Three transfer phases
  - Handshaking (greeting)
  - Transfer of messages
  - Closure

<!-- Lecture 5 -->

### Domain Name System (DNS)

Provides lookup from domain name (e.g. `www.google.com`) to IP address
(`173.194.204.99`)

#### DNS Services

- Host name to IP address translation
- Host aliasing
  - Canonical, alias names
- Mail server aliasing
- Load distribution
  - Replicated web servers: many IP address correspond to one name

> **Q:** Why not centralize DNS?

- Single point of failure
- Traffic volume
- Distant centralized database
- Maintenance
- Doesn't scale

#### DNS Structure

- Internet distributed database implemented in hierarchy of many name servers
- Application-layer DNS protocol: hosts, name servers communicate to resolve
  names (address / name translation)
  - Core internet function implemented as application layer protocol

#### Key Concepts

- Names are hierarchical
  - No flat names (e.g. `university_ucla_cs_kiwi`) for hosts
  - Only hierarchical names (e.g. `kiwi.cs.ucla.edu`)
    - Highest hierarchy: `edu`, `com`, `gov`, `org`, ... `us`, `jp`, `fr`, ...
    - Next-level hierarchy: `ucla`, `mit`, ... `google`, `ibm`, ... `ca`, ...
    - All are under certain root DNS servers
  - Hierarchical names form the _hierarchical name space_
- _Name servers_ (that store and resolve names) are also organized into a
  hierarchy
  - Each name server handles a small portion of the name space hierarchy
- _Name resolution_ follows the hierarchy to resolve names to IP addresses

#### Root DNS servers

- Official, contact-of-last-resort by name servers that cannot resolve name
- _Incredibly important_ internet function
  - The internet could not function without it!
  - DNSSEC
    - Provides security (authentication and message integrity)
- ICANN (Internet Corporation for Assigned Names and Numbers) manages root DNS
  domain

#### Authoritative servers

- TLD (top-level domain) servers
  - Responsible for `.com`, `.org`, etc.
  - Part of an authoritative registry
- Organization's own DNS servers
  - Can be maintained by organization or service provider

#### Local DNS name servers

- Do not strictly belong to the hierarchy
- Each ISP (residential, company, university) has one
  - Also called "default name server"
- When a host makes a DNS query, it is sent to its local DNS server
  - Has a local cache of recent name-to-address translation pairs (but may be
    out of date!)
  - Acts as a proxy, forwards query to hierarchy

#### DNS Name Resolution

##### Iterated Query

Contacted server replies with the name to contact

> "I don't know this name, but ask this server:"

##### Recursive Query

- Puts burden of name resolution on the contacted name server
- Heavy load at upper layers of the hierarchy?

#### DNS Records

- Distributed database storing resource records (RR)
- RR format: `(name, value, type, ttl)`
  - `ttl` is _time to leave_
- Entries have different types
  - **`A`**
    - `name` is host name
    - `value` is IP address
  - **`CNAME`**
    - `name` is alias name for some "canonical" (the real) name
      - E.g. `www.ibm.com` is really `servereast.backup2.ibm.com`
    - `value` is the canonical name
  - **`NS`**
    - `name` is the domain (e.g. `foo.com`)
    - `value` is the host name of the authoritative name server for this domain
  - **`MX`**
    - `value` is the name of the mail server associated with `name`

#### Caching, Updating DNS Records

- Any name server can cache the record once it learns
  - Cache entries timeout (disappear) after some time (TTL)
  - `TLD` servers are typically cached in local name servers
    - Thus root name servers are not often visited
- Cached entries may be out of date (best effort name-to-address translation)
  - If name host changes IP, it may not be known internet-wide until every TTL
    is expired!

#### DNS Protocol

Client-server based: DNS _query_ and _reply_ over UDP/TCP

- Message header:
  - **Identification**: 16 bit number for query, the query reply uses the same
    number
  - **Flags**:
    - Query or reply
    - Recursion desired
    - Recursion available
    - Reply is authoritative
- _Query_ and _reply_ messages have the same format

<!-- Lecture 6 -->

### Video Streaming

- Major consumer of internet bandwidth

#### What is video?

- Sequence of images displayed at a constant rate
  - Each image is an array of pixels arranged by bits
- Stored using redundancy _within_ and _between_ images to decrease the size
  needed to encode the image
  - Spatial (within image)
  - Temporal (from one image to the next)
- CBR (constant bit rate): video encoding rate is fixed
- VBR (variable bit rate): video encoding rate as amount of spatial, temporal
  storage changes

#### DASH

- Dynamic, Adaptive Streaming over HTTP
- Server:
  - Divides video file into multiple chunks
  - Each chunk is stored, encoded at different rates
  - _Manifest file_: provides URLs for different chunks
- Client:
  - Periodically measures server-to-client bandwidth
  - Consulting manifest, requests one chunk at a time
    - Can choose different encoding rates at different times, depending on
      current bandwidth
  - Client is "intelligent"; it determines:
    - **When** to request a chunk (so that buffer starvation or overflow does
      not occur)
    - **What encoding rate** to request (higher quality when more bandwidth is
      available)
    - Where to request a chunk (can request from a URL server that is close to
      the client or one that has high bandwidth)

#### CDNs (Content Distribution Networks)

- **Challenge:** How do we stream content at large scales (1M or more
  simultaneous users)?
  - **Naive solution:** Have a single, large "mega server."
    - This is a single point of failure
    - It would be a point of network congestion
    - There would be a long path to distant clients
    - Multiple copies of a video would be sent through outgoing links
    - This is a non-scalable solution
  - **Real-world solution:** Store and serve multiple copies of videos at
    multiple geographically distributed sites (CDNs)
- How it works
  - CDN servers store multiple copies of video content at different nodes
  - Users request content and are redirected to the nearest node (but may select
    a different one if the nearest node is congested)

## Transport Layer

### Transport Services

- Provide _logical communication_ between application processes running on
  different hosts
- Work for two end hosts only
  - Not internet routers
- Target _one-to-one_ communication initially
  - Multi-party group communication was added later

### Transport Protocols

- Transport services are realized by internet transport protocols
- Protocol actions in end systems:
  - Sender: breaks application messages into segments, pass to network layer
  - Receiver: reassembles segments into messages, passes to application layer
- Only two basic transport protocols: TCP and UDP

#### Transport Layer: Sender Side

- Sender is passed an application-layer message
- It determines segment header fields values
- It then creates the segment and passes it to IP

#### Transport Layer: Receiver Side

- Receiver gets segment from IP
- It checks header values
- It then extracts the application-layer message
- Finally it demultiplexes the message up to the application via a socket

### Multiplexing

Multiplex (combine) to-be-sent-out data from different processes together. At
the sender, it handles data from multiple sockets, adds transport header, and
passes them all to network layer.

### Demultiplexing

De-aggregate received data streams and relay all data to the intended receiving
application process. At the receiver, it uses header info to deliver received
segments to the correct socket.

#### How it works

- Uses IP addresses and port numbers to direct a segment to its appropriate
  socket
  - Host receives IP datagrams
  - Each datagram has a source IP address and a destination IP address
  - Each datagram carries one transport-layer segment
  - Each segment has a source port number and a destination port number

There are two types of demultiplexing:

#### Connectionless Demultiplexing

- Used by UDP
- On sender side:
  - When creating a socket, must specify host-local port number
  - When creating a datagram to send into the UDP socket, must specify
    - Destination IP address
    - Destination port number
- When a host receives the UDP segment:
  - It checks the destination port number in the segment
  - It directs the UDP segment to the socket with that port number
  - IP/UDP datagrams with the same destination port number will be directed to
    the same socket at the receiving host
    - **NOTE:** They may have different source IP address and/or port numbers.

#### Connection-oriented Demultiplexing

- Used by TCP
- TCP socket identified by a tuple:
  - Source IP address
  - Source port number
  - Destination IP address
  - Destination port number
- Demultiplexing: Receiver uses _all four values_ to direct a segment to the
  appropriate socket
- Servers may support many simultaneous TCP sockets:
  - Each socket is identified by its tuple
  - Each socket is associated with a different connecting client

### UDP: User Datagram Protocol

- Simplest, "bare bones" internet transport protocol
- "Best effort" service, UDP segments may be:
  - Lost
  - Delivered out-of-order to an application
- _Connectionless_
  - No handshaking between UDP sender and receiver
  - Each UDP segment is handled independently from others
- Benefits?
  - No connection establishment (less `RTT` delay)
  - Simple: no connection state
  - Small header size
  - No congestion control
    - Can fire away information as fast as it wants
    - Can function in the face of congestion
- UDP headers are 8 bytes only <!-- Lecture 7 -->
  - Contain:
    - Source port number (for demultiplexing)
    - Destination port number (for demultiplexing)
    - Length (in bytes) of the UDP segment (including the header)
    - Checksum

#### Checksum

- Exists to detect errors in the transmitted segment
- Sender
  - Treat contents of UDP segment (including header fields and IP addresses) as
    a sequence of 16-bit integers
  - Checksum is the sum of the segment content
    - This value is what is stored in the checksum header field
- Receiver
  - Compute checksum of received segment
  - Check if this value is equal to the given checksum value:
    - **Not Equal:** Error detected
    - **Equal:** Assume no errors (though there still may be)

### TCP: Transmission Control Protocol

- Point-to-point
  - One sender, one receiver
- Reliable, in-order data transfer
  - No "message boundaries"
- Full duplex data transfer
  - Bidirectional data flow in the same connection
  - **`MSS`**: Maximum segment size
- Connection-oriented
  - Handshaking (exchange of control messages) initializes sender and receiver
    state before data exchange
- Flow controlled
  - Sender will not overwhelm the receiver
- Congestion controlled
  - Sender will not overwhelm the internet
- Four essential features:
  - Reliable data transfer
  - Connection setup and close-down
  - Flow control
  - Congestion control

#### Reliable Data Transfer

- Important for internet data delivery
  - We also have to deal with imperfect packet deliver
    - With bit errors, dropping packets, out-of-order delivery, duplicate
      copies, long delay, ...
- Built using a reliable data transfer channel (which is really an abstraction
  over the Internet's unreliable data transfer channel)
- Develop sender and receiver sides of the protocol
- Consider only unidirectional data transfer (even though control info will flow
  both ways!)
- Use finite state machines (`FSM`) to specify sender, receiver
- `FSM` for `rdt1.0` (reliable data transfer 1.0) over a reliable channel
  - Underlying channel is perfectly reliable
    - No bit errors, no loss of packets
  - _Separate_ `FSM`s for sender and receiver
    - Sender sends data into the underlying channel
    - Receiver reads data into the underlying channel
  - **Starting Scenario:** Stop and wait
    - Packets are sent one at a time, with the sender not sending the next
      packet until the previous has been received and acknowledged by the
      receiver
- `rdt2.0`: Considers channel with bit errors (unreliable channel)
  - How do we _detect_ errors?
    - Checksum algorithm
  - How do we _recover_ from detected errors?
    - Acknowledgments (`ACK`s): receiver explicitly tells the sender that packet
      is received
    - Negative acknowledgments (`NAK`s): receiver explicitly tells the sender
      that packet has errors
    - Sender retransmits upon receiving a `NAK`
  - **FLAWS:** What happens if `ACK`/`NAK` is corrupted?
    - Sender doesn't know what happened on the receiver's end, and we can't just
      resend a packet.
    - How do we handle duplicates?
      - Sender retransmits if `ACK`/`NAK` is corrupted
      - Receiver discards (doesn't send to application layer) the duplicate
        packet
      - Sender adds _sequence number_ to each packet.
        - We need this so the client knows whether the packet is a duplicate and
          it can discard it
        - This is a characteristic of `rdt2.1`
        - This sequence number only has to be one bit!
- `rdt2.0`
  - Removes `NAK`
  - `ACK` is _explicitly_ set as the sequence number of the received packet
  - The negative acknowledgments becomes the `ACK` of the previous packet
- `rdt3.0`: Channels with errors _and_ loss
  - Packets can be lost or dropped
  - How do we stop this?
    - Sender waits a "reasonable" amount of time for an `ACK`
    - If no `ACK` is reached in time, the sender will retransmit
    - If the `ACK` was just delayed, there will be a duplicate send but this
      doesn't matter since the sequence number already handles this

Table summary of reliable data transfer:

| Version  | Channel              | Mechanism                                                                                |
| -------- | -------------------- | ---------------------------------------------------------------------------------------- |
| `rdt1.0` | Reliable channel     | Nothing                                                                                  |
| `rdt2.0` | Bit errors (no loss) | Error detection via checksum, receiver feedback (`ACK`/`NAK`), retransmission upon `NAK` |
| `rdt2.1` | Same as 2.0          | Adds sequence number for each segment                                                    |
| `rdt2.2` | Same as 2.0          | Duplicate `ACK` = `NAK`                                                                  |
| `rdt3.0` | Bit errors + loss    | Retransmission upon timeout                                                              |

What if the first packet is a `NAK`? Then that means we didn't establish a
proper connection in the first place (first packet is usually used for this) so
in this case we will just try to reestablish.

<!-- Lecture 8 -->

#### Pipeline sending

There are two main types of pipelined protocols.

##### Go-Back-N

- Receiver buffer size = 1 packet
- Sender can have up to `N` unacknowledged packets in the pipeline
- Receiver only sends a _cumulative `ACK`_
  - Doesn't `ACK` packet if there's a gap
- Sender has timer for oldest unacknowledged packet
- Better for smaller buffers because the receiver discards all successful
  packets that come after one lost packet
  - Sender resubmits the initial lost packet and again pipelines those that
    follow

##### Selective Repeat

- Receiver buffer size = N packets
- Sender can have up to N unacknowledged packets in the pipeline
- Receiver sends an _individual `ACK`_ for each packet
- Sender maintains a timer for each unacknowledged packet
  - When the timer expires, retransmit only that unacknowledged packet
- Packets that follow a lost packet are acknowledged and must remain in the
  buffer until the timeout for the lost packet hits, and the sender resends the
  initial lost packet. After this point all following packets are delivered to
  the application, and the original `ACK` of the lost packet is finally sent.

#### TCP Segment Structure

- Source and destination port numbers (each 16 bits)
- Sequence number (32 bits)
- Acknowledgment number (32 bits)
- Length of header, unused bits, congestion notifications, TCP options, `ACK`
  bit, `RST`, `SYN`, and `FIN` connection management, flow control, receive
  window (16 bits, number of bytes the receiver is willing to accept)
- Checksum (16 bits), `URG` data pointer (16 bits)
- TCP options
- Application data

<!-- Lecture 9 -->

#### TCP Timeout Prediction

- Timeout interval is the `EstimatedRTT` plus a safety margin
  - A larger variation in `EstimatedRTT` demands a larger safety margin
  - $\text{TimeoutInterval} = \text{EstimatedRTT} + 4 \cdot \text{DevRTT}$
- `DevRTT` is the safety margin (`EWMA` of `SampleRTT` deviation from
  `EstimatedRTT`)
  - $\text{DevRTT} = (1 - \beta) \cdot \text{DevRTT} + \beta \cdot
    |\text{SampleRTT} - \text{EstimatedRTT}|$
  - $\text{EstimatedRTT} = (1 - \alpha) \cdot \text{EstimatedRTT} + \alpha \cdot
    \text{EstimatedRTT}$

#### Summary on TCP Reliable Data Transfer

- Header fields used: sequence number, `ACK` number, `ACK` flags, checksum, ...
- Operations at the sender and the receiver
  - Fast retransmit is used to enable retransmission (faster than timeout)
- Timeout estimation algorithm
  - Based on `RTT` samples (the `RTT` between a sent segment and its received
    `ACK` at the sender)
  - Estimated using both mean and variance
  - Use Karn's algorithm to take segments (ignore all ambiguous segments)
  - Exponential back-off for multiple retransmission timeouts

#### TCP Connection Management

- 3-way handshake, connection establishment
  - Client chooses initial sequence number $x$, sends TCP `SYN` message
    - `SYN` bit = `1`, sequence = $x$
  - Server chooses initial sequence number $y$, sends TCP `SYNACK` message,
    acknowledging `SYN`
    - `SYN` bit = `1`, sequence = $y$, `ACK` bit = `1`, `ACK` number = $x + 1$
  - Client receives `SYNACK`, indicating the server is live. It sends `ACK` for
    `SYNACK`; this segment may contain client-to-server data.
    - `ACK` bit = `1`, `ACK` number = $y + 1$
  - **NOTE:** $x$ and $y$ are chosen randomly, not just always at zero.
- 4-way handshake, connection close-down
  - Client can no longer send data, but can receive it; sends `FIN` bit.
  - Server acknowledges.
  - **\*SERVER WILL CLOSE AFTER A TIMER\***
  - Server again sends, this time `FIN` bit
  - Client sends a final acknowledgement (signaling) but without data. This is
    not a connection.

#### TCP Flow Control

- TCP receiver "advertises" free buffer space in `fwnd` field in TCP header
  - `RcvBuffer` size set via socket options (typical default is 4096 bytes)
  - Many operating systems auto-adjust `RcvBuffer`
- The sender limits the amount of unacknowledged "in-flight" data to received
  `rwnd`
- This guarantees the receive buffer will not overflow
- Sender does not transmit faster than receiver's available buffer
  - It sends its window size as $N \le \text{Receiver's advertised buffer size}$

#### TCP Congestion Control

- Congestion
  - Informally: too many sources sending too much data too quickly for the
    network to handle
  - Manifestations: long delays (queuing in router buffers), packet loss (buffer
    overflow)
  - Different from flow control: a _top-10_ problem!
- TCP increases its sending rate until packet loss occurs at some router's
  output: the _bottleneck link_

Key issues in TCP congestion control

---

##### How to detect internet congestion in TCP

- _**Key assumption made:**_ All lost TCP segments are due to internet
  congestion
  - Negligible transfer corruption errors (because link quality is generally
    good)
- Detect congestion == detect segment loss in TCP
- How do we detect this?
  - TCP timeout event _(or)_
  - TCP sender receives three duplicate `ACK`s (a la fast retransmit)

##### How to react to congestion or non-congestion

- <u>Golden rule:</u> `AIMD` rule (additive increase, multiplicative decrease)
  - Additive increase: increase sending rate linearly until congestion/loss is
    detected
  - Multiplicative decrease: reduce sending rate by 50% after loss is detected
  - Results in "sawtooth" behavior

##### How to improve TCP throughput while controlling congestion

<!-- Lecture 10 -->

- `AIMD` rule has problems
  - We need to jump-start the initial sending rate
    - **Idea:** Exponential sending increase
  - Decreasing the sending rate by 50% is still too slow
    - **Idea:** Reset sending to smallest value upon heavy congestion

##### Congestion Control Algorithms

> **NOTE:** `cwnd` is the congestion window, in bytes.
>
> And as a reminder, **`MSS`** is the maximum segment size.

###### TCP Slow Start Algorithm

- **Initialization:** When TCP starts, $\text{cwnd} \le 2 \text{MSS}$
  - Typically we set it to $1 \text{MSS}$.
  - Example: $\text{MSS} = 500\text{B}$, $\text{RTT} = 200\text{ms}$
    - Then initial rate is 20 kbps
- Remark: Available bandwidth may be $\gg \text{MSS}$
  - It is desirable to quickly ramp up to the respectable rate
- **When to stop:** Increase sending rate exponentially quickly until `cwnd`
  reaches a threshold (slow-start-threshold `ssthresh`)
- Example: Start sending one segment, then two, then four, etc.

###### Congestion Avoidance Algorithm

- Increase `cwnd` by 1 `MSS` per `RTT` until congestion (loss) is detected

###### Fast Retransmit, Fast Recovery

- Observe the `AIMD` rule by reducing when loss is detected by receiving third
  duplicate TCP `ACK`.
- **Fast retransmit:** When receiving multiple duplicate `ACK`s
  - Use 3 duplicate `ACK`s to infer packet loss
  - Retransmit the lost segment
  - Set $\text{ssthresh} = max(\frac{\text{cwnd}}{2}, 2\text{MSS})$
  - $\text{cwnd} = \text{ssthresh} + 3 \text{MSS}$
- **Fast recovery:** Governs the transmission of new data until a non-duplicate
  `ACK` arrives
  - $\text{cwnd} = \text{cwnd} + 1 \text{MSS}$

###### Retransmission Timeout

- When retransmission timer expires
  - $\text{ssthresh} = max(\frac{\text{cwnd}}{2}, 2 \text{MSS})$
    - `cwnd` (in bytes) should be in-flight size, to be more accurate
  - $\text{cwnd} = 1 \text{MSS}$
- Resetting because heavy loss/congestion was detected

> **WARNING!** Don't increase `cwnd` for only 1 or 2 duplicate `ACK`s. This
> allows for transient out-of-order delivery.

<!-- Lecture 11 -->

## Network Layer: The Data Plane

### Network Layer Services

- Now hop by hop (not end-to-end anymore)
- There are two key functions:
  - **Forwarding**
    - Move packets from a router's input link to an appropriate router output
      link
    - Analogous to getting through a single interchange in a trip
  - **Routing**
    - Determine route taken by packets from source to destination
    - Analogous to planning a trip from a source to a destination
- Data plane
  - Data forwarding
  - _Local_, per-router function
  - Determines how the datagram arriving on router input port is forwarded to
    the router output port
- Control plane
  - Routing
  - _Network-wide_ logic
  - Determines how datagram is routed among routers along end-to-end path from
    the source host to the destination host
  - Two control-plane approaches:
    - _Traditional routing algorithms:_ Implemented in routers
    - _Software-defined networking (`SDN`):_ Implemented in remote servers

<!-- Lecture 12 -->

### IP Protocol

#### IP header fields

- Protocol version number
- Header length (bytes)
- "Type" of service
- Datagram length (bytes)
- TTL: remaining max hops (decremented at each router)
- Upper layer protocol (TCP or UDP)
- Header checksum
- Source IP address (32 bits)
- Destination IP address (32 bits)
- Options (if any) (e.g. time-stamp, record route taken)
- Payload data

#### IP addressing

- IP address: 32-bit identifier associated with each network (host or router)
  interface
- Network interface: connection between host/router and physical link
  - Routers typically have multiple interfaces
  - Host typically has one or two interfaces (e.g. wired (Ethernet), wireless)

#### Subnets

- What is a subnet?
  - Device interfaces that can physically reach each other without passing
    through an intervening router
- IP addresses have structure:
  - **Subnet part**: Devices in the same subnet have common high order bits
  - **Host part**: Remaining low order bits

#### `CIDR`

- Classless Inter-Domain Routing
  - Subnet portion of address of arbitrary length
  - Address format: `a.b.c.d/x`, where `x` is the number of bits in the subnet
    portion of the address

#### DHCP

- Dynamic Host Configuration Protocol
- Goal: Host _dynamically_ obtains IP address from network server when it joins
  the network
  - Can renew its lease on address in use
  - Allows reuse of addresses (only hold address while connected/on)
  - Support for mobile users who join/leave network
- How to broadcast?
  - Use the special address for IP broadcast: `255.255.255.255`
- How to start without a valid IP address?
  - Use `0.0.0.0` (reserved, special-purpose address for "this host, this
    network")
- Can return more than just allocated IP address on a subnet:
  - Address of first-hop router for client
  - Name and IP address of DNS server
  - Network mask (indicating network versus host portion of address)

#### NAT

- Network address translation
- All devices in local network share just one `IPv4` address as far as the
  outside word is concerned
  - All datagrams _leaving_ the local network have the same NAT IP address, but
    _different_ source port numbers
- Implementation: NAT router must (transparently):
  - Replace source IP address, port number of every outgoing datagram to the NAT
    IP address and new port number
    - Remote clients/servers will respond using the NAT IP address and the new
      port number as the destination address
- Has been controversial:
  - Routers "should" only process up to layer 3
  - Address "shortage" should be solved by `IPv6`
  - Violates end-to-end argument (port number manipulation by network-layer
    device)
  - NAT traversal: what if the client wants to connect to a server behind NAT?
