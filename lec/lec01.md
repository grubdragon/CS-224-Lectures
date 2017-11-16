Lecture 1: Introduction
========================

* What is the Internet made of?

- Computers connected by links via switches / routers. Multiple links
  terminate at a switch/router, which enables connectivity between
  them.

- Many switches/routers are present in an organization like IITB to
  connect all the end-hosts. End-hosts provide and consume useful
  services (e.g., web, email).

- Internet Service providers (ISPs) connect multiple such end-user
  organizations. ISP networks have several routers that provide
  interconnections between end organizations, and form the "backbone"
  of the Internet.

* Notion of IP addresses: every machine on the Internet has a unique
  address called the IP address (well, almost...). Assigned statically
  or dynamically (e.g., DHCP). Can be public or private (e.g., through
  a NAT). IP addresses have a hierarchical structure. Machines also
  have human-readable names that map to IP addresses.

* Notion of a packet: unit of communication between machines. 

* Example: what happens when you open a web page from a computer by
  typing in a URL?

- At the browser: The URL gets resolved into an IP address of the web
  server that hosts this page. The browser sends a special message
  (GET request) to the server, and the server sends back the HTML
  content. The browser then displays the page to the user.

- At the end hosts: Browser opens a "TCP connection" to the web
  server. Messages between the browser and server are sent over this
  connection. TCP (transport control protocol) is a transport
  mechanism that reliably sends information back and forth between two
  end points. TCP modules (running in the end host operating systems,
  typically) make sure that messages are delivered reliably.

- How are the TCP messages sent back and forth? The TCP data to the
  server must be routed to the server's IP address via various
  "routers" running IP (internet protocol). Your computer sends the
  packet to an IP router or default gateway that sends it to another
  router, which sends it to another router, and so on. The IP routers
  "talk" amongst themselves and so they know how to get to any
  destination IP address. Thus, several routers along the way forward
  these TCP segments from one IP hop to the next based on the
  destination server's IP address.  At the final IP hop, the IP router
  next to the wbe server hands the packet to the web server over the
  LAN.

- Several TCP segments and ACKs go back and forth this way to transfer
  the complete web page.

* The above example illustrates the concept of layering.  There are
  several functions in the networking "stack", each handled by a
  layer. A layer at one host talks to its "peer" layer at the other
  host using a set of "protocols" or conventions that they both
  follow. Every layer provides a "service" or "abstraction" that the
  higher layer uses.  For example, the application layer at client
  says GET a page and the application layer at server sends the
  page. This is the HTTP protocol. HTTP protocol uses the transport
  "service" provided by the TCP layer. That is, once it hands over the
  GET message to the TCP layer, the TCP layer takes care of actually
  transporting the bits in the message to the other end.

* Why layering? Modularity. Each layer can evolve
  independently. Easier to build large networks over differing
  technologies.

* There are 5/7 layers depending on which system you look (OSI vs
  Internet). The 5 layers are the most important ones: from top to
  bottom they are: application, transport, network, link, and
  physical.

* Encapsulation and decpasulation. Every layer adds headers to the
  bytes it gets from the layer above. The headers are useful for
  operations of that layer. These headers are consumed and removed by
  the peer at the other end.

* Where are these layers implemented? Application and transport layers
  run on end hosts - user programs, operating systems etc. IP routing
  protocols typically run in specialized routers. Link layer and
  physical layer run in network hardware (e.g., ethernet card) and
  accompanying device hardware.

* Application layer: just concerned with the application-level
  semantics. GET a web page and display it. Browser issues requests
  and displays result. Web server holds all the pages on its
  disk/memory (or computes them dynamically), and serves them when a
  request arrives. Several application layer protocols. HTTP is the
  most popular one, but several others exist (SMTP for email), P2P
  protocols etc. 

* The transport layer (usually TCP, but sometimes UDP/RTP etc) is
  concerned with the transfer of the bits and bytes that make up the
  application data. This layer takes the application layer "messages",
  makes them into TCP "segments", transmits them at a suitable pace
  that the network can handle, makes sure they are received and
  acknowledged by the other end host, retransmits them when
  needed. TCP does not care about what the data is (that's the job of
  the application layer), or how the data gets there (thats the job of
  the lower layers). That is, TCP provides a reliable in-order data
  stream abstraction. TCP performs congestion/flow control to ensure
  that it sends data at an optimum rate that the underlying network
  can handle (not too slow, not too fast). TCP is the most common, but
  UDP is used when reliability is not needed. RTP is used for real
  time services like VOIP applications. There are several variants of
  TCP also.

* The application and transport layers are end-to-end, run at only the
  end hosts (in operating systems and user programs), they don't care
  what the network does in between.

* Next is the network or IP layer or Layer 3. TCP segments are
  converted to "IP datagrams". The datagram has a source and
  destination IP address. The job of the network layer is to route the
  datagram to its destination IP address. Each IP router along the way
  looks up the destination address and decides which link it should
  forward the packet next.

* Do IP routers have separate entries for every IP address? No. IP
  addresses have hierarchical structure. An aggregate of several IP
  addresses is called an IP prefix. Every IP router has several
  incoming links/ports (port is physical attachment point for an
  incoming/outgoing link) and several outgoing links/ports. For every
  incoming packet, IP router looks up a "forwarding table" and decides
  which port to send packet on. A forwarding table maps IP prefixes to
  next hop IP addresses (and hence to outgoing port that connects to
  the next hop IP router).

* Forwarding tables are computed by routing protocols. Routing
  protocols help compute "routes" which decide how to forward a
  packet. A route is a specification of the path to reach a certain IP
  prefix. Minimally, for every IP prefix, its route tells us the next
  hop IP router on the path and the distance/metric to get to the
  destination along this path. Optionally, it can provide more details
  about the path. For every IP prefix, routing protocols gather
  information about several routes, pick the best one by some metric
  (e.g., shortest path), and insert this best route into the
  forwarding table. A routing table is this a superset of the
  forwarding table.

* Intradomain routing (within an organization) and
  interdomain/wide-area routing (across the internet) are two
  different things. The former just deals with shortest paths while
  the latter has to deal with policy as well (i.e., the guy on the
  shortest path may not carry your data if he has no financial
  relationship with you). So separate routing protocols for intra and
  interdomain routing. We will study this in detail later.

* Next is link layer (layer 2). Deals with delivering a link layer
  "frame" over an IP hop or "link", for example, through a shared LAN
  or a switch or a point-to-point fiber link or a wireless
  link. Finally, the physical layer deals with transmitting the actual
  bits over a physical medium like a copper wire, optical fiber, or
  wireless radio. We won't deal much with physical layer in this
  course.

* Note that several link layer technologies can be connected together
  via IP. A host connected to an Ethernet network can communicate with
  another host connected to a wireless network, as long as they both
  speak IP and are connected by a path of IP routers. That is, the
  links between any two IP hops can be very different. Thus the
  Internet has grown as a network of networks due to the unifying IP
  protocol.

