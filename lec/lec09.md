TCP basics
=============

* Now, we will understand the congestion control and reliability
  mechanisms in TCP. Recall, TCP connection setup is the 3-way
  handshake, after which client and server each have a socket. TCP
  provides reliable in-order bi-directional byte stream abstraction
  between these two sockets.

* When sockets are created, a send and receive buffer is
  allocated. When applications write data into socket, it is placed in
  the send buffer till TCP transmits it. When data is received by TCP,
  it is placed in the receive buffer till the application reads it.

* What does TCP do with the data? First, it takes data from the send
  buffer and makes segments. What size segments? If enough data
  available, TCP prefers to make segments of size MSS - maximum
  segment size. How is MSS calculated? TCP MSS (maximum segment size)
  = MTU (maximum transmission unit allowed by all links on the path) -
  IP and TCP headers. Usually MTU is 1500 bytes, so MSS is 1500 - 20 -
  20 = 1460. 

* If TCP segment is larger than MTU on any path, then IP-level
  fragmentation (we will study this when we see the IP
  layer). Usually, hosts try to avoid this. So they discover the
  smallest MTU along the path and use this for MSS calculation, so
  that one TCP segment is sent as one IP datagram.

* What after segmentation? Sender adds TCP header. Among other things
  (like port numbers), the TCP header has sequence numbers. Why
  sequence numbers? Since the channel can reorder and delay packets,
  sender needs to put a sequence number on packets to tell the
  receiver which packet is what, so that receiver can suitably
  reassemble packets. Also, receiver can use the sequence number to
  tell sender which packets it got (ACKs) for reliability.

* Sequence number can be per-packet or per-byte. In TCP, sequence
  number is based in bytes. Sequence number of packet is the number of
  the first byte in the packet. Together with length in the TCP
  header, we know which packet has which bytes.

* The fundamental mechanism for reliability is acknowledgements. ACKs
  can be positive ACKs ("I got packet X or bytes Y--Z") or negative
  ACKs ("I didn't get ..."). ACKs can also be specific to a particular
  packet ("I got this one") or cumulative ("I got everything up to
  here"). TCP ACKs are sequence number based and cumulative. TCP ACK
  indicates the next sequence number expected, saying "I got
  everything till X".

* What if ACK is lost or corrupted? Then sender will unnecessarily
  resend packet? So receivers must be capable of handling duplicate
  segments (seqeuence numbers will help identify duplicates).

* So what if a packet is lost? TCP maintains a timer for every segment
  it sends. If no ACK within timeout, it must retransmit that
  segment.(Automatic repeat request or ARQ mechanism) Timeout is
  estimated by seeing the RTT values in the past. Sequence numbers,
  ACKs, timeouts, and retransmissions form the base of any protocol
  that provides reliability.

* How many segments to send? One option is to send a segment and wait
  for an ACK before moving on. So a simple reliability-based protocol
  would send one packet, wait for ACK, retransmit as many times as
  needed, before moving on to next packet. Such "stop-and-wait"
  protocols waste time, especially if RTTs are long. For example,
  suppose a data packet takes time "d" to send, and the RTT for
  getting an ACK is "T". Then, only d/(d+T) fraction of time was used
  in sending, and the sender was idling the rest of the time. What is
  the solution? Keep sending packets before waiting for ACKs. Deal
  with retransmissions later on. This technique is called
  "pipelining". Note that pipelining is not strictly necessary for a
  reliable protocol, but is highly desirable for performance reasons.

* With pipelining, sender can have not just one, but a certain maximum
  number of packets "outstanding" or "unacknowledged". When packets at
  the beginning of this "window" are acked, the window moves forward
  over the sequence number space. Hence the name "sliding window"
  protocols. The maximum number of packets allowed in the window is
  called the window size. TCP is a sliding window protocol.

* In a window of packets, what happens if a packet is lost and
  subsequent packets are received?  The TCP ACK is the next sequence
  number expected. So, if there is a gap in sequence space or a "hole"
  due to some packets lost (or reordered), then the ACK number will
  still indicate the first sequence number expected to fill the
  hole. Every subsequent packet received after the hole will cause the
  receiver to send an ACK for the same sequence number. These are
  called duplicate ACKs. These indicate lost segments. Sender uses 3
  dupacks to retransmit a segment. This is called fast retransmit, as
  opposed to timeout-triggered retransmits. 3 dupacks is a form of
  NACK.

* It seems wasteful to only send the same ACK sequence number (dup
  acks) and not communicate information about what was actually
  received beyond the hole. TCP also has a more advanced mechanism
  called "selective ACKs" or SACKs that can indicate some additional
  data received beyond a hole. Widely deployed today. 

* What to do with out-of-order packets? You can throw them away, or
  buffer them. TCP standard doesn't specify what to do, but most
  receiver implementations buffer out-of-order packets.

* What to retransmit? Suppose window size is N. Suppose current window
  is from i to i+N-1. Now suppose we know that packet "i" is
  lost. What should the sender retransmit? Two possible solutions: GBN
  or SR. GBN or Go-back-N (the sender resends the entire window of packets
  starting with "i"), or SR / Selective Repeat (sender retransmits only "i"
  and hopes other packets will reach).

* Clearly, GBN results in many unnecessary transmissions. Why would
  anyone use it? Receiver simplicity. A receiver does not have to
  buffer out-of-order packets. A receiver, when it gets the next
  packet in order, sends an ACK for it. Suppose packet "i" is lost,
  and receiver gets packet "i+1", then it can throw it away, because
  sender will retransmit the entire window starting from sequence "i"
  anyways. Also note that ACKs in GBN are cumulative for this
  reason. Because a receiver does not process out of order packets, an
  ack for seq "i" means all packets less than "i" have been received.

* SR is a more sensible choice. When receiver gets a packet out of
  order, it buffers it, and sends an ACK saying that it got a certain
  packet. That way, when a timeout of a packet in a window happens,
  sender can only send the unacked packets.

* Is TCP like GBN or SR? TCP does something in between GBN and SR,
  closer to SR. TCP sends only one packet that it thinks is lost, not
  entire window, so not like GBN. TCP receiver buffers out of order
  segments. But ACK indicates the sequence number that is missing
  (unlike SR). TCP selective ACK (SACK) option exists to ACK a few out
  of order segments also. But the main ACK sequence number in TCP
  header is used to inidcate the first packet that is expected
  next. TCP with SACK is a lot like SR. 

* Suppose window is packet [i, i+n-1]. Suppose packet "i" has been
  lost, but receiver has received the other N-1 packets in the
  window. Note that the receiver is buffering all these N-1 out of
  order packets now. Even though the sender has gotten N-1 acks, he
  cannot advance the window to send the next N-1 packets, because that
  would increase the out-of-order buffer size at receiver. The sender
  must maintain that the highest seq number sent is only N packets
  away from the start sequence of the window. So, window size is not
  the number of unacked / outstanding packets, but rather the
  differennce between highest seq number sent and highest seq number
  acked. This is to ensure that the receiver never has to buffer more
  than N out-of-order packets.

* So far, we have seen ACKs and retransmissions for reliability. Next
  big question is, what should be size of this sliding window?
  Ideally, it should be equal to the bandwidth delay product. If you
  send BDP worth of packets, by the time you finish sending your last
  packet, you will get ACK for first packet, and you can send the next
  packet. This process of ACKs triggering new packets is also called
  "ack-clocking", as acks arrive at the rate at which network is able
  to send data. So if you send data whenever ack arrives, you are
  automatically doing the right thing.

* In reality BDP is hard to know. What happens if you send more than
  BDP? Packets may be queued up behind congested links and take longer
  to reach (higher RTT). Worse yet, some router buffer may overflow
  and drop packets. So we must use packet losses or RTT increases as
  signal to adjust window size and try to learn BDP. 

* TCP performs congestion control as follows. So it maintains a cwnd
  (congestion window) of the maximum number of bytes it can send from
  first unacked sequence number. The value of cwnd is determined by
  congestion control algorithms. As long as there is space in cwnd,
  sender keeps sending TCP segments of size MSS. ACKs will help in
  clearing space in cwnd, and also increase and decrease cwnd.

* TCP congestion control has two parts: slow start and congestion
  avoidance. And optional fast recovery in newer versions (by default
  now). 

* Initially, TCP starts with cwnd of 1 MSS. On every ack, it increases
  cwnd by 1 MSS. That is, cwnd doubles evert RTT. Initially sends 1
  segment. On ack, sends 2 segments. After these 2 acks come back,
  sends 4 segments etc. TCP rate increases exponentially during slow
  start. Slow start continues till cwnd reaches "ssthresh" threshold.

* After sshthresh is reached, cwnd increases more slowly, by one 1 MSS
  evert RTT.

* Now, if we get 3 dupacks, we do fast retransmit of the lost
  segment. Along with it, we also do fast recovery. Fast recovery can
  be reached from slow start or congestion avoidance. Again, we set
  sshthresh = cwnd/2. However, we do not set cwnd all the way to 1
  MSS. We set it to half the value where congestion occured. Thus this
  congestion avoidance is also called additive increase multiplicative
  decrease (AIMD). 

* Finally, fast recovery ends after the loss has been recovered and we
  get a new ACK (not dupack). Typically, once the hole has been
  plugged, a large number of segments will be covered by the new ack.
  We start with the halved value of cwnd (that is stored in
  sshthresh), and start AIMD congestion avoidance again.

* Finally, note that even if the network allows, sender must sometimes
  slow down when receive buffer is full. This is called flow
  control. Receiver indicates the space available in its receive
  buffer with every ack. Sender always takes min(cwnd, receive window)
  when deciding how many bytes should be outstanding.

* Note: do not confuse congestion control (adjusting cwnd in response
  to packet loss and congestion in network) and flow control
  (adjusting sending rate so that we do not overwhelm the receive
  buffer).
