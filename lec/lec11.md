TCP analysis
==============

* Ideally, the window size or number of packets in flight of the
  transport protocol should be set to the BDP of the path, where
  bandwidth = rate of the bottleneck link, and RTT is end-to-end RTT
  from sending data to receiving ACK. Understand why. Suppose BDP = N
  packets. Then, after you have sent N packets, the ACK for the first
  packet would have come back, enabling you to send the next packet
  and so on. So in the ideal case, a window size of exactly N will
  give you just enough pipelining to fully utilize the
  bottleneck. Acks will come at precisely the correct time when you
  need to send the new packet ("ack clocking"). With less than N
  packets in flight, bottleneck may be idle some times, leading to
  lower throughput. If you have more than N packets in flight, extra
  packets will get queued up at the bottleneck router (and eventually
  dropped when buffer is full). You won't increase throughput, but
  will increase delay per packet. So, in ideal world, maintain BDP
  worth of window size, and no buffer at bottleneck router.

* Understand the graph of throughput vs. number of packets in flight
  (increases up to BDP, and flattens out). Understand graph of
  observed delay of packets vs number of packets in flight (flat
  initially as no packet sees queueing, increases linearly after
  number of pakcets crosses BDP due to queueing delays). [See the
  bufferbloat reference.]

* In practice, BDP is hard to measure, so TCP uses a self-learning
  mechanism. It increases window size when all is going well, and
  reduces when it observes congestion (as indicated by packet loss for
  example). Ensures that it hovers around the ideal window size. So
  can undershoot and overshoot the ideal window. A buffer in the
  bottleneck link is needed to accommodate these fluctuations.

* BDP is also variable due to cross traffic (other flows in the
  network). So, bottleneck buffer is needed to absorb these
  fluctuations from other traffic and ensure bottelneck is utilized
  and packets are not dropped when available bandwidth changes
  suddenly. 

* The bottleneck buffer size is a very important parameter in TCP
  performance. What is the ideal bottleneck buffer size? If no cross
  traffic and ideal world, its ok to not have any buffers (as we
  explain above, buffer only hurts delay with no throughput benefit in
  ideal case). However, given that BDP is not known and is variable,
  and TCP uses some heuristics to discover BDP and ideal window size,
  we need to have some buffering in the real world. How to set buffer
  size in the real world?

* Even before we set buffer size to ideal value: what happens if
  buffer size is below ideal value? Overbuffering, too much delay, no
  gain in throughput. What happens if we undersize the buffer? Not
  enough packets to send and bottleneck link may be underutilized. So
  the buffer size should be set such that it is just large enough to
  keep link occupied, but not too much beyond. Now let's see what the
  ideal value of the buffer is.

* One simple heuristic: set buffer size equal to BDP, so that even if
  BDP worth of packets come in a burst, you can handle it. A more
  analytical argument for BDP-sized buffers is below.

* Consider a very simple TCP model. TCP sender is in congestion
  avoidance steady state. Increases window up to W packets, packet
  loss happens, reduces window to W/2. Let's think in terms of packets
  for ease of analysis.

* Now, packet loss happens when window reaches W packets. At this
  point, the bottleneck buffer and the pipe have both filled up and so
  bottleneck router has dropped a packet. Let the size of bottleneck
  buffer be B packets. Let the rate of bottleneck be R and end to end
  delay on the path be D. Now BDP = R * D. And W = R * D + B.

* Now, the sender's window reduces to W/2. Under the sender gets at
  least W/2 acks, it cannot send next packet. In this time that the
  sender slows down, the bottleneck buffer should have enough packets
  to sustain the link. Since sender receives acks at the bottleneck
  rate (ack clocking), the time taken for sender to receive W/2 acks =
  time taken for bottleneck link to send W/2 pakcets. That is, the
  bottleneck buffer should be able to send at least W/2 packets at
  bottleneck rate before it starts getting new packets. Therefore, B
  should be at least W/2. From these two points, we conclude W/2 = R*D
  or W = 2 R*D and hence B = R*D.

* If buffer is set at this value of R*D, it will empty exactly when
  the sender has gotten W/2 acks and starts sending new data. So
  buffer occupancy goes from R*D to 0, and starts filling up when the
  sender starts ramping up again. 

* When buffer is set to BDP, the max window size W = 2 BDP, and min
  window size W/2 = BDP. So window keeps oscillating from BDP to (BDP +
  buffer size = 2 BDP). Average window size is 1.5 * BDP.

* When queue is empty (buffer has fully emptied, just before TCP
  starts ramping up after loss), queueing delay = 0, so total delay =
  D. When queue is full (TCP is at max window size, buffer is full,
  just before a buffer drop), queueing delay = D, so total delay =
  2D. That is, the TCP segment RTT goes from D to 2D due to queueing
  delay, so average RTT = 1.5 * D.

* What happens if buffer is too large or small? If buffer is too
  large, it will never empty between two TCP cycles, it will lead to
  extra queueing delay. Note that TCP can fill up any buffer space
  available by choosing a larger-than-required window size. This is
  called buffer bloat problem. Exists in the Internet today to some
  extent, especially in wireless networks, home cable networks
  etc. [See the bufferbloat reference paper.] For example, if buffer
  size is 3 * BDP, TCP window size will oscillate between 4* BDP and
  2* BDP, so after 2* BDP out of the 3* BDP buffer clears out, TCP
  will start ramping up again. So the buffer will never empty, leading
  to unnecessary extra queueing delay.

* If buffer size is too small, it leads to buffer underflow. That is,
  the buffer cannot keep the bottleneck link busy when TCP slows down
  (and TCP will be slowing down too much due to frequent buffer
  drops). Frequent buffer drops (and other sources of loss) are
  potential factors that can cause low TCP throughput due to
  fewer-than-ideal number of packets in the window. Suppose buffersize
  = 0.5* BDP. Then window size will oscillate between 1.5 * BDP and
  0.75 * BDP. TCP will start ramping up only after getting 0.75 * BDP
  worth of ACKs, but during this time, the buffer can only push
  through 0.5 * BDP worth of data. Rest of the time, TCP sender is
  waiting for ACKs, but buffer is empty and link is not utilized.

* What happens when there is more than one flow? Does BDP heuristic
  for buffer size hold? When multiple flows share a link of rate R,
  their throughputs will add up to R, so their BDPs will add up to R*D
  (asusming equal delays D). So conventional wisdom was that links
  should be provisioned for bandwidth * average delay
  expected. However, analysis ["Sizing Router Buffers" reference]
  shows that it can be lower values are enough because the peaks of
  all flows won't be synchronized.

* Average window size = 3/4 * W. So average throughput = 3/4 * W * MSS
  / RTT. Clearly, TCP throughput achieved has an inverse relationship
  with RTT. This is called RTT unfairness of TCP. If two links share
  bottleneck link. Then the flows with higher RTT will get a lower
  share of the bottleneck link.

* What is the relationship between TCP throughput and loss rate? [See
  reference on TCP throughput model.]  Consider the sawtooth
  diagram. Consider one cycle where window goes from W/2 to W. Since
  window size increases by 1 segment every RTT, it increases by W/2
  segments in W/2 * RTT. The number of pakcets transmitted in one
  cycle area under one "tooth" = (W/2)^2 + 1/2 * (W/2)^2. The expected
  number fo packets in one cycle is also 1/p if p is the probability
  of packet loss. Equating the two, we get W =
  sqrt(8/3p). Substituting in the throughput formula, we get

  throughput = sqrt(3/2) * MSS / ( RTT * sqrt(p) )

* In general, more losses means TCP is frequently reducing window
  size, so it may lead to lower than optimal utlization of bottleneck
  link and lower than ideal throughput.

* Finally, discuss fairness. Consider two users sharing a link. Let
  (x,y) be their throughputs. We can represent their achieved rates on
  a graph at point (x,y). Now, for both flows to utilize the link
  capacity C fully, we want x+y = C. For fairness, we want x=y. We
  want the congestion control algorithm to converge to the
  intersection of the two lines. We can show via graphical reasoning
  that Additive Increase Multiplicative Decrease (AIMD) converges to
  the ideal point, no matter what value of x and y we start with.

