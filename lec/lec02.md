Lecture 2: Packet switching
===========================

* The Internet today is what is called a packet switched
  network. Information is broken into small packets. Each packet has a
  destination address. Each router / packet switch along the path
  looks up the destination address, and sends the packet along in the
  right direction. Is this the only way to build networks? No. There
  is also circuit switching.

* Circuit-swtching (CS) vs packet switching (PS). These are two
   fundamantal ways of transporting data through a network. In circuit
   switching, before starting a flow, a setup phase happens. A setup
   message goes through the network, resources at all links are
   reserved for the flow. That is, a dedicated circuit is established
   from one end point to the other, through which you can push
   data. The links along the path are shared by time/frequency
   division multiplexing (TDM/FDM) between various circuits. With TDM
   for example, you have a timeslot reserved for every established
   flow on all links. Telephone networks work this way. Why? Lower
   delay and jitter (variance of delay). But CS may potentially waste
   resources (e.g., when voice call is idle). 

* Packet switching is different from CS. There is no explicit setup
   phase for a flow. Whenever a source has a packet for a destination,
   the packet is sent along the first link. The node at the end of
   that link receives the packet and forwards it along as soon as it
   can. And so on. No explicit resources are reserved at any
   intermediate node for any flow. So if a link is busy, incoming
   packets may have to be temporarily stored in a PS network. Thus the
   intermediate nodes are called "store-and-forward" switches.

* The links in PS are shared not by explicitly reserving resources,
   but by "statistical multiplexing". That is, all packets that arrive
   at a node are sent out on the outgoing link in a
   first-come-first-serve or some other order. Intermediate nodes have
   buffers to store packets till they are forwarded. Packets may incur
   a "queueing delay" if there are many packets arriving at a node and
   waiting to be forwarded. Some packets may also be dropped if the
   buffer at an intermediate node overflows. Thus you cannot provide
   any guaranteed Quality of service (QoS), it is only "best effort".
   Packet switched networks incur longer delays and jitter, but lead
   to better utilization of resources. 

* The concept of packet swtching started in the 1960s. Until then, all
   networks (telephone) were all circuit switched. PS was seen as a
   better option for data transfer (not voice) due to the burstiness
   and long idle periods in data transfer (as opposed to voice). The
   Internet is packet switched. Recently, cellular data networks (4G
   etc) are also moving towards packet switching, but with better
   scheduling algorithms to provide QoS.

* Problem: statistical multiplexing vs TDMA.

  Consider a link of rate 1 Mbps. Users generating data at 100 kbps
  when busy, but are busy with probability 0.1. With TDMA, we can
  support only 10 users, and 90% of slots are wasted. If M users in
  total, probability that N users are active = choose(M,N) * p^N *
  (1-p)^{M-N}. So, for 35 users, prob that 11 or more active is
  0.0004. So most of the time it is okay as 10 or fewer users are
  active. When more than 10 users active, queueing takes place, and
  queue drains when fewer than 10 users are active. Statistical mux
  can support 3 times as many users. Also, leads to better utilization
  of the link.

* Note the notion of a packet: the unit at which you transmit
  information. If you have a big application message, you split it
  into smaller packets, put a header on each one, and send each one
  independently through the packet switched network. Headers are not
  so important for circuit switching because a circuit and timeslot
  schedule is established. Headers are a must for packet
  switching. How big should a packet be? Small packets - relatively
  greater overhead due to repeating header information in every
  packet, so larger packets are better this way. But with large
  packets, there is a long delay for the packet to traverse every
  hop. With smaller packets, the first packet can do down the second
  link while the first packet arrives. Also, even if one bit is
  corrupted, the full packet must be retransmitted (at link layer or
  transport layer). So choosing optimum packet size is a fine
  balancing act.

* History of networking: The only networks were telephone networks for
  a long time. Packet switching concepts and theory developed in the
  1960s, mainly to transfer data. Initially, it wasn't clear if the
  queueing delays and uncertainty of PS (as compared to CS) would be
  manageable or not, but several important theoretical results proved
  that it would all work fine.

* Several proprietary packet switched networks started in the 1970s
  beginning with ARPANET. The need came to up to integrate these
  networks using a few top-level store-and-forward gateways. The
  primitive TCP/IP protocols came into place, but TCP and IP were
  tightly coupled were integrated. What lead to the split into layers
  we see now?

  - One of the main goals of the internet - survivability, fault
   tolerance. The intermediate gateways have a lot of state. How
   should this state be preserved across failures? It was decided that
   the core would store soft state (periodically refreshed, nothing
   permanent, okay to lose state once in a while), and would only
   forward datagrams, in a connection-less manner. The end hosts would
   maintain any connection state needed. 

  - People also realized that TCP provides only one type of transport,
    and people may want to use different transport protocols for
    different types of applications. For example, not all applications
    will want the reliability provided by TCP. So, TCP and IP were
    split. UDP was provided as another alternative simpler transport.

  - Finally, we have IP that runs on any network that can deliver a
    datagram from one IP address to another. No assumptions made on
    the underlying network (in order, reliable etc). TCP and other
    transport run on top of IP and customize the transport protocol to
    the application needs.

  - The split of TCP and IP is also referred to as the end-to-end
    argument. All application and transport "intelligence"
    (reliability, security etc) should reside at the end hosts. The
    core of the network should be kept "dumb", with only the job of
    delivering datagrams as well as it can. It may be possible to
    spend a lot of energy in engineering the network core to do some
    of the jobs of the higher layers (e.g., provide reliability
    guarantees), but it was decided that the effort to realize it in
    the core was too high to be worth it, and the end points would be
    doing it anyways. End-to-end checks, retries, recovery at
    end-to-end level etc makes more sense. The tradeoff of where to
    place a certain functionality - end hosts or network - is a common
    debate in several aspects of networking.

* Continuing the story, the 1980s saw a refining of the TCP congestion
  control algorithms. The internet grows. US government lets go of the
  backbone routers and decides to commercialize it. ISPs come up in
  1990s. Late 1990s, early 2000s - dotcom bubble, explosion of usage
  and applications. Late 2000s - stabilization, growth of
  cellular/mobile data and convergence with the Internet.

* Demo: show wireshark trace of a browsing session
