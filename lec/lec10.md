More on TCP
=============

* TCP congestion control summary. : understand 3 states of TCP (slow
  start, congestion avoidance, fast recovery) and transitions between
  them. Transitions are triggered by dupacks, new acks, or timeouts.

* In slow start:

- On every new ACK: cwnd = cwnd + MSS
- 3 dupacks: go to fast recovery
- Timeout: restart slow start.
- cwnd >= ssthresh: go to congestion avoidance.

* In congestion avoidance:
- On every new ACK: cwnd = cwnd + MSS/(cwnd/MSS)
- 3 dupacks: fast recovery
- Timeout: go to slow start.

* In fast recovery:
- New ACK: set cwnd to half the value of cwnd when entering fast recovery. go to congestion avoidance.
- Dupack: continue in fast recovery. send a new segment for every segment that has reached receiver.
- Timeout: go to slow start.

* Every time we have a timeout, we set ssthresh to half the cwnd where
  timeout occurred. ssthresh = cwnd/2, and then set to cwnd = 1 MSS.

* Everytime we have 3 dupacks also, we set ssthresh to half the cwnd
  value when dupacks occurred. That is, we do ssthesh = cwnd/2 before
  going into fast recovery. 

* Finally, note that even if the network allows, sender must sometimes
  slow down when receive buffer is full. This is called flow
  control. Receiver indicates the space available in its receive
  buffer with every ack. Sender always takes min(cwnd, receive window)
  when deciding how many bytes should be outstanding.

* Note: do not confuse congestion control (adjusting cwnd in response
  to packet loss and congestion in network) and flow control
  (adjusting sending rate so that we do not overwhelm the receive
  buffer).

* Older versions of TCP (TCP Tahoe) went into slow start even for
  dupack triggered losses. Next version of TCP (TCP Reno onwards) use
  fast recovery. TCP New Reno improves over Reno in that it exists
  fast recovery only after all the outstanding segments at the time of
  entering fast recovery have been acked. It retransmits in response
  to "partial acks". That is, when you do fast retransmit, if another
  dupack occurs indicating multiple losses in window, NewReno
  immeidately retransmits that packet also. This way, we can recover
  from multiple packet losses without going in and out of fast
  recovery. TCP along with the SACK option lets you learn well about
  multiple losses in a single window, and recover from them quickly.

* Note that TCP congestion control is inherently unfair to long RTT
  flows. Whenever a loss happens, it takes an RTT duration to find
  out. The ACK clock is also dictated by the RTT. Therefore, higher
  RTT flows see lower throughput.

* The AIMD form of congestion control can take a long time to discover
  the ideal cwnd, espcially in high BDP networks. However, if we are
  too aggressive in newer versions, we can be unfair to existing older
  TCPs. Moreover, TCP must be fair to RTT variations. As a result,
  there has been lot of work done in designing TCP variants that
  perform well in a variety of conditions and are "TCP friendly",
  i.e., work well with older TCPs.

* The default TCP in Linux today is CUBIC, that aims to discover ideal
  cwnd faster for high bandwidth delay networks. CUBIC is conservative
  around up to a certain window size, but beyond that, it explores
  aggressively to discover ideal cwnd in high BDP networks.

* Other variants like TCP Vegas aim to monitor signals other than
  packet losses to determine the value of cwnd. For example, Vegas
  detects congestion by realizing that the throughput is flattening
  out even when the cwnd is increased (as a result of queues building
  up at the bottleneck). Thus it reduces cwnd when it notices this
  signal, hopefully much earlier than when the buffer overflows and
  drops packets. However, Vegas is somewhat sensitive to RTT
  estimation, and is not widely deployed.

* Different TCP variants differ in the following:

- What are the signals? Most TCP variants use ACKs (new ack / dupack)
  and timeouts as signals. Some variants use other measurements like
  rate of sending.

- How is increase/decrease done? TCP Reno does AIMD. Different
  variants differ in this method.

* Is dropping packets the correct signal? Should we wait till packet
  drop to learn about congestion. Another idea: Random Early Detection
  (RED) gateways. When a router implements RED, it notices when the
  queue length is building up (meaning congestion is about to happen),
  and randomly drops packets so that TCP adjusts itself. Alternately,
  it can also "mark" packets, and TCP sender should look for this
  Explicit Congestion Notification (ECN) bit, and react to it like a
  packet loss. 

* RED should be careful to mark/drop only in response to persistent
  congestion, and not transient congestion. So should use average
  queue size. Should not be biased towards bursty traffic, and should
  not cause synchornization of all flows. So RED drops/marks packets
  randomly.

* How does RED work? RED uses two thresholds of queue sizes. If queue
  length below the lower mark, do nothing. If netween lower and higher
  threshold, then use a drop probability that increases linearly
  proportional to queue length. If queue length above higher
  threshold, drop always. Setting these various thresholds is a tricky
  problem.

* Next, details of TCP segments and connection setup.

* What does a TCP segment have? Source and destination ports (2+2 = 4
  bytes). Source and destination IPs are part of IP header. 32-bit
  sequence number and 32-bit ack number (4+4 = 8 bytes). Various flags
  like SYN, SYN ACK, ACK, FIN, along with header length (2 bytes). A
  16-bit receive window size for flow control (2 bytes). 16-bit
  checksum (2 bytes). Urgent data pointer for any urgent data (2
  bytes, rarely used). Total 20 bytes. Plus variable number of
  options.

* Note that 16-bit window size only allows window size up to 64 KB to
  be announced. 32-bit sequence number does not wrap around for a long
  period of time in normal links, but conflict of seqeuence number due
  to wrap around can happen in a few minutes on high speed links. TCP
  has extensions to deal with both these issues.

* A note on sequence numbers and window sizes. For windowed protocols,
  the sequence space should be at least 2X window size. To see why,
  suppose sender has sent a window of packets and receiver has acked
  them. But acks are yet to reach sender. Then sender is still waiting
  at window [1..N], while receiver is expecting packets in window
  [N+1..2N]. The sequence numbers in these two sets should be
  different; otherwise, receiver cannot tell a retransmission from a
  new transmission. So sequence numbers should not repeat for at least
  2X window size. In practice, if there is possibility of arbitrary
  reordering and delays, sequence numbers should be distinct for much
  longer than 2X window.

* TCP treats data as byte stream. So sequence number is number of
  first byte in the segment. So, a segment of sequence number "i" has
  data from i to i+MSS-1 bytes. Sequence number in ACKs is sequence
  number of next byte receiver is expecting. ACKs are comulative in
  TCP. If only one side is sending data, seq number in one direction
  and ACK in other direction are the ones to watch. If both sides
  sending data, ACKs are piggybacked on data segments.

* SYN and SYN ACK occupy one byte each in sequence space. So SYN
  packet has seq=0, ACK=<anything>, syn flag set. SYNACK has seq=0, ack=1 (ack
  for syn byte), syn flag set. SYN ACK ACK has seq=1 (one byte left
  for syn), ack=1 (one byte acking for syn ack), and syn flag not set.

* In fact, client and server pick random initial sequence numbers, for
  several reasons. Suppose client initial seq number is CISN and
  server's is SISN. Then, SYN has seq=CISN, ACK=<some invalid
  number>. SYN ACK has seq=SISN, ACK=CISN+1. SYN ACK ACK has
  seq=CISN+1, ack=SISN+1.

* After SYN and before SYN ACK ACK at server - it is called a half
  open connection. Server must maintain some state for half-open
  connections, so that when it gets a future SYN ACK ACK, it can
  correlate with earlier SYN. However, if server keeps, lots of state,
  a common attack exists. SYN flood attack - exhaust resources with
  half open connections. Client sends lots of SYN, server allocates
  resources, client never sends SYN ACK ACK. Client loses nothing with
  SYN, but server will allocate resources and crash eventually. To
  avoid this, server picks its ISN as a hash of a secret key and TCP
  4-tuple. When SYN ACK ACK arrives, server checks that it is from the
  same guy by checking hash again (no one else could have generated
  SISN+1 unless they are the original guy that sent SYN and received
  SYNACK). This special server init seq numner is called SYN cookie.

* Another reason for random ISN - to prevent overlap with earlier
  connection packets with same 4 tuple. Also security as we see above.

* Port scanning is a common attack used where attacker sends SYN
  messages to all ports on a machine. If a server is listening, you
  get SYN ACK reply. Else you get ICMP error message. The "nmap" tool
  does this (check it out).

* Once client or server finishes their work, they send FIN, expect FIN
  ACK from other size. Then the other side also winds up everything
  sends FIN, waits for FIN ACK. Then the connection has closed. If
  anything unexpected, reset (RST) is sent. After client sends FIN,
  gets FIN ACK, FIN from server, it sends ACK, and starts a final
  timer (TIMED_WAIT). In this time, it waits for any other packets
  from server (say another FIN because ACK was lost). Finally, after a
  small wait of 1-2 minutes, it packs up all state. FIN also occupies
  1 byte in sequence space like SYN. (why is timed wait needed?
  because there is no ACK for the last ack, and it may be lost.)

* TCP states of a client: Init -> (send syn) -> SYN SENT -> (receive SYN ACK, send ACK) -> ESTABLISHED -> 
  -> (send FIN) -> FIN_WAIT_1 -> (receive FIN ACK) -> FIN_WAIT_2 -> (receieve FIN, send ACK) -> TIMED_WAIT -> (wait 30 sec) -> CLOSED

* TCP states of server: Init -> (create socket, bind) -> LISTEN -> (receive SYN, send SYN ACK) -> SYN_RCVD -> (receive ACK) -> ESTABLISHED 
  -> (receive FIN, send ACK) -> CLOSE_WAIT -> (send FIN) -> LAST_ACK -> (receive ACK) -> CLOSED

* Lots of subtleties in the corner cases of connection establishment and teardown.

* Now we move on to timeout mechanism. TCP estimates RTT with every
  received ack (time between sending and ack). TCP does not use RTT
  samples from retransmissions. TCP maintains a EWMA of sampled RTT.

  estimated_rtt = (1-alpha) * estimated_rtt + alpha * sampled_rtt. 

* TCP also maintains the deviation expected from RTT

  dev_rtt = (1-beta) * dev_rtt + beta | sample_rtt - estimated_rtt|

* Finally TCP sets timeout to estimated_rtt + 4 * dev_rtt

* New data packets get timeouts as computed above. For packets
  retransmitted after timeouts, timer is set to twice the original
  timer.

* Some implementation-based tricks:

- When receive window is 0, sender must not send any data until
  receive window opens again. But how will receiver communicate that
  receive window is free if no data and ack exchange? So sender will
  keep sending 1-byte packets to check if receive window opens.

- Delayed ACKs. TCP tries to wait for some time before sending ACK, to
  send ACKs for two segments at a time. Only for in order
  segments. Out of order segment ACKs (dupacks) are sent immediately.

- Nagle's algorithm. If TCP has less than MSS worth of data to send,
  it can decide to wait for some time to wait for more data. Nagle's
  algorithm says that it is ok to wait if there is data in flight,
  which means that an ACK may arrive soon to clock the next
  transmission. This wait may add some delay to interactive
  applications like ssh. This wait can be disabled using a socket
  option.

* Where is TCP implemented in real systems? Usually, it is part of the
  kernel. When a user-space application opens a socket and pushes data
  into it, the kernel takes care of transmitting the TCP using UDP or
  TCP as requested by the application. The "sock" structure in the
  kernel also holds TCP-specific state like cwnd, retransmissions,
  sequence numbers, and a linked list of packet buffers with unacked /
  received data. The TCP and IP processing happens in kernel, before
  data is handed off to a device driver for link-layer processing. The
  receiver's kernel handles the receive side TCP processing, and
  places data in the buffers linked to the receiver "sock" structure.

* Multipath TCP: a way to make TCP use multiple network paths betwene
  two hosts. Why? To utilize multiple interfaces at a device (e.g.,
  WiFi and 3G on mobile phone, or multiple ISPs), to load balance data
  across many paths (e.g., in a data center). Normally, TCP uses only
  one path - it is bound to the source and destination IP interfaces,
  and cannot use other interfaces even if available. While the concept
  itself is simple and old, it is only recently that a practical
  implementation has been developed (that can work around various
  constraints like middleboxes that mess with traffic). The current
  MPTCP implementation does not require any changes to the application
  or network layers, and is backward compatible with TCP.

- MPTCP starts out as a regular connection. Say you establish a TCP
  connection on 3G first, specifying the MP_CAPABLE option. How can
  this connection use WiFi also? Cannot send packets with 3G source IP
  on WiFi link (IP spoofing packets will be dropped). Cannot simply
  send on WiFi without establishing a connection first (firewalls may
  drop). So, will need to do a handshake on WiFi path too, giving the
  MP_JOIN option, saying this connection is part of the previous
  connection. Some cryptography used to authenticate this subflow as
  belonging to the original flow. In addition, one TCP endpoint can
  inform the other endpoint that it has an additional address, and ask
  it to initiate a connection (in case of private IP addresses and
  NATs, where server cannot send SYN to client).

- What about sequence numbers? Each subflow will have its own sequence
  number space (otherwise, middleboxes will get upset). How do two the
  streams fit together - another special sequence number field is used
  to give the final ordering.

- Congestion control is implemented in each subflow separately. All
  subflows decrease by half when congestion is detected. Faster paths
  increase more than slower paths, to ensure more traffic goes on
  better paths. Also, the aggregate data sent will not be more than a
  regular TCP connection, to avoid being unfair to regular TCP.
