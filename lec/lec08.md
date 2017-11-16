Transport Layer: Introduction, basics of UDP
=============================================

* We have seen that application layer operates as processes that
  communicate between hosts. Transport layer provides
  process-to-process delivery of messages. The underlying network
  layer provides host-to-host delivery of messages, without any
  reliability guarantees. So, job of transport layer is to take this
  host-to-host delivery service from the network layer, and provide
  process-to-process delivery abstraction to the application layer.

* Processes send messages via sockets. So, we can also think of the
  transport layer as providing a data delivery service from socket to
  socket.

* Some terminology: application layer "message" -> transport layer
  "segment" -> IP/network layer "datagram".

* Main functions of transport layer:

- Multiplexing and demultiplexing (mux/demux): several processes on a
  host may be sending messages. Each application process is assigned a
  port number. Transport layer takes the application layer message,
  appends port number, and passes to network layer. At the receiver
  end, it hands over message to correct application process based on
  port number. Both TCP and UDP provide this basic mux/demux
  functionality. In fact, UDP is a barebones transport layer that
  provides ONLY this functionality.

- Reliable data delivery: TCP provides reliable data delivery. That
  is, the application can be assured that the message will reach the
  other side (or it will know if lost). TCP provides reliability using
  several mechanisms, on top of a unreliable network layer.

- Congestion control: TCP ensures that data from one application does
  not swamp all the links and routers on the path. Congestion control
  is not so much a service to the application as it is a service to
  the Internet.

* What are function that transport layer cannot provide? It cannot
  provide bandwidth or delay guarantees because the underlying network
  layer does not provide guarantees. Note that IP layer does not
  provide security either, but TCP has special mechanisms (SSL) to
  provide security over unsecure transport. But bandwidth/delay
  guarantee is harder.

* Two main transport protocols: TCP and UDP.

* TCP is a connection oriented protocol, i.e., there are a few
  connection establishment messages before data can be sent from
  sender to receiver. However, this connection state resides only at
  end hosts; unlike connections in circuit switching where state is
  established at all routers on path. The side that initiates
  connection is called TCP client. The side that responds is called
  server. Data flow can happen both ways, TCP is full duplex.

* Recall from socket programming: both client and server first create
  sockets. Client sends a connect request to server IP and port. Then,
  the "TCP handshake" takes place. TCP client sends a SYN packet to
  server to initiate connection. Server responds with SYNACK. Client
  then sends SYN ACK ACK (or simply ACK), along with any data
  possibly. At this point, the accept call at the server returns,
  i.e., the server accepts the request, a new client socket is created
  for this connection, and connection is said to be established. Once
  connection is established, both client and server may send data
  through the TCP connection via the sockets. This data exchange
  translates to read/write on sockets.

* So when a packet arrives at a host, how does TCP know which client
  socket to deliver message to? TCP uses 4-tuple (source IP, source
  port, destination IP, destination port) as the demux key to identify
  appropriate socket. Why can't it just use destination IP and port?
  Because server can have several sockets open for different clients.

* How is mux/demux done in UDP? Note that there is no handshake in
  UDP. Sender and receiver applications create sockets, and fill in
  the destination port and IP in the socket structures. Then they
  simply send and receive messages using the socket handle. The source
  and destination port numbers are embedded in the UDP header. Note
  that a UDP-based server only has one listening socket, not one per
  client. When a packet reaches the destination IP address (via
  network layer), the destination port is looked up to identify the
  correct socket to deliver the message to. That is, the demux "key"
  is the 2-tuple (destination IP, destination port). 

* Why is demux key different for UDP (2-tuple) vs TCP (4-tuple)?
  Because TCP maintains a separate socket for every client-server
  communication (connection-oriented), whereas UDP server processes
  all clients via the same socket. So, next question, why is TCP
  connection oriented and why does it have to maintain a separate
  socket for every connection? Because TCP implements reliability, it
  is easier with separate sockets. For example, when a message arrives
  at a socket, it is easy to simply see the source IP/port associated
  with that socket and send ACK. If you want to send a reply to a UDP
  message, you need to extract the source IP and port from the UDP
  header for every message.

* There is not much else to UDP other than mux/demux. UDP is just
  barebones transport: mux/demux with light error detection (checksum
  over header: includes port numbers and checksum (IP addresses are from IP
  header). Lower header overhead than TCP (UDP-8 bytes, TCP-20 bytes).

* Which applications use UDP?

- DNS: simple request reply does not justify the overhead of
  connection setup of TCP.

- Multimedia applications: they have more control over what data is
  sent when. For example, if the voice sample is stale, no point in
  TCP retransmitting it.
