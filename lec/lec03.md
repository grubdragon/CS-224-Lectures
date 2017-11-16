Lecture 3: Modeling the performance of computer networks
=========================================================

* Two main attributes that measure the performance of a network:
  throughput (how many bits per second are going through the network)
  and delay (how long does it take a bit from one end to the
  other). Note that these are two orthogonal concepts: think of it as
  width of a pipe and length of a pipe through with data flows.

* Throughput is related to other quantities like bandwidth and
  datarate of a link. A link can have a certain "nominal" bandwidth or
  datarate to send data at, however, all of it may not be used all the
  time to send useful bits. You may also have packet losses and
  retransmissions. Throughput measures the number of useful bits
  delivered at the receiver, and is different from but related to the
  individual link data rates.

* The throughput of a transfer is limited by the link with the slowest
  throughput along the path - bottleneck link. You cannot pump data
  faster than the rate of the slowest link. Note that the bottleneck
  link need not always be the link with the slowest nominal
  datarate. Sometimes a high speed link may be shared by several
  flows, causing each flow to receive a small share, thus becoming the
  bottleneck.

* Sometimes, you may not always be able to send at the bottleneck
  rate, because your protocol may have other delays, like waiting for
  ACKs. So, while instantaneous throughput can be the bottleneck link
  rate, average throughput may be lower. The way to compute average
  throughput is always: see the data sent over a period of time, and
  get the ratio. A file of size F takes T units of time to be
  transferred. Average throughput is F/T. 

* Problem: Concept of average throughput. Note the difference between
  bottleneck bandwidth/instantaneous throughput and average throughput
  in this problem.

  Consider a 125 KB file that needs to be sent through a network
  path. The bottleneck bandwidth of the path is 1 Mbps. The one way
  delay between sender and receiver is 20 ms. Suppose the sender
  continuously sends data at the bottleneck rate, and no packets are
  lost, no retransmissions. How long does it take to send the file?
  Ans: 125*8*1000 bits/ 1000*1000 bps = 1 second. Average throughput
  is 1 Mbps, which is the bottleneck bandwidth.

  Suppose the sender needs to wait for an ACK after sending
  every 1 KB packet. Assume ACK also takes 20 ms to come back. Now,
  the sender can send 1 KB in 20+20 = 40 ms. So the average throughput
  is 1*8*1000 bits / 40 ms = 200 kbps. So the average throughput is
  one-fifth of what it was before, with the new ACK requirement. So
  the time taken to send the file will be 5 times larger = 5
  seconds. You can also compute 5 seconds as follows: 1 KB takes 40
  ms, so 125 KB takes 125 * 40 ms = 5 sec.

* Delay: delay of an end-to-end path is sum of delays on all links and
  intermediate nodes. Several components of a delay:

  (1) When a packet leaves a node, it first experiences transmission
  delay. That is, all the bits of a packet have to be put out on the
  link. If a link can transmit data at R bits/s, a packet of size B
  bits will require B/R seconds to be just put out there.

  (2) Next is propagation delay. That is, the bits have to propagate
  at the speed of waves in the transmission medium to reach the other
  end. This delay depends on the length of the wire, and is usually
  only significant for long distance links. If d is the distance the
  wave has to travel is s is the speed in the medium, the propagation
  delay is d/s. Speed of light is 3*10^8 m/s in free space. So a radio
  wave takes 1 microsec for a distance of 300 metres. Speed of light
  in copper is around 2*10^8 metres. So it takes 10 nanosec to travel
  a 2 meter long wire.

  (If prop delay < trans delay, then the first bit of the packet would
  have reached the other end point before the sender finishes putting
  all bits on the wire. So the limiting factor is really how fast the
  link is. On the other hand, if prop delay > trans delay, as is the
  case for long distance links, then the first bit reaches the other
  end point much after the last bit has been sent.)

  (3) Next, once it arrives at the other end point, it must be
  processed by the switch or router. This processign delay could
  involve looking up routing tables, some computations of header
  checksums etc. Again, this is usually not a significant component
  with today's high-speed hardware.

  (4) Once the other end point processes the packet and decides which
  link to send it on, the packet may potentially be queued until the
  next link becomes free. This delay is called the queueing
  delay. This is the most unpredictable part of the delay, as it
  depends on traffic sent by other nodes. Note that queueing can
  happen at the input port or output port, depending on design of the
  switch/router.

  A large branch of study ("Queueing theory") is devoted to modeling
  and understanding this delay under various conditions. Internet
  traffic is often bursty, and hence queueing delays occur even if the
  aggregate traffic is less than the capacity of the links on an
  average. That is, suppose incoming packets arrive at an aggregate
  rate of L bits/s and link rate is R bits/s, then as long as L<R, it
  appears that there should be no queueing. However, packets don't
  arrive in an equally spaced fashion, and the arrival pattern is
  often random. In such cases, the queueing delay maybe high even if
  L/R < 1. In fact, queueing delay increases quite steeply as L/R
  approaches 1 (approx 1 / (R-L)). Usually, network designers try to
  keep this ratio well below 1.

  Once the packet gets out of the queue and gets ready for
  transmission, the cycle begins again with the transmission delay on
  the next link. So we add one of each of the 4 delays for every link
  traversed. Some switches can also start transmission even before
  reception fully completes. But most often, switches today are
  store-and-forward. That is, they wait for entire packet to arrive,
  then start forwarding.

* Once a queue is full, it may also drop packets, leading to
  losses. Losses can also occur due to transmission errors on the wire
  (more common in wireless links; wired links are pretty reliable).

* Problem: concept of end-to-end delays. 

  Consider a link A--S--B. A and B are end hosts and S is a
  switch/router in between. Each link has length 200 m (speed of light
  in medium is 2*10^8 m/s). Each link has bandwidth of 1 Mbps. A has
  sent a 125 byte packet. Each node takes 9 microsec to process each
  received bit. Let's compute various end-to-end delays.

  Propagation delay on each link is 1 microsec. Once a bit is sent, it
  takes 1 microsec to reach end of link + 9 microsec to be processed
  by endpoint (switch/endhost). Suppose A starts sending the 125 byte
  = 1000 bit packet. It takes 1000 microsec = 1 millisec to put the
  packet on the link fully. Now, this last bit takes an additional 1+9
  = 10 microsec to propagate and be processed. By this time, the other
  bits have reached as well. So it takes 1010 microsec from node A
  starting transmission to switch S receiving the packet.

  In reality switch S will also incur some extra delay to queue packet
  (due to other flows), but lets ignore that. Now after receiving the
  packet on link A-S, switch S will start sending packet on link S-B,
  which takes another 1010 microsec. So, overall end to end delay from
  starting transmission at A to completing reception at B is 1010*2 =
  2020 microsec.

* Problem: benefits of packetization in store-and-forward networks.

  Consider the above A--S--B problem. Suppose A wants to send a 1MB
  file to B. A has two options: send the entire file as one big packet
  on both links. Or, it can break up the packet into 100-byte chunks,
  add a 25 byte header per chunk, and send it on the links. Let's
  compute end-to-end delays in both cases. For now, let's ignore
  propagation and processing delays, as they are small.

  Sending 1 MB packet on 1 Mbps link takes 8 sec. So 8 sec on A-S and
  8 sec on B-S = 16 sec total.

  Suppose A divides file into 10,000 100-byte chunks. Each chunk with
  headers comes up to 125 bytes. A sends first 125-byte packet in 1
  millisecond. While S forwards this packet to B, A can send the next
  packet to S. (Switches can normally send packets while receiving
  them on another port.). So A takes 10,000 * 1 millisec = 10 seconds
  to send all the packets to S. The last packet takes an additional 1
  millisecond to get from S to B (the other chunks would have already
  been sent). So, it takes 10.001 seconds to transfer the file, even
  though we are sending lot of extra bytes due to headers in the case
  of packetization.

* Bandwidth-delay product (BDP). If bandwidth is width of a bit on the
  link, and delay is length of the bit, then BDP is the volume of the
  pipe in terms of how many bits it can hold. BDP signifies how many
  bits the sender can send before the first bit reaches receiver. For
  example, consider a 1 Mbps link or propagation delay 1
  millisec. Then the BDP = 1 MBps X 1 millisec = 1000 bits. That is,
  when the sender has sent 1000 bits, the receiver has just started to
  receive the first bit. Suppose an extra signal/packet has to be sent
  by receiver that it has started to receive the bits. Let us consider
  this sum of forward delay and reverse delay together as the
  round-trip-time (RTT). Then the product of bandwidth and RTT is the
  number of bits "in-flight" before the sender can expect to hear back
  from receiver. BDP can refer to product of bandwidth and one-way
  delay or RTT, depending on context.

* Problem: BDP calculation

  A sender and receiver are communicating over a 1 Mbps link. Sender
  sends 1 KB packets. Receiver sends 250 byte ACKs. Suppose round trip
  propagation delay is 10 millisec (assume long distance link).

  Now, transmission time of data packet is 8 millisec. Transmission
  time of ACK is 2 millisec. Total propagation delay (forward and
  reverse) is 10 millisec. Thus the amount of time from sender sending
  first bit of packet to receiving last bit of ACK is 20 millisec. If
  the sender transmits only one packet and waits for ACK, his average
  throughput will only be 1 KB / 20 millisec = 400 kbps.

  The bandwidth delay product (using RTT) of the link is 1 Mbps * 20
  millisec = 20,000 bits = 2.5 packets of 1 KB each. That is, if the
  sender were to be continuously sending without waiting for an ACK,
  he would have sent 2.5 packets before receiving the ACK for the
  first packet from the receiver. The concept of BDP is essential to
  understanding how much data to keep in flight to fully utilize a
  link. Also, signifies how much data is in flight before you hear
  about packet losses etc from receiver. We will revisit this concept
  of BDP when studying TCP. TCP tries to keep BDP worth of data in
  flight to fully utilize the bottleneck link.


