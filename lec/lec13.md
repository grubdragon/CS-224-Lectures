Network Layer - Details
===============================

* Recap: in the previous lecture we learnt about IP addresses (format
- dotted decimal notation, allocation - static vs dynamic, public vs
private), prefixes (subnet mask / prefix length), routing or network
control plane (routing protocols that figure out network layer paths
between hosts), and forwarding or network data plane (match
destination IP address using longest prefix match in the forwarding
table, and forward packet along to the correct next hop).

* What is the difference between a public IP and a private IP? A
  public IP is routable. That is, an organization announces the public
  IPs it owns to everyone else, so the public prefixes are present in
  the routing and forwarding tables of routers all over the
  Internet. Therefore, any packet destined to a public IP address will
  correctly reach the destination (usually). The same is not true for
  private IP addresses. 

* The IP address ranges set aside for private addresses are 10.0.0.0/8
  and 192.168.0.0/16. These two prefixes are never advertised outside
  of an organization where they are used. For example, IITB allocates
  IP addresses from the 10.0.0.0/8 prefix to its users, but no one
  outside IITB can recognize or route to this address. 

* How are private IP addresses connected to the Internet? Via a NAT
  (Network Address Translator). Suppose a user with a private IP
  address IP_priv opens a TCP connection to an external destination
  host (suppose the source port is PORT_priv). On the outgoing SYN
  packet, the NAT rewrites the source IP/port to IP_pub, PORT_pub and
  forwards the SYN packet. The NAT also stores this mapping it just
  created. When the SYN ACK comes back to IP_pub, PORT_pub, the NAT
  uses its mapping table to rewrite addresses back to IP_priv,
  PORT_priv and forwards the packet to the correct end host with the
  private address. In this manner, hosts using private IPs can
  initiate connections and talk to hosts outside. What about incoming
  connections? Does not work by default. Therefore, hosts that receive
  TCP connections (web servers, mail servers etc) are usually assigned
  public IP addresses.

* How many connections can a NAT support. A NAT can reuse the same
  public IP address for several different outgoing connections by
  using different port numbers. Because port numbers are 16 bits, a
  NAT can multiplex 2^^16 ~ 60,000 simultaneous connections on a
  single public IP address.

* What if a host behind a NAT wants to act as a TCP server and receive
  packets? This problem is called "NAT traversal". Even if we obtain
  public IP addresses for well-known services (e.g., web server, mail
  server), some hosts that participate in P2P communication (e.g.,
  Skype calls) will need to receive TCP connections. How can P2P
  protocols contact hosts behind NATs? Usually, a NAT also has a
  firewall that stops incoming packets to any public IP/port that it
  believes do not belong to any existing flow. Suppose two hosts A and
  B are both behind such NATs (+ firewalls), and they want to
  communicate with each other, say, as part of a P2P
  application. Without any external mechanism, such communication
  cannot happen.

* First let us consider NAT traversal for UDP traffic. One method
  followed is to have special servers called STUN (Session Traversal
  Utilities for NATs) servers. Hosts A and B communicate with the STUN
  server initially. This way, the server will know the external IP
  address and port of the hosts. It will then send this information to
  A and B. Now A and B will try to send a packet to the external
  addresses of each other. Suppose A sends a packet first. Now when
  B's packet reaches A, the firewall believes this is reply to A's
  packet and allows this packet through. (Even if A hasn't already
  sent the packet, B can retry periodically, and succeed within a few
  attempts.) This technique is also called NAT hole punching. Such
  elaborate techniques are used by P2P software like Skype to help
  hosts behind NATs to connect with each other.

* Basic premise of the above STUN-based method: the NAT uses the same
  external IP address and port for a given internal IP address and
  port of host A, irrespective of whether it is communicating with the
  STUN server or host B. However, some type of NATs called
  symmetric NATs pick a random port number for different connections
  to different destinations from the same internal source IP and
  port. NAT traversal is very hard with such NATs. In such cases,
  users must program the NAT (if allowed by the network administrator)
  to allow traffic on certain ports or allow certain traffic to pass
  through using special protocols that communicate with NATs. For
  example, with the UPnP (universal plug n play) protocol, hosts can
  request a specific public IP and port for their connections and
  convey it to the other end points, so that connections can be
  initiated. However, not all NATs and networks support such
  functionality.

* What about NAT traversal for TCP flows? TCP is a much harder problem
  even with non-symmetric NATs. With TCP, host A generates a SYN
  packet to B, but the reply (SYN ACK) cannot be generated by the
  application layer at B (it is in the control of the OS). So, if both
  end points start a TCP flow, both sides will send SYN and not get
  SYN ACK as a reply because the SYN will be dropped by the other
  NAT. So NAT traversal is a much harder problem; out of scope for
  this course.

* Even though NATs are a pain to deal with, NATs have helped to
  significantly slow down the IPv4 address depletion problem. As a
  result, IPv6 deployment hasn't been as fast as expected.

* Another concept related to private IP addresses: VPN (virtual
  private network). Suppose you have two different locations that use
  a private IP address space (e.g., you live off-campus, and want to
  have a 10.* private IP address from IITB in your home so that you
  can connect to other private IP addresses within IITB).  A VPN
  connects two such islands of private IP addresses over the public
  Internet. Another use case: you have two different locations that
  have upgraded to IPv6, but must be connected by a part of the
  Internet that does not support IPv6.

* How does a VPN work? By using a concept called IP-in-IP
  tunnelling. That is, you can tunnel an IP packet inside another IP
  packet. For example, consider a network path A-B-C-D. The links A-B
  and C-D are over the private IP space, and B-C is over public
  IPv4. Now, when B gets a private IP datagram destined to D, it puts
  it inside (as payload) another IPv4 packet whose destination address
  is C. When C gets this regular looking packet, a field in the header
  will indicate that the payload is another IP datagram (not a TCP
  segment for example), so C removes the outer IP header and forwards
  the original private IP datagram to D. This idea of tunneling is
  widely used in providing VPN access to private address spaces inside
  an organization. In addition to tunneling, VPNs also encrypt data on
  the public Internet to provide security.

* Is an IP packet always guaranteed to reach the destination? Several
  errors may happen. For example, an IP packet may be stuck in a
  forwarding loop (usually due to inconsistent routing tables; we will
  see why they occur in the next lecture). To avoid such packets from
  staying in the network forever, IP datagrams have a TTL (Time To
  Live) field, usually set to 64 by default. Every time a datagram
  crosses one IP hop, its TTL is decremented by 1. An IP datagram is
  discarded once its TTL hits 0. Therefore, even if a loop is present,
  a datagram can loop over a maximum of 64 hops. Most correct network
  paths are usually shorter than 64 IP hops.

* More sources of error: An IP datagram may reach a router that does
  not have an entry for the destination and hence does not know what
  to do with the packet. In such a case, the router that fails to
  forward the packet sends a "host unreachable" error message to the
  source.

* All these error messages (host unreachable, TTL expired etc) are all
  sent using a protocols called ICMP (Internet Control Message
  Protocol). ICMP consists of error messages and other such
  information at that network layer. Whenever an error occurs at the
  network layer, an ICMP message is sent to the source. ICMP message
  is carried as a payload in an IP datagram. Common ICMP messages are
  echo reply (in response - to echo request ICMP message, used by
  ping), TTL expired (used by - traceroute to discovers routers at
  various hop lengths).
  
* How does traceroute work? The source running traceroute sends IP
  datagrams with increasing TTL values (1,2,3..). An IP datagram with
  TTL N will expire at the Nth hop IP router, which will send an ICMP
  message back to the source, using which the source can know which is
  the router at the Nth hop.

* How does ping work? When you send a ping to destination host X, an
  ICMP request is sent to X. Host X then sends an ICMP reply back to
  the source IP address which sent the ping. The ping utility
  calculates the delay, loss etc and displays to the user.

* IP fragmentation. Usually, the payload (say, TCP segment) is sized
  so that one TCP segment fits inside one IP datagram and reaches the
  receiver intact. However, in some cases, if the maximum transmission
  unit (MTU) along the path is incorrectly estimated, then an IP
  datagram may become too big to be send in one piece over a link. In
  that case, the datagram is fragmented. All fragments of one datagram
  are given the same identifier. Each datagram has an offset
  associated with it, specifying where it must be inserted during
  reassembly. The last fragment has a flag indicating all pieces are
  done. Fragmentation can happen anywhere along the path, reassembly
  of all datagrams happens at the receiver.In general fragmentation is
  avoided because it is expensive, and opens up lots of attacks where
  weird fragments are sent causing receiver to collapse while trying
  to fit them together.

* What are all the fields in the IP header?

  Version - 4 bits
  Header length - 4 bits
  TOS (type of service) - 8 bits
  Datagram length - 16 bits

  Fragmentation related (identifier, offset, flags) - 32 bits

  TTL - 8 bits
  Protocol (TCP, UDP etc) - 8 bits
  Header checksum - 16 bits

  Source and destination IP addresses - 32 + 32 bits
  
  
