Looking ahead: some recent advances

* Software defined networking (SDN): the concept of the control plane
  and data plane of a networking element, and what it means to
  separate them. For example, consider a simple router. The control
  plane is the routing protocol that decides how to forward a packet
  to a destination. The dataplane does the actual job of implementing
  the control's plane decision, by matching every packet to the
  forwarding table entries.

* SDN version 1: The control plane is run in a centralized software
  controller. The controller makes decisions on what the dataplane
  should do, and communicates them to the dataplane switches via a
  standard protocol (Open Flow). SDN switches are very simple with a
  set of tables that specify what fields of a packet header to match,
  and what action to take for matched packets (match-action
  tables). Every new flow is sent to the controller for a decision,
  and the controller installs rules for the flow in the switches.

* What are the advantages of such a method? Controller is centralized
  and in software: can make smarter decisions with network wide view,
  can implement complex control plane protocols easily, and so
  on. Disadvantages: centralized controller can become bottleneck. Not
  many compelling usecases.

* Hardware SDN switches vs. software SDN switches. Hardware SDN
  switches are not widely available, so a software switch running on a
  Linux box can approximate a hardware SDN switch by implementing
  match action tables efficiently in software. Software SDN switches
  have made SDN adoption easier.

* With this version of SDN, the switches themselves are still
  constrained to run with existing/well known headers and
  protocols. For example, you can use a new routing algorithm to
  compute the path in the SDN controller. However, in order to convey
  the decision to the switch, the decision should still be based on
  looking up the standard header fields. For example, you cannot do
  routing based on application payload, but only using some field in
  the TCP/IP header.

* SDN version 2: programmable dataplanes. The dataplane switches no
  longer match on fixed header fields. Instead the protocols can be
  defined in a high level language (e.g., P4) and compiled to run on
  programmable switches. Now, one can define their own headers and
  protocols in software, and the hardware switch can be programmed to
  match on those headers.

* Software SDN controller + programmable dataplane switches = highly
  programmable 'software defined' network. The network protocols are
  no longer fixed but configurable by the user. The networking
  hardware only provides basic primitives like match-action tables,
  and how the matching is done is completely specified by software.

* New research topics: discovering the power of programmable
  dataplanes. What all can you do with it? People are just discovering
  the flexibility and power of this idea.

* Another recent topic: Network Function Virtualization (NFV) =
  running various network 'functions' like firewalls and load
  balancers in software rather than hardware.

