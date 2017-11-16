Routing Protocols
==================

* What is routing? Determining the best path to reach a
  destination. Routing protocols form the control plane of the network
  layer, and responsible for coming up with a set of routes for any
  destination, and for picking the best route.

* Any routing protocol roughly consists of two phases: route
  advertisements using which information about destinations is
  exchanged via messages, and best route computation where a best
  route is chosen to be installed into the forwarding table.

* What are the criteria to pick a path? Usually, intra-domain routing
  protocols pick the shortest cost path. Every link has a metric, and
  the path that minimizes the sum of all link metrics is chosen as
  best path. How are metrics chosen? Can be simple hop count, based on
  bandwidth, latency, physical distance etc. That is, based on
  desirability of using the link. Inter-domain routing protocols may
  use other criteria like policy. Network operators may also wish to
  balance load across links (traffic engineering). But usually, the
  simplest way to pick path is shortest path or least cost path.

* Load-sensitive vs load-insensitive routing: most routing protocols
  do not pick best path based on congestion, as it can lead to
  oscillations and instability. That is, link metrics do not depend on
  current load level. For example, if some link is loaded and its
  metric reduces, all flows may move away from it, leading to load on
  another link, and all flows will move back again.

* Static vs dynamic routing: for small networks and/or networks that
  rarely change, manually setting the route once is sufficient. This
  is called static routing. For more complicated networks, dynamic
  routing protocols are used, that adapt routes in response to changes
  in network topology. We will study two classes of dynamic routing
  protocols: link state (LS) routing protocols, and distance vector
  (DV) routing protocols.

* Link state routing: "tell about your neighbors to everyone". Each
  node collects information about all its neighbors and the link
  metrics. This LSA (link state announcement) of every node is flooded
  through the entire network. So, at the end of it, each node has
  complete view of the network graph. Each node then independently
  runs Dijkstra's shortest path algorithm to get the shortest path to
  every destination, based on which it figures out the next hop for
  every destination.

* Distance vector routing: "tell about everyone to your
  neighbors". Every node exchanges a distance vector (a vector
  containing its estimate of distance to each destination) with its
  neighbors. Upon receiving a neighbor's distance vector, a node
  updates its distance vector by adding its link cost to neighbor's
  distance vector, and if a better path is found through neighbor, it
  updates its best route. This is called the Bellman-Ford update
  algorithm.

* Distance vector has "count to infinity" problem. Simple solution
  is "split horizon" or "poisoned reverse".

* OSPF (Open Shortest Paths First) is a popular intra-domain routing
  protocols that uses simple LS routing logic. RIP (Routing
  Information Protocol) is a simple DV protocol.

* Work out examples of LS and DV routing protocols in action.
