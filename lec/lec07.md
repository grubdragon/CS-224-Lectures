Application layer: SMTP, P2P, web services
==========================================

* Another example of a client-server application: email or SMTP
  (Simple Mail Transfer Protocol). Two entities: user agents and mail
  servers. User agents (email clients like Thunderbird or Outlook)
  interface between users and mail servers. Mail servers are for a
  group of users.

* Example. Suppose A (userA@sender.com) wants to send email to
  B (userB@rx.com)

- User agent of A sends the mail to his mail server, say,
  mail.sender.com. (How? we will see later)

- Mail server of sender.com opens SMTP connection with mail server of
  rx.com. SMTP runs on TCP. How does it know the IP and port on which
  to open TCP connection? It resolves domain name to target mail
  server IP address using DNS; see below. Port number is standard for
  SMTP.

- Once mail is delivered, B uses his user agent to retrieve the mail
  (how?)

* Why split functionality between user agent and mail server? Why
  can't A and B run SMTP between their machines? Because machines
  can't be always on, may need to retry etc. So mail servers manage
  mail boxes of many users.

* SMTP has simple commands like HELO, MAIL FROM, RCPT TO, DATA, QUIT
  etc. to transfer the message. SMTP uses persistent TCP connections,
  so can send/receive multiple mails at once.

* Differences between HTTP and SMTP? HTTP is pull vs SMTP is
  push. HTTP has separate responses for each object, but all
  attachments and objects are sent as one mail in SMTP.

* SMTP is between mail servers. What about user agent to mail
  server. At sender side, we can use SMTP again. That is, A's user
  agent can be SMTP client and S's mail server is SMTP server. Note
  that A's mail server acts as SMTP server to A and SMTP client when
  sending to another mail server. 

* Can we use SMTP at receiver side?  No, because it is a push
  protocol. For receiver, we need pull protocol, where user
  periodically checks if he has any mail from his mail server. These
  protocols are called Mail Access Protocols. E.g., POP3, IMAP etc.

* Even HTTP can be used when you used between user agents and mail
  servers - this is webmail. However, mail server to mail server
  communication is always SMTP.

-------------

* What are web services? Any service you can get using the web or the
  Internet. Browsing news is one common service people avail. Maps is
  another. However the term web service usually refers to several
  machine-to-machine communications that happen over the Internet
  (beyond the human use of web for just browsing etc). For example,
  you can view your location on a map, and provide/view real time
  traffic. Here, you are not only consuming the map information, but
  you are also populating the database at the mapping service about
  speed and traffic from your side. The common term for what you can
  do with web services is: CRUD (create, read, update, delete) any
  piece of data over the Internet. Summary: web services refer to the
  generic way of exchanging information over the Internet (usually
  excluding the easy-to-understand case of human using the Internet).

* The nascent way to implement web services is via RPC (remote
  procedure call). You have a client that calls a certain procedure on
  a server with certain parameters. The client and server need to
  agree on the data format, APIs etc. Some web service standards are
  built along the lines of this RPC model. For example, SOAP-based web
  services (google up if you want to know more). A SOAP web service
  client is tightly coupled with the server, and they both agree on
  data formats, function calls etc.

* Newer and simpler ways exist to easily develop web services
  today. One such example is called REST (representational state
  transfer). REST uses HTTP protocol (can run on anything that
  supports viewing/updating URIs). It uses the 4 HTTP request verbs
  (GET, PUT, POST, DELETE) for reading, updating, creating, deleting
  respectively. For example, you can use GET to get information from a
  database server, or use PUT to update information at the server. The
  URL/URI contains information on what you want to get/put. REST is
  stateless and simple, while RPC is more generic/powerful but
  complicated to use.

---------------

* So far, we have seen client-server application architectures. What
  is the limitation? Scalability / cost. Suppose a server needs to
  distribute a large file to a large number of clients. The server
  needs to be very powerful and have a high bandwidth link. Instead,
  the server can send the file to a few clients, and the clients can
  help distribute to other clients. This is the idea behind
  peer-to-peer (P2P) architectures.

* Most popular P2P application: BitTorrent. How does BitTorrent work?
  The collection of all peers that participate in distributing a file
  is called a "torrent" or a "swarm". A file is divided into chunks
  (say 256KB) and peers download chunks from one another. Peers that
  have a chunk also upload the chunk to other peers.

* How to locate peers in a torrent/swarm? Each torrent has a
  centralized node called "tracker" (information about the swarm and
  tracker can be got from other means, say webpage). Every peer
  informs tracker when it joins, so tracker knows which peers
  exist. When a new peer joins, the tracker picks a random subset of
  peer and introduces them to the new peer (i.e., provides IP
  addresses of other peers). The new peer then establishes TCP
  connections to other peers.

* Peers exchange information on who has which chunks of files. Then a
  peer will request chunks that it does not have from other
  peers. Peers use "rarest first" policy. That is, they first download
  the chunk that has least presence amongst peers. This way, less
  popular chunks get distributed fast.

* In addition to downloads, peers also have to help other peers by
  uploading. BitTorrent uses a tit-for-tat policy, where a peer
  uploads most to those peers who are supplying it data at highest
  rate. It calculates these peers every few seconds and these peers
  are said to be "unchoked". Typically, 4 unchoked peers. A peer may
  also choose to optimistically unchoke a random peer. Lot of research
  done on the incentives of BitTorrent, can it be gamed etc.

* But the tracker is centralized and can become a bottleneck / point
  of failure. How to deal with it? We could build a distributed
  tracker that maps a file to a node/nodes that is/are responsible for
  it. We need this information to be distributed across several nodes,
  so that no one node is the bottleneck. The generalized version of
  such a distributed application is called a DHT (Distributed Hash
  Table). DHTs store (key,value) pairs. For example, in BitTorrent,
  the key is name of torrent identifier (say, movie name) and value is
  list of IP addresses of peers. These key-value pairs are not stored
  in a central database but distributed over several nodes. A DHT can
  survive even if some nodes join/leave etc.

* Let's design a simple DHT. What is the key step in a DHT? We should
  be able to map keys to nodes. Once we know which nodes are
  responsible for which set of keys, then any operation like creating
  / searching / updating / deleting a key-value pair can happen by
  simply going to the node responsible for the key, and requesting the
  specific operation. So the key step is, given key, identify
  node. 

* How do DHTs solve this problem? The basic recipe of all DHTs is as
  follows. Pick a name space (say 160 bit numbers), and map keys and
  nodeIDs in this region. For example, key=hash(movie_name), and
  nodeID = hash(node IP address). Now, assign every key-value pair to
  its closest (or immediately suceeding) node in the 160-bit
  space. This is called "consistent hashing". For example, if we
  consider 4 bit name space, let the keys to be assigned be 2, 7,
  10. Let the nodes have IDs 3, 6, 9. Then key 2 will be assigned to
  node 3, 7->9, 10->3. So every time you want to search / update a
  key, go to the node with the ID immediately after the key and ask
  for the key-value pair.

* But how do you know which node ID is close to which key? Must
  maintain a list of all nodeIDs and their IP addresses. Or, learn
  about a small subset of "neighbors" (e.g., nodes with IDs just
  before and after yours), and pass on the query. All DHTs have this
  basic tradeoff - the more neighbors you have, the faster you will be
  able to search/update a key-value pair.
