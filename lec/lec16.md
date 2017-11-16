BGP: Traffic Engineering, Security, Convergence, and Demo
=========================================================

* Recap of BGP from last class, and continue discussion on BGP: Who
  needs to run BGP? Does a stub organization have to run BGP? In the
  common case, a small organization that buys internet service from an
  ISP just has a static route pointing to the ISP, and gets a small
  chunk of the ISP's IP address space. The ISP takes care of
  advertising a (usually) larger prefix that covers the IP space of
  several customers in one aggregated entry.

* However, many organizations try to have more than one ISP for
  redundancy. This is called multihoming. In this case, the
  organization usually prefers to get its own IP address block, and
  announce it over BGP to several ISPs. Goals of multihoming: load
  balancing across ISPs so that the traffic rate is within the
  contract, to provide redundancy as primary / backup (use second ISP
  only if first is down), use specific ISPs for specific kinds of
  traffic (for example, some ISPs or backbone networks charge less for
  traffic between academic institutions).

* What happens if you use IP prefixes from one ISP's address space,
  while being multihomed?  Suppose your first ISP gives you a prefix
  A/24, and the ISP itself announces a larger A/20 prefix. Now, you
  take your A/24 and announce it via your second ISP also. Now, due to
  longest prefix match, your traffic will always arrive via the second
  ISP which is announcing A/24.

* How do you do load balancing with multihoming? Announce some
  prefixes via one ISP and rest via the other. If a prefix is
  announced via two ISPs, you can load balance on outgoing traffic
  (you can set your forwarding tables to send half the traffic on one
  link, and rest on the other). However, load balancing for inbound
  traffic is hard. That is, you have no control over which link the
  traffic destined to your hosts arrives on. It depends on how other
  routers compute their best paths.

* How do you do configure primary/backup links when you have
  multihoming? Suppose you have two ISPs, P and S, to serve as your
  primary and secondary ISPs. That is, you want traffic to come via S
  only if your link via P is down. If you announce the same prefix via
  P and S, you have no control on which link traffic arrives on. So
  you can do some tricks using BGP. For example, you may announce an
  aggregated prefix (say, one /24) via S, and more specific prefixes
  (say, two /25) via P. So, by default, all traffic will come via the
  P link. Only when the /25 prefixes announced by P are withdrawn due
  to failure of the link via P, then traffic will come via the /24
  through S. Another trick is AS path padding. You can pad your origin
  AS number several times on the path announced via S (e.g., AS1 AS2
  AS3 AS3 AS3), so that it appears as a longer AS path and will not be
  picked by default.

* Aggregating routing table entries. About half the prefixes in
  Internet routing tables are more specific versions of larger
  prefixes. So people argue that a lot of these entries can be
  aggregated to save valuable space, especially in core routing tables
  that tend to be very large. However, note that about half these
  cases of a larger and smaller prefix both appearing are due to
  multihoming. That is, the larger prefix and the smaller prefix have
  different AS paths, indicating that some kind of traffic engineering
  is being done. In such cases, if you aggregate the prefixes, then
  the desired traffic engineering cannot be achieved. BGP has a field
  to indicate if the route can be aggregated or not, which should be
  respected by other BGP routers.

* BGP security: several security lapses occur because BGP has no way
  of authentication the origin of a prefix or the path. For example,
  any one can claim to own and advertise any prefix. Discussion of
  some security incidents: announce invalid prefixes for a short time
  to send spam, announce a prefix that you do not own (intentionally
  or unintentionally) and blackhole the destination, and so on.

* BGP routing convergence. When BGP routing information changes (e.g.,
  a new prefix or path has come up, or an existing one has gone down),
  BGP updates (announcements or withdrawals) are sent between BGP
  routers. What is the time taken from the time the change happens to
  the time that all routers know of and act according to this new
  information? This time is called the convergence time of a BGP
  update. Empirical studies have shown that this time is several tens
  of seconds. For example, researchers have inserted artificial path
  changes and observed how long it takes for these changes to get
  reflected in publicly available routing tables. This delay is around
  40-80 seconds for new routes coming up, and 80-200 seconds for old
  routes going down. It has also been observed that connectivity to
  the prefix is bad before convergence.

* What explains this high convergence time and issues with BGP? BGP
  undergoes what is called "path exploration" when a routing change
  happens. Since no router has complete knowledge of the topology, a
  few paths have to be tried before the correct routing tables are
  computed. While BGP does not have the routing loops and count to
  infinity problems with distance vector protocols, it does not have
  the quick convergence property of link state protocols either.

* Example: consider 3 ASes 1, 2, 3, all connected to each other and to
  a router R. Ignore the policy aspects of BGP for now. Initially, all
  ASes use their direct path to R, but also learn of the other 1-hop
  paths to R. Now suppose 1 discovers first that its link to R is
  down. Then it sends a withdrawal for its direct route, starts to use
  the route 2->R, and announced this to everyone. Now, this route will
  be come invalid when 2 discovers its link to R is down and withdraws
  the route. But in the meantime, 3 discovers its link to R is down,
  and decides to use the route 3->1->2->R that was announced by 1. So
  messages from everyone about their direct routes (and all other 1
  hop / 2 hop / 3 hop routes) going down have to reach everyone
  else. Only then will the routing table entries converge to the new
  state of no routes to R.

* Note that path exploration will be more as the connectivity of the
  AS graph increases. If the AS graph were a tree, then there would be
  no exploration, as everyone would have only one path between any two
  points. However, once some extra edges are added, path exploration
  starts to happen.

* In addition to the path exploration described above, routers also
  have a certain timer called the minimum route advertisement interval
  (MRAI). When an update to a prefix is sent by a router, it waits for
  MRAI before sending next update. This is to bunch updates related to
  a certain network event instead of flooding the network with
  information. The MRAI adds to the convergence delay, because nodes
  have to wait 30 seconds between successive steps in the path
  exploration.

* Not all updates seen in BGP are due to genuine network topology
  changes either. Sometimes a router gets overloaded and stops
  responding to BGP keepalive messages temporarily. The other BGP peer
  assumes the link is down and sends a withdrawal, followed by an
  announcement soon after. So the route goes between on and off states
  quite frequently. This is called "route flapping". BGP also has a
  mechanism to identify such unstable prefixes. Prefixes that change
  often are awarded a penalty, and updates to such prefixes are
  suppressed for a certain duration once the penalty crosses a
  threshold. This is called route flap dampening.

* Now, BGP changes lead to disruption of network connectivity. During
  path exploration, packets may often go down temporarily incorrect
  paths. Various pathologies can occur. For example, you can have
  transient forwarding loops where packets go around in loops and get
  discarded. Various measurements have observed the correlation
  between temporary bad connectivity to a destination, and the BGP
  updates for that destination.

