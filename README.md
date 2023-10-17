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
