MAC: Switching, VLANs, MPLS
============================

* In the last lecture, we have seen that shared broadcast links like
  Ethernet LAN have a fundamental scaling limit. Solution: switched
  ethernet. That is, hosts connect to a switch via Ethernet cable.

* What is an Ethernet switch? When multiple hosts connect to the
  various ports in a switch, the switch forwards a frame only to the
  output port that leads to the destination (unlike an ethernet hub
  that forwards to all ports). How does a switch learn this
  information? When node A sends a frame through a port numbered "i"
  on the switch, the switch learns that MAC address of A is reachable
  via port "i". So the next time a frame arrives with destined to A's
  MAC address, the switch will forward the packet only on port
  "i". When a switch sees a destination MAC for the first time, it has
  to flood the frame on all ports, as it doesn't know the mappings.

* What is the difference between a bridge and a switch? A bridge is a
  term used for a device that connects 2 or a small number of LANs. A
  bridge performs similar functions as a switch (learns which MAC
  addresses are on which ports, and forwards only on the right
  ports). So a switch is just a multiport bridge.

* If the topology of switches / bridges in the network has a loop,
  broadcast frames will keep circling forever. To avoid this problem,
  switches in a LAN run the spanning tree protocol (STP). That is,
  switches use a distributed protocol to construct a spanning tree
  over all the switches, with the switch of the smallest ID (say, MAC
  address) as the root of the tree. Only ports of a switch that are on
  the spanning tree are active, and switches do not send or receive
  frames over the inactive ports. Switches re-run STP to construct a
  new topology as switches fail or topology changes.

* Why have switch topologies with loops in the first place? For fault
  tolerance: if one link fails, another can be used.

* Details on STP: initially the designated root bridge sends special
  messages (bridge protocol data units or BPDUs) on all its
  ports. These BPDUs are passed around the network, and every node
  tries to identify the shortest path to the root. When a BPDU is
  received on a port, the cost of that port is added to the path cost
  before the BPDU is forwarded on other ports. Note that the cost of
  the port is configured based on the link speed of the port etc.

When a switch receives BPDUs of different costs from the root on
different ports, it picks the port on which it got the shortest cost
as the forwarding port to the root, and blocks the ports on which it
received higher cost BPDUs. In the course of normal operation, the
blocking ports are not used to send or receive data (except BPDUs
periodically). When links fail or topology changes, the blocked ports
may change back to being used again.

* Switches (layer 2) vs. routers (layer 3):

Switches are better because

- Switches need no configuration (as they learn which ports to forward
  on automatically). Routers need to be configured with which prefixes
  they own etc. So switches are better for quickly interconnecting
  nodes.

- Switched networks can have heterogenity of link types etc., unlike
  IP routers which only work with the IP protocol.

- Switches perform simple lookup on fixed MAC addressses and layer 2
  protocols are also simple (learning bridges etc). So, chepaer than
  IP routers that perform longest prefix match and complicated routing
  protocols.

Routers are better because

- Switches require lots of broadcasts (first packet to a destination,
  ARP etc), and need to store per-MAC address state like ARP
  tables. As such, they don't scale well. Routers scale much better as
  they can aggregate IP addresses into prefixes, and need not flood.

- Switched paths are not guaranteed to be optimal (for example, even
  if a shorter path exists you may not take it due to spanning tree
  restrictions), whereas intra-domain IP paths are always shortest
  paths.

* In conclusion, small networks are usually built as switched LANs,
  and larger networks have IP routers and perform layer 3 routing.

* Brief discussion of WiFi link layer: note that an access point can
  either be a layer 2 switch or a layer 3 router (and NAT too). What
  role it plays depends on the configuration.

* We have seen that switched LANs have to broadcast packets over the
  entire domain ocassionally, leading to scalability issues. One
  solution: virtual LANs. When a switch is configured with VLANs,
  broadcast packets are exchanged only between hosts of a particular
  VLAN, and not across the entire domain. In other words, a VLAN
  redefines the broadcast domain of a node. VLANs can be configured as
  port-based (certain ports of a switch are designated to certain
  VLANs), or MAC address-based (certain MAC addresses are mapped to
  certain VLANs). A special VLAN tag is added to packets by the first
  switch, so that all switches know how to deal with the packet. 

* ATM and virtual cirtuit switching. ATM is a parallel network stack
  to layer2/3 of IP. When it was first developed, ATM was considered a
  serious competetitor to packet switched IP networks. ATM is based on
  the concept of virtual circuit switching. When A wants to start a
  flow to destination D, it sends a setup message that is routed along
  the correct path to D (say, via ATM switches B and C). After the
  virtual circuit is setup is complete, state is established along the
  path on how to forward data of this circuit. Every subsequent packet
  only carries a virtual circuit identifier (VCI) and intermediate
  switches will forward data using this VCI. ATM uses fixed length 53
  bute frames called "cells".

* Note that a VCI need not have global scope (i.e., unique across the
  network). VCI in ATM only has "link local" scope. That is, flows
  over a given link need to have unique VCIs. So, when a node sends a
  cell on a link for the first time, it picks a VCI that is not in use
  at that link. The receiving switch notes the incoming VCI, picks an
  outgoing link and VCI again. So, switches in ATM maintain a table
  mapping incoming link and incoming VCI to outgoing links and
  outgoing VCI. Whenever a cell arrives on the link, an ATM switch
  looks up the entry corresponding to the incoming link and VCI, finds
  out the outgoing link for that cell, and rewrites the outgoing VCI
  number.

* ATM was considered to be better than datagram based IP networks
  because of fixed length VCI and cells, leading to bounded latencies,
  and the benefit of setting up circuits to provide guarantees
  services. However, the best effort Internet performed reasonably,
  and ATM lost out to IP. 

* MPLS (multi protocol label switching) is a more recent technology
  that borrows some aspects from ATM. MPLS is not based on circuit
  switching, and is designed to work with packet switching and IP
  datagrams. However, it wishes to modify the forwarding logic of IP
  datagrams. At some point, it was thought that IP lookup algorithms
  based on longest prefix match would be too slow to forward data on
  high speed links, and that a fixed label lookup was needed. This was
  the initial motivation for MPLS. 

* MPLS works similar to ATM in the forwarding path. There is no
  connection setup. When an IP datagram arrives into an MPLS enabled
  network, the first MPLS edge router introduces a label on the
  packet. Sunsequent MPLS routers perform label switching, much like
  ATM cells. That is, all MPLS routers along the path maintain mapping
  from incoming label and link to outgoing link and label, and swap
  labels on packets. So MPLS routers are also called label switching
  routers (LSRs). Note that ATM switches can be easily reused to be
  LSRs.

* Where is MPLS label added? MPLS has a 20 bit label in a 4 byte
  header. This header is usually placed between IP and layer 2
  headers, so MPLS is considered layer 2.5.

* While the initial motivation for MPLS wasn't strong (IP lookup
  algorithms became fast enough), MPLS has found other uses. Some of
  these are listed below.

- Traffic engineering. MPLS can used to "pin" different IP flows to
  the same destination to different label switched paths (LSPs), to
  distribute load evenly and perform traffic engineering in ISP
  backbone networks. Explain with the example of the "fish topology":
  packets to the same destination arriving from different sources can
  be pinned to different paths.

- MPLS can be used for fast reroute, to compute alternate paths in
  case of link failures before STP recovers and finds an alternate
  path.

- MPLS can be used to build Layer 2 and layer 3 VPNs. The concept is
  similar to IP-in-IP tunneling. That is, to connect two private
  networks, the IP datagram in the private space is encapsulated with
  an MPLS header and tunneled to the other end point, after which the
  MPLS header is removed. However, MPLS based VPNs are more efficient
  because the MPLS header (4 bytes) is smaller than the IP header
  overhead (20 bytes) of IP-in-IP tunneling.

- In general, MPLS has found many uses because the labels can be
  refurbished to mean different things and serve different
  purposes. It can be used at any point where simple destination IP
  based forwarding is not good enough.

* How are MPLS labels distributed? That is, once a flow with an MPLS
  label arrives at a MPLS router, how does it decide the outgoing
  port/label of the flow? It depends on the purpose of using MPLS. For
  simple destination based forwarding, labels can be announced as part
  of the routing protocol. For example, a downstream router can tell
  an upstream router that it can forward packets of a certain label to
  a certain destination. For traffic engineering, the network operator
  can decide optimal routes and assign labels. For VPN services,
  labels are distributed with BGP. So distribution of labels requires
  some routing protocol, much like normal IP.

* What other alternatives to MPLS for traffic engineering? OSPF ECMP
  (Equal Cost Multi Path) can take advantage of multiple paths to a
  destination and split traffic among multiple paths. ECMP works fine
  for the most part, except when there is large asymmetry in the
  network traffic patterns / link capacities etc. For example, when a
  large flow starts on one of the paths, equally splitting traffic
  over all paths may not be the best idea.





