Application Layer: DNS, CDNs, Server Design
============================================

* DNS: Domain Name System. What is DNS? DNS maps hostnames (e.g.,
  www.cse.iitb.ac.in) to IP addresses. How to design such a system?
  Initially, one file used to hold all mapping when the Internet was
  only a few hundreds of hosts. Now, such a centralized system won't
  scale. Instead we use a heirarchical setup for DNS. Let's look at a
  simplified resolution of the hostname www.cse.iitb.ac.in.

- All DNS requests go to one of the 13 root servers. The root server
  then redirects to one of the TLD (top level domains), here the
  server that handles "in" domain. That is, root server says "I can't
  provide you the IP address of www.cse.iitb.ac.in, but here is the IP
  address of the server that knows about hosts ending with .in".

- Next, you contact the TLD server for "in" domain, which gives you
  the IP address of the server for "ac.in". You walk down the
  hiererchy to the iitb.ac.in name server, to the cse.iitb.ac.in
  server, which gives you the IP address of the web server (www host)
  in its domain.

* The name server that is responsible for a certain domain (e.g.,
  ac.in, iitb.ac.in, cse.iitb.ac.in) is called the authoritative name
  server for that domain. There are usually more than one for
  redundancy. You can also outsource your auth server to a third
  party.

* Every group of hosts also has a "local DNS server" that does all
  this resolution of contacting multiple hosts for you on your
  behalf. For example, each department typically has a local DNS
  server (which is configured along with your default gateway). All
  DNS queries from individual machines are sent to this local server,
  which does the job of contacting various authoritative DNS servers.

* DNS responses can be cached. When you get a name->IP mapping, it
  comes with a TTL (time to live). You can reuse that mapping in that
  time without traversing the entire chain again.

* Note that DNS resolves not just names of web servers but also names
  of mail servers. For example, if you send mail to someone
  @iitb.ac.in, your SMTP mail server contacts the domain iitb.ac.in
  asking for the IP address of that SMTP mail server of that domain.

* DNS is also used for load balancing. For example, if you resolve a
  web server's name, you can get different IP addresses depending on
  which replica you are assigned to.

* DNS can also return multiple IP addresses in a response, in which
  case you can try contacting any of the hosts.

* DNS servers store different types of records. "A record" maps name
  to IP address, and this is the most important type of record. An
  "NS" record maps a name to another name, for example, when you are
  walking down the hierarchy. For example, when you contact the
  authoritative server for "ac.in" domain, it gives you an NS record
  that points to the authoritative server for iitb.ac.in {ac.in,
  iitb.ac.in, NS}, and an A record of the IP address of the auth
  server {iitb.ac.in, XXX.XXX.XXX.XXX, A}. With these two, you can
  contact the next authoritative server in the hierarchy. You also
  have other types of records: MX records for mail servers, and CNAME
  records to give an alias to a hostname.

* Next we move on to the topic of designing large scale servers for
  real services. The simple server socket program we saw could only
  handle one client at a time. It could not multiplex several
  clients. For example, if it decides to just "accept" new connections
  all time time, it will not have time to "read" from other
  sockets. Similarly, if it is reading from one socket, the other
  sockets could lay idle. So, at the very least, servers need a
  mechanism for multiplexing several client connections in an
  efficient manner. Several techniques exist for this (e.g., the
  server could create separate threads or processes for each client
  etc.). 

* In general, designing servers to handle a large number of clients is
  a challenging problem. Handling several sockets simulatenously is
  only one part of the puzzle. Typical web servers today do extensive
  processing on each request, e.g., read/write files or database
  tables, perform computation etc. For example, consider the web
  server of a online travel portal. The web server has to receive the
  user's request, check a backend database for information like ticket
  availability etc., run some computations for cheapest travel
  options, and return the response to the user. Each client request
  involves multiple steps. Therefore, in addition to handling several
  client sockets, the server must also efficiently juggle several
  operations on each of the client's requests. So a good web server
  needs to be able to multiplex multiple requests in an efficient
  manner, and serve as many clients as it can for a given hardware
  configuration. The performance of a server is measured by its
  throughput (how many req/s can it serve) and latency (how long does
  it take to serve a request). As load increases, throughput flattens
  out (or falls) and latency increases (mainly due to queueing
  delays). [Show graphs of throguhput, latency, capacity.]

* However, no matter how much multiplexing you do, at some point, some
  system resoruces of the server are bound to run out. That is, after
  the number of clients exceed a certain capaity, a single server
  cannot handle the load anymore, because say, the CPU is too busy or
  disk is the bottleneck or no more free sockets are available. This
  is called server overload. When the server is overloaded, its
  performance suffers. Requests fail, response times are high, this is
  called a "server crash".

* Several techniques exist to scale web servers. The most common idea
  is to have several "replicas" of the server in a server farm, and
  have a "load balancer" to distribute load between the replicas. The
  load balancer can do redirection at the DNS level, i.e., a DNS
  request can return several IP addresses, and the client can try to
  contact the servers in a round-robin fashion. Or, the load balancer
  can work at the application layer as a "proxy" server, receive the
  HTTP request from the client, and make a request to one of the many
  replica servers on behalf of the client. Or, the load balancing can
  happen transparently at the network layer. For example, when you
  contact a web server on a particular IP address, the load balancer
  located at the entry of the server farm rewrites the IP header to
  point to one of several server replica IP addresses. This way, even
  though the client believes it is talking to one IP address, it is in
  fact talking to several machines that are juggled by the load
  balancer. Load balancers also need to ensure same user goes to same
  server replica in some cases ("stickiness").

* Content Distribution Network (CDN). CDNs are third party
  applications that outsource content distribution from content
  providers. A CDN manages several geographically distributed replica
  servers that hold content from several web sites / content
  providers. Each user is directed to closest replica via DNS.

