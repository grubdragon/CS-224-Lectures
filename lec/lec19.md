Link layer: Introduction
==========================

* We will deal with the link and physical layers together, since they
  are usually designed in close coordination. We will only touch upon
  physical layer aspects in this course.

* What are the main function of the link layer? To transport an IP
  datagram between two IP hops. Links can be between end host and
  first hop router, or between two IP hops in the core. Different link
  layer technologies are united by IP. Link layers run on many
  different physical media. Examples: Ethernet over copper or fiber,
  wireless, long distance fiber links.

* The link layer is implemented in software + hardware. Some L2
  functionality is in software (in device drivers in the OS for
  example), rest of the L2 and PHY functionality is usually in
  hardawre. The link layer hardware consists of the network interface
  card (NIC) at the end hosts, and intermediate L2 network elements
  like switches.

* Link layers forwarding happens using Layer-2 or MAC addresses, which
  are 48 bits long. MAC addresses are unique across all adapters (a
  manufacturer gets a prefix, and ensures uniqueness within the
  adapters of the organization). A special broadcast MAC address is
  the address with all 1s - ff:ff:ff:ff:ff:ff. Link layer headers have
  the source and destination MAC addresses. The network adapter
  filters out non-broadcast frames that are not destined to its MAC
  address, except when configured to be in a special "promiscuous"
  mode.

* When forwarding an IP datagram, the IP layer gives the next hop IP
  address. The link layer then maps the next hop IP address to the MAC
  address of the next hop IP router, places this destination MAC in
  the link layer header and forwards the packet. How does a node know
  the MAC address of the first hop IP router? Using Address Resolution
  Protocol (ARP).

* How does ARP work? When A is communicating with a host B (say, its
  first hop IP router) for the first time, A sends a broadcast packet
  requesting to know the MAC address of B. Then the destination B
  replies with its MAC address to A. B also stores a mapping between
  A's IP and MAC, as it may need to communicate with A in the
  future. All other nodes which have mappings for A and B also refresh
  them. The ARP mappings are valid for a certain period and
  timeout. The ARP mappings are stored as part of the forwarding table
  itself - map from prefix to next hop IP, interface / link, and MAC
  address of next hop.

* The notion of a "broadcast domain": a set of nodes you can reach via
  a link layer broadcast. It is required that your next IP hop is
  within your broadcast domain, so that it can reply to your ARP
  query and enable communication.

* Note that one IP "link" need not be one physical link. There can be
  several physical nodes along this layer-2 path that forward the
  packet towards the destination MAC address / next IP hop.

* Example: consider a simple two-hop network A to B to C. When A has a
  datagram to send to C, the source and destination IP addresses in
  the IP datagram are always that of A and C. Then A learns that the
  next hop IP is B, figures out the MAC address of B via ARP, and
  places the MAC addresses of A and B as source and destination MAC
  addresses in the link layer header. When packet reaches B, B looks
  up its forwarding table, realizes that the next hop is C, and
  modifies the link layer header to have MAC addresses of B and C as
  source and destination MAC addresses (B learns of C's MAC address
  via ARP again). This is how the IP datagram finally reaches C. Note
  that the source and destination IP in the packet is always that of A
  and C, but the source and destination MACs keep changing with every
  IP hop.

* What are the main functions of the link layer for every transmission?

- Framing. Place an IP datagram (or any other network protocol packet)
  in a frame. Framing usually involves adding a special start symbol
  or preamble to denote start of frame, and an optional end-of-frame
  marker at the end of the frame. A frame is a unit of transmission at
  the link layer. A network datagram may or may not fit in one link
  layer frame. For example, ethernet frames are big enough to hold TCP
  segments if MSS is sized properly. On the other hand, some optical
  links use much smaller frame sizes, so need to partition datagrams.

  What if preamble occurs within the data part of a frame? Either
  escape it (break the pattern with a special symbol) or use fixed
  size frames, so you know where to look for preambles. Different link
  layer protocols use different frame formats. The only requirement is
  that an IP datagram should be transported.

- Error detection, correction, recovery. Link layers use techniques
  like parity bits, checksums, or Cyclic Redundancy Check (CRC) to
  detect bit errors in received frames. Ocassionally, error correcting
  codes can also be used to recover from bit errors. More wired link
  layers use only error detection (e.g., Ethernet uses 32 bit
  CRC). Wireless link/physical layers often use error correction
  because bit errors are more prominent.

  Some link layers also retransmit frames (stop-and-wait or sliding
  window) to recover from bit errors. Ethernet does not do
  retransmissions at the link layer, as packet errors are rare.

- Link access: Some link layers like point-to-point have only one
  sender at a time on a link. However, some link layers like Ethernet
  and wireless are shared broadcast media, and require protocols to
  arbitrate access to the medium.

