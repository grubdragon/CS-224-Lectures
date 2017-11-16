Network Layer - Introduction
=============================

* So far, we have studied the network edge (application and transport
  layers). Now we come to the network core. The network or IP layer
  takes transport layer payload (e.g., TCP segments), encapsulates it
  in an IP header to get IP datagrams. The IP datagrams have a source
  and destination IP addresses. The job of the network layer is to
  transport this IP datagram from the source IP to destination IP.

* The network layer runs on every node that has an IP address,
  including end hosts and IP routers that connect end hosts (unlike,
  say, the transport layer that only runs on end hosts). Consider a
  host A connected over an Ethernet LAN to its "first-hop" IP router
  R_A, that wants to communicate with host B, which is connected to
  its IP router R_B. Note that adjacent IP hosts (especially, end
  hosts and first hop IP routers) needn't be directly connected by a
  wire (point-to-point link). They can be connected by an Ethernet
  LAN, wireless LAN, satellite link, whatever. The link layer (layer
  2) takes care of moving the IP datagram from from IP hop to the next
  IP hop, e.g., from end host to its first hop IP router. The network
  layer deals only at the level of IP hops.

* Difference between a router and a switch? Generally, all network
  elements in a packet-switched network are packet switches, so IP
  routers are also packet switches, more specifically, layer 3
  switches. Link-layer switches or layer 2 switches connect nodes at
  Layer 2 and route between IP hops based on Layer 2 (MAC)
  addresses. To avoid confusion, we will refer to layer 3 switches as
  routers.

* Who gets an IP address? Every host? Well, technically, every
  interface. A host can have multiple network interfaces, each will
  have its own IP address. [Exercise: run the linux command "ifconfig"
  to see all the interfaces and IP addresses on your machine.] Why
  should each interface have a separate IP address (instead of just
  one IP per host)? Suppose a host has both a wireless interface and a
  wired interface. For traffic that must flow through the wireless, it
  will use the wireless interface's IP as the source IP (so that
  traffic comes back via wireless). If it used same IP for both
  interfaces, the host wouldn't have control over which traffic flows
  on which link.

* IPv4 addresses are 32-bit (4 byte) numbers, represented by the
  dotted decimal notation. Groups of IP addresses are called IP
  prefixes or subnets (subnetworks). E.g., 10.0.0.0/8 denotes a set of
  2^24 IP addresses that have the first 8 bits common. This number of
  bits in common is called the prefix length, and represented by a
  subnet mask (e.g., subnet mask of 255.0.0.0 denotes IP prefix length
  8). 

* How do you make subnets? Take the number of hosts that will be in
  close proximity, and allocate a subnet with enough hosts to manage
  them, so that they can all be served by the same LAN and by the same
  IP router. For example, suppose CSE has the 10.129/16 prefix. We
  want to create 4 contiguous subnets for 4 labs that have 200, 100,
  50, 50 computers respectively. Then, we take 10.129.0.0/23 (512
  addresses) and break it up into 4 subnets as follows: 10.129.0.0/24
  (256 addresses), 10.129.1.0/25 (128 addresses), 10.129.1.128/26 (64
  addresses), 10.129.1.192/26 (64 addresses).

* Special broadcast IP address: 255.255.255.255. Packets sent to this
  broadcast address reach all hosts in the "broadcast domain", which
  is typically all the hosts in a given LAN.

* Two functions of the network layer: routing (also called the control
  plane) and forwarding (also called the dataplane). The granularity
  of routing and forwarding is an IP prefix (instead of individual IP
  addresses) for scalability. That is, you make the same routing and
  forwarding decisions for all IP addresses in a prefix. Understand
  the various terms - routing protocols, routing tables, forwarding
  tables.

* A forwarding table maps an IP prefix to the next hop IP
  address. That is, for a given destination IP prefix, the forwarding
  table tells us which is our next hop IP router, and hence, which
  link / interface to send packet. At end hosts with one interface
  typically, the forwarding table is simple, and points to the first
  hop IP router and the single interface. [Exercise: run the linux
  command "route -n" at the command line and see your forwarding
  table.] At the routers, the forwarding tables are more complex,
  since a router has several interfaces (links), and the forwarding
  table helps us which outgoing link / interface to send packet on, or
  equivalently, what the next hop IP address is. How do we obtain the
  forwarding table on IP routers?

* A routing table maps an IP prefix to a set of possible network paths
  to reach that destination. Out of the various paths, the best path
  is chosen, and the next hop information is populated in the
  forwarding table. How to we obtain the routing table? You can of
  course manually configure routing and forwarding tables for small
  networks. What about large networks?

  [A note on longest prefix match: note that all prefixes in the
  forwarding table need not be disjoint. That is, you can have a
  forwrding table entry for the "larger" 10.0.0.0/8 and "more
  specific" 10.0.0.0/16 prefix. If you are looking for a particular IP
  address, you normally match it with the more specific entry, or a
  longest prefix match.]

* Routing protocols are algorithms (often distributed / decentralized)
  that run at various routers. Typically, it involves a route
  announcement phase, where each router conveys some information to
  the other routers (say, I can be used to reach so-and-so prefix),
  followed by a route computation phase, where this information
  collected from other routers is digested. Together, the routing
  protocols result in routers discovering several paths to a given
  destination, and hence building their routing tables. Link state
  (LS) and Distance Vector (DV) are two popular types of routing
  protocols. At the core, these are simple shortest path computation
  algorithms. 

* Routing in the Internet is typically hierarchical. Typically in an
  organization (Autonomous System or AS), hosts are grouped into
  subnets based on physical proximity. A subnet is connected by a
  layer 2 technology (e.g., Ethernet) to its first hop IP
  router. Multiple IP routers are connected to each other by
  point-to-point links or Ethernet. All these routers in an
  organization run an "intra-domain" routing protocol (like OSPF or
  RIP) to compute paths among themselves. For every router in an
  organization knows how to reach other internal IP destinations.

* What about IP destinations outside the organiztion? There are
  special "border" routers at the edges of organizations that connect
  to other border routers. Internet Service Providers (ISPs) run a
  bunch of borer routers that connect various "stub" organizations and
  other ISPs. These border routers run "inter-domain" routing
  protocols (BGP is the defacto standard today). These inter-domain
  routing protocols determine paths between organizations. The
  intra-domain and inter-domain routing protocols together fill up the
  forwarding tables in such a way that every IP router along the path
  can correctly route packets.

* Why separate inter-domain and intra-domain? For scale (do not want
  to run routing protocols over millions of routers) and policy
  (inter-domain routing involves more than shortest path
  routing). More on interdomain routing later.

* How are IP addresses allocated? One technique is to get IP addresses
  from your ISP. For example, suppose your ISP has 103.21.0.0/16. He
  may decide to give 103.21.0.0/24 to the customer network. The ISP
  will jointly announce the bigger /16 prefix. When traffic to the /24
  prefix reaches the ISP, the traffic is routed to the customer
  organization.

* When you have IP addresses from your ISP, what happens if you change
  ISPs? You have to renumber. Instead, you can apply and get your own
  IP address prefix from Internet registries. You can then announce
  your prefix via ISPs and the rest of the Internet can reach you.

* IP addresses were initially allocated in units of /8 (class A), /16
  (class B) or /24 (class C) addresses. However, class A prefix (2^24
  addresses) was too big for most organizations, and class C prefix
  (2^8 addresses) was too small. The class B addresses were running
  out fast. So today, we have a classless allocation of addresses -
  registries can allocate any length prefix that they wish.

* Once you have a prefix, how do you allocate individual IP addresses?
  You can allocate them statically or dynamically using DHCP. How does
  DHCP work? Whenever a client joins, it sends a DHCP discover message
  to the broadcast 255.255.255.255 IP address. ANy DHCP servers that
  listen to this message will send out DHCP offers. The client will
  accept one of the offers, and send a DHCP request message to the
  DHCP server whose request it has accepted. The server will then send
  DHCP ACK. The DHCP address has a validity.

* Sometimes the IP prefix you get may not have enough addresses for
  all your hosts, especially now due to IPv4 address exhaustion. What
  do you do? Have some public IPs and rest are private IP addresses,
  like in IITB. The private IP addresses are 10.0.0.0/8 and
  192.168.0.0/16. In IITB, we assign addresses to all hosts from
  10.0.0.0/8 prefix. Only web servers and other hosts that receive
  traffic from outside get public IPs.

* What is the problem with private IP? Private IP addresses do not
  have routes in the external Internet, so hosts outside cannot reach
  you. Even if you originate the TCP connection, you must still
  receive SYN ACK etc at your IP address. Solution: Network Address
  Translator (NAT). IITB has a NAT (several of them) at the edge. A
  NAT rewrites IP headers, replaces the private IP, port with a public
  IP, port. A NAT can handle several (more than 60,000) connections
  using one public IP and all port numbers. More on NATs later.

* A more permanent solution to IP address exhuastion is to move to
  IPv6. IPv6 has 128 bit addresses (instead of 32-bit in IPv4), so
  everyone (and everything!) can have an IP address. 

