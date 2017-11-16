Interdomain routing
====================

* Routing in the Internet is typically hierarchical. Typically in an
  organization (Autonomous System or AS), hosts are grouped into
  subnets based on physical proximity. A subnet is connected by a
  layer 2 technology (e.g., Ethernet) to its first hop IP
  router. Multiple IP routers are connected to each other by
  point-to-point links or Ethernet. All these routers in an
  organization run an "intra-domain" routing protocol like OSPF (link
  state routing protocol) or RIP (distance vector protocol) to compute
  paths among themselves. In this way, every router in an organization
  knows how to reach other internal IP destinations.

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

* Why separate inter-domain and intra-domain? 

- For scale. The internet routing tables will become very bulky if
  every IP router runs shortest path to every IP prefix. Instead, with
  the interdomain and intradomain separation, each set of protocols
  needs to handle lesser information.

- For policy. Interdomain routing may not want to do simple shortest
  path, but complex policies based on business deals and trust, as we
  see below.

* History of BGP. Initially, when the Internet comprised of a small
  number of universities, the edge routers at these organizations just
  ran a simple routing protocol between them. These edge routers and
  few more routers added for connecting these (together called the
  Internet backbone) was managed by the US government for free. Soon,
  the internet expanded, and the internet backbone was
  commercialized. We now have Internet Service Providers (ISPs) that
  run a bunch of BGP routers (and intradomain routers to connect their
  organization) that connect various organizations. So the backbone of
  the internet is now composed of several ISPs. All the backbone
  routers have adopted the current version of BGP as the interdomain
  routing protocol since 1990s.

* BGP is a "path vector" protocol. That is, it is based on the
  distance vector philosophy, where you tell your neighbors about all
  the destinations you know of. However, you don't just tell the cost,
  but you tell the entire path to the destination. This addition of
  specifying the entire path avoids routing loops and counting to
  infinity (because if a neighbor is announcing a path through you
  already, you won't use that path). Like DV protocols, there is a
  route announcement phase (where you exchange path vectors) and a
  best route computation phase (where you decide what your best path
  is). These two phases have slight differences with general DV
  protocols (that do simple shortest path) in order to incorporate
  policy.

* What is the granularity at which the path is specified? Do you list
  all the IP routers on the path> No, that's too messy. The internet
  paths are listed as sequence of Autonomous System Numbers(ASN). An
  AS is an organization that can be considered as one unit for the
  purpose of interdomain routing. Every AS has a unique AS number. For
  example, IITB is an AS. An ISP is an AS. A large organization that
  is spread across many locations can have different AS numbers for
  each part. Routers inside an AS run intradomain routing, and routers
  across ASes run interdomain routing.

* So how does a path vector announcement in BGP look like? For every
  prefix, you list the AS path so far, and a few extra attributes for
  policy. For route selection, you pick the shortest AS path, along
  with some other decision criteria based on policy. 

* ASes are mainly of two types: stub ASes or end-user organizations,
  and ISPs that provide connectivity between these organizations. Of
  course, there is a thin line between the two. ISPs are classified
  into tiers 1, 2, and 3 (informally). AS to AS relationships are of
  two types: transit and peering.

* Transit relationship exists between ISPs and their customers. A
  customer pays an ISP some money for Internet connectivity. This
  means that the ISP takes the responsibility of announcing the
  customers prefixes over BGP to the rest of the internet. Note that
  advertising a route implies a promise to carry traffic, and traffic
  flows in the reverse direction of route advertisements. This means
  that traffic to the customer from other hosts in the Internet will
  flow via the ISP. Similarly, when the customer sends traffic, the
  ISP will have routes to several destinations, and will forward the
  customer's traffic onwards. "Buying service from an ISP" means that
  the ISP is getting paid for announcing your routes, bringing you
  traffic, and sending your traffic. This type of a provider-customer
  relationship is also called a transit relationship.

* Consider two organizations that have lots of traffic to each
  other. Instead of both of them paying a provider to forward traffic
  to each other, they may seek to establish a connection directly, and
  forward each other's traffic directly. Such a relationship is called
  a peering relationship. Peering is usually between similar ASes that
  have roughly equal traffic to each other, and does not involve any
  payments. It is intended to carry traffic between peers by cutting
  out the middleman ISP.

* Is your ISP always guaranteed be connected to everyone in the entire
  Internet, and have routes to every destination? Not in the case of
  smaller ISPs. Usually, the smaller ISPs buy service from bigger
  ISPs, which buy from bigger ISPs are so on. The ISPs at the top of
  the hierarchy are called tier-1 ISPs. By definition, a tier-1 ISP
  does not have any other ISP as its provider. All tier-1 ISPs peer
  amongst themselves. The customers of tier-1 ISPs are tier-2 ISPs and
  so on. [Draw an example topology with transit and peering
  relationships.]

* ASes or BGP routers which have BGP sessions between them are
  "neighbors" as far as the path vector routing protocol is
  considered. We will now revisit the question of what gets advertised
  to neighbors and how best path is computed.

* Policy decision: when do you advertise a route? Two rules. First,
  routes from customers and self routes (routes to destinations within
  an ISP or organization) are announced to all neighbors - because you
  want to provide as much visibility as possible. Two, routes about
  all destinations that you learn from your neighbors are announced
  only to your customers. 

* What is the logic behind these rules? Let's understand. Why not
  announce peer or provider routes to other peers and providers?
  Because you don't want to carry traffic on behalf of your peers and
  providers (you don't make any money from it). The only announcements
  that happen are intended to provide reachability to self and
  customers, and let customers and self reach everyone. No other type
  of connectivity suits business interests. Contrast this with an
  intradomain routing protocol whose goal is to provide shortest path
  connectivity between everyone.

* Policy decision: which routes do you prefer during best route
  selection? Even before shortest AS hop count, you have a policy
  based rule. Prefer customer routes > peer routes > provider
  routes. Why? If you use customer route, it means you will send
  traffic through customer, and customer pays for it. If you use peer
  routes, nothing lost nothing gained. If you use provider route to
  send traffic, you pay for it. So use the cheapest option. Typically,
  routes that come in on a link are marked with an attribute called
  "localpref" to indicate if it is a customer link / peer link /
  provider link. This attribute is checked first before seeing
  shortest AS hop coount.

* How do typical routes / forwarding paths look at AS level?
  Typically, you have zero or more customer to provider links, then
  you hit a peering link or a tier-1 provider, then you go down zero
  or more provider to customer links to reach the other end point.

* BGP sessions are of two types: eBGP (external BGP) between border
  routers in different organizations, and iBGP (internal BGP) between
  routers in the same organization. That is, an organization may have
  more than one BGP routers (each potentially talking to several other
  BGP routers in different organizations).
 
 - Each BGP router sends all the external routes it learns to all other
  internal BGP routers using BGP itself - this is called an iBGP session.
  So there may be several internal routers also in an organization that
  know of external routes. Typically, all BGP routers talk to each other
  and exchange information about external routes, so all internal BGP
  routers will have BGP routes from all external BGP routers. 
  
 - The internal and external BGP routers will also be running an
  intradomain routing protocol to populate routes to internal
  destinations. So when a packet comes in for the external
  destination, it is first directed to the internal BGP router using
  forwarding tables populated by the inrtadomain routing protocol. The
  internal BGP router looks up the BGP-based forwarding table. It may
  find several routes from several border BGP routers. It will pick
  the closest border router. Then the path to the border router will
  be determined by the intradomain protocol. So the forwarding is done
  by combining the intradomain routing tables with the interdomain
  forwarding tables.

* Some implementation details. BGP runs as a userspace process over
  TCP on a well known port (179). Two routers that want to become BGP
  neighbors (also called BGP peers, do not confuse with peering
  between ASes) open a TCP connection between them. Then, they
  exchange the path vectors (and other information) for all
  prefixes. This is called the full table dump. From then on, only
  updates (announcements, withdrawals, any changes) are
  communicated. Because BGP runs over TCP, reliability is
  assured. Periodically, the peers / neighbors also exchange heartbeat
  / keepalive messages to let the other end point know they are
  alive. Note that BGP messages between internal BGP routers are
  placed in IP datagrams and forwarded using the intradomain routing
  protocols, much like other traffic.

* The BGP routing table consists of prefixes followed by a set of
  attributes for each prefix. Here is are some important attributes
  associated with each prefix:

- Localpref: a local variable (specific to each link) which indicates
  whether the route came from a customer, provider, or peer.

- AS path: sequence of ASes the route has traversed.
-
- Next hop: the IP address of the next hop BGP router for this
  route. If this route is picked as the best route, traffic will be
  sent to this next hop. For eBGP sessions, the next hop is the BGP
  peer at the other end of the BGP session. For routes learned over
  iBGP, the next hop is the border BGP router that first introduced
  the route into the network.

- A few other attributes that we won't cover.

* BGP neighbors exchange their current best route with their peers
  subject to the policy we discussed earlier (announce customer and
  self routes to everyone, and all best routes to customers). When
  updates arrive from neighbors, a BGP router updates its best route
  for every prefix using the following criteria in this order.

- Pick the route with the highest localpref (prefer customer > peer > provider)

- Shortest AS path preferred

- A few other checks

- Lowest intradomain path cost to next hop border router

- Smallest router ID or IP address to break ties.

