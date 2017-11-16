Link layer: Shared broadcast and switching
==========================================

* Two types of links / channels / transmission media: shared broadcast
  and point-to-point. A natural shared broadcast medium is
  wireless. For example, a WiFi channel. Ethernet is also a shared
  broadcast medium. That is, multiple transmitters connected to the
  channel can all hear each other. The other type of links are
  point-to-point. Only the two end points connected to the link can
  communicate with each other. Link layer design varies for these two
  types of links.

* Medium access: when multiple nodes share a shared broadcast medium,
  the link layer decides who transmits when using a medium access
  control (MAC) protocol. One technique is to partition the channel
  using techniques such as TDMA or FDMA. However, this is inefficient
  if all users do no always use their allocated partition. Another
  class of MAC protocols called random access protocol, which is more
  suited for bursty traffic patterns found in packet switched
  networks. The first random access protocol was ALOHA. In "slotted"
  ALOHA, time was divided into slots. At the beginning of the slot,
  the N nodes which have packets to send try to independently access
  the medium with a probability "p". The transmission succeeds if
  exactly one node accesses the medium. If two or more nodes transmit
  together, the transmissions fail due to "collisions". The
  utilization of the medium, defined as the probability that the
  channel does useful work, is defined as the probability that only
  one of the N nodes transmits. Therefore, U = N p (1-p)^{N-1}. This
  utilization is maximum when p = 1 / N, and the asymptotic value of
  the optimum utilization is 1/e = 0.37 for large N. That is, ALOHA
  works well when the value of the access probability p is set
  carefully, and the best it can work is still only 37% of the
  capacity of the link.

* Random access protocols perform better when they are based on
  "carrier sense" - that is, nodes sense the energy on the medium to
  determine whether someone else is sending or not, and transmit only
  if the medium is free. CSMA (carrier sense multiple access)
  protocols work this way. Are collisions completely eliminated with
  carrier sense? No, because there is a finite detection delay between
  a node starting transmission and other nodes identifying the medium
  to be busy. Therefore, if two nodes start transmissions within the
  detection delay, then collisions can still happen. What causes
  non-zero detection delay? Two factors: the propagation delay of the
  signal (at speed of light), and hardware processing delay to detect
  medium as busy. The longer this "blind spot", the greater the chance
  of collisions.

* Ethernet uses the CSMA/CD MAC protocol (CSMA with collision
  detection). Whenever a node has a packet to send, it senses the
  medium a small duration, and start transmitting if medium is
  idle. If medium is busy, it waits till it gets idle for a certain
  period of time. Once transmission starts, the transmitter monitors
  for collision. If a transmitter detects a collision has occurred
  after sending its preamble, (say, by finding that energy on channel
  is more than what can be caused by one transmission), then it sends
  a special jamming sequence and aborts the transmission. The node
  then waits for some time before retransmitting again. The wait time
  between successive retransmissions of the same frame increases
  exponentially ("exponential backoff") to avoid collisions.

* The maximum size of Ethernet is restricted to 2500m. Why? The longer
  the cable is, the longer it takes for faraway nodes to detect the
  channel is busy, and the higher the probability of collisions. The
  minimum size of Ethernet frame is fixed to 64 bytes (14 byte header
  + 46 byte payload). Why? Let "d" be the propagation/detection delay
  between two nodes A and B. When A starts sending a frame, B senses
  the medium busy only after time d. If B starts transmitting in this
  period, then the jamming signal takes a time d to reach
  A. Therefore, the minimum frame size should be such that it lasts
  for a period of 2d at least. The minimum frame size of 64 bytes is
  picked such that the frame lasts at least for a duration 2d, where d
  is the propagation delay along the maximum sized Ethernet cable.

* Other details of Ethernet. Ethernet frame consists of a payload (46
  to 1500 bytes), source and destination MAC addresses, a CRC for
  error detection, and a preamble at start of frame. Note that
  Ethernet does not do any error correction - no extra bits to recover
  from errors. No retransmissions by sender if checksum fails at
  receiver. Ethernet uses Manchester encoding: 0 is transmitted as a
  low to high transition, and 1 is encoded as a high to low
  transition.

* WiFi is another shared broadcast medium. It also uses CSMA. However,
  the main difference is that detecting collisions is hard in wireless
  channels (for reasons that are hard to explain in this
  course). Therefore, WiFi has link layer ACKs. When a sender
  transmits a frame, it waits for an ACK from receiver. If no ACK,
  assumes collision and retries. This is called CSMA/CA (CSMA with
  collision avoidance). Because wireless channel introduces a lot of
  errors, WiFi link layer also uses error correcting codes (not just
  error detection).

* As the number of transmitters increases in a shared broadcast
  channel, the probability of collisions increases. Therefore such
  link layers are not scalable for a large number of users.

* Ethernet hubs: when multiple ethernet cables terminate in a hub, the
  hub transfers frames from one port to all other ports to emulate a
  shared medium broadcast. Ethernet hubs have scalability issues of a
  shared broadcast medium. However, most LANs today are made of
  switched Ethernet.

