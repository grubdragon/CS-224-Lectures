Router architectures
=====================

* In this lecture, we will understand how IP networking is implemented
  in a typical router. Note that most operating systems implement IP in the
  kernel, so a Linux box can serve as a simple router. Why
  do we need specialized routers? Because a Linux machine cannot
  process packets at several Gbps.

* Main components of a commercial router:

- Input ports: perform L1/L2 (physical and link layer) functions. Then
  destination IP lookup (coordinate with forwarding engine), update
  headers, and send packet to output ports via switching fabric. Each
  input port may have a separate forwarding engine to achieve high
  speed lookups. Several input ports are placed in a single linecard
  hardware.

- Forwarding Engine: performs longest prefix match using destination
  IP and forwarding table, returns the output port / link / interface
  the packet has to leave. [In a simple linux router, this is handled
  by CPU.]

- Switching fabric: responsible for transferring packets from input
  ports to output ports.

- Output ports: handle L1/L2 processing during transmission. Also
  responsible for any scheduling policy to be followed.

- Routing processor: runs routing protocols and computes forwarding
  tables. It is like a general purpose CPU.

* Budget to process packets: consider a 10 Gbps link, with 64-byte
  packets. The rate in packets per second (pps) is 10 Gb / 64*8 =
  approximately 20 million pps. That is, we have 1/2million = 50
  nanoseconds per packet. In this time, we must lookup destination and
  transfer packet to output port. These two tasks form the bottleneck
  of any router design.

* IP forwarding algorithms: the algorithms responsible for performing
  a longest prefix match (LPM) of the destination address. Performed
  by the forwarding engine. Why is this a hard problem? With classful
  addressing, you had only 3 possible prefix lengths, so lookup was
  easy. Now, with CIDR, you can have any possible prefix length. So,
  we may have to lookup multiple routing table entries. With a 50 ns
  budget per packet, and typical memory access times being a few
  nanoseconds, we must lookup very few entries to meet this goal. 

* First, why is one address covered by multiple prefixes? Several
  reasons like prefix hole punching - the ISP has a larger (shoter)
  prefix and it gives a smaller (longer) prefix to customer, called
  hole punching. Then the customer might announce this smaller prefix
  through other ISPs. So we see the shorter prefix from ISP and longer
  prefix from customer in the routing tables. We must match on the
  longer one for correctness.

* What is the simplest data structure to implement a forwarding table
  and do LPM? A trie (pronounced as try). A trie is a tree-like
  structure. Every node has 2 branches for 0 and 1. The nodes at any
  level hold information about the prefix formed by walking down the
  tree up to that level. Branches that do not have any prefixes are
  pruned. To perform LPM, walk down the tree as far as possible. This
  method can take O(N) lookups, where N is length of IP address. And
  each level of the tree may be in a separate memory location. So too
  many memory lookups. Improvements to trie based lookup algorithms:
  compress portions of the tree that do not have many branches (path
  compressed tries).

* Another method is to search using prefix length. Organize prefixes
  into different sets based on length. Do a binary search on prefixes
  at various lengths, and find the longest one that matches. For
  example, for a 32 bit address, start with searching at prefixes of
  length 16. If a match is found, continue with prefixes in range
  16-32, and so on. Takes O(log N) steps. Another method is to
  represent prefixes as intervals, and find the shortest interval that
  holds a certain prefix. A lot of work has been done on efficient
  algorithms using all these approaches.

* Now we move on to switching fabric designs. The most common design
  is a "crossbar". A cross bar consists of N data transfer "buses"
  running from each of N inputs, and N buses from each of N
  outputs. The cross bar can be configured to allow any input port to
  send to any output port. Every input will have packets to some
  outputs, and a bipartite matching will be done from inputs to
  outputs to configure the crossbar for each round. In the matching,
  every input can be paired with only one output, and every output
  with only one input. A crossbar based switch will need efficient
  algorithms to perform matching. A cross bar can potentially transfer
  N packets from N input ports to N output ports, so in the best case
  can keep the inputs and outputs fully utilized. A crossbar is the
  most common fabric design used in high speed routers today.

* The speed up of the crossbar is defined as the speed of the crossbar
  I/O bus relative to the input port linespeed. For example, if the
  inputs are 1 Gbps links, and the crossbar can potentially transfer
  10 Gbps betwen each pair of matched input and output in each round
  of the crossbar schedule, then speedup is 10. In practice, you need
  a speedup > 1 to achieve good performance. Of course, the actual
  performance depends on which inputs and outputs are popular, how
  efficiently you can match etc. For example, if all inputs always
  send to the same output, then the crossbar cannot schedule many
  packets in parallel, and you will get a degraded performance even if
  the speedup is high.

* Now, recall we discussed buffers at routers, buffer drops,
  scheduling etc. Where do all of these happen in the router? At input
  ports or output ports? 

- Input ports need to have queues, since the switching fabric may not
  always be available to transport the packet. Suppose there are N
  inputs and N outputs. Suppose the switching fabric speed is same as
  input linespeed (speedup = 1). If all inputs have packets for only
  one output, then the fabric can send packets of only one input to
  the output. And all but one input that is matched will have to queue
  the packets. This is called Input Queueing (IQ). If the switching
  fabric speed is N times that of the input speed (speedup = N), then
  the fabric can transfer all N packets from all N ports in the time
  that each port receives one packet. So, in such cases, IQ won't be
  needed. In general, the switching fabric speed is not N times but
  greater than input linespeed, so a small amount of queueing may
  occur. A reasonable speedup (say 2X), a reasonable distribution of
  traffic from inputs to outputs (not all inputs going to same output
  always), and a good matching algorithm can avoid too much input
  queueing.

- What if the first packet in the input queue is to a busy output
  port, but the rest of the packets are to free output ports? If we
  have only one queue for all packets destined to all outputs, we may
  have head of line (HoL) blocking. That is, we cannot send the first
  packet (to the busy outout port), so we don't even get to the
  packets destined to less busy ports. To avoid this, inputs have
  different queues for packets destined for different output
  ports. That is, after packet arrives, forwarding table is looked up,
  and packet is placed in a queue at the input port, with packets
  destined to different outputs residing in different sub-queues. This
  is called Virtual Output Queueing (VOQ).

- After crossing the fabric, outputs also have queues to implement QoS
  related scheduling policies. It makes sense to implement
  these at the output queue because we want to distribute the outgoing
  link bandwidth between flows. 
