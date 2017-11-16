Application Layer: Introduction to the Web
===========================================

* Application layer consists of all the useful things we do with
  computer networks. Examples: web, email, audio/video streaming over
  Internet, voice/video calls using Internet, file downloads from P2P
  applications etc. The next several lectures will study these
  applications.

* Applications are typically structured as client-server
  applications. When you browse the web, your browser is the "client"
  and the web server is the "server". The server is always on, waiting
  for clients. Clients approach the server when needed. Servers are
  maintained in data centers or server farms.

* In contrast, in P2P applications, all peers are equal. Anyone can
  come and go at any time. Sometimes we have hybrid architectures,
  where a centralized entity/server facilitates P2P interaction. E.g.,
  in bit torrent, a "tracker" informs you of the IP addresses of
  peers, after which the file download happens in P2P fashion.

* For now, let's focus on client-server architectures. A popular
  example is the World Wide Web. This lecture, we'll see how the web
  works. Two pieces: a browser/client and a web server.

* Network applications are typically user space processes. The
  transport layer and below is usually implemented in the operating
  system kernel. The kernel exposes the "socket" API to the user space
  programs. Application developers use the socket interface to send
  and receive "messages" (e.g., client sends server a message
  requesting a web page). Think of sockets as post boxes, and
  application layer messages as mails that you put and get from post
  boxes. The transport layer handles how the message is delivered,
  much like how the postal system handles how mail is delivered.

* What services does the socket/transport layer provide? TCP provides
  connection-oriented reliable in-order delivery of a byte
  stream. Useful to tranfer files/text. UDP just delivers messages,
  without any reliability promises. UDP is preffered by some real time
  applications, as retransmissions can add to delay. Security add-ons
  exist (like "Secure Sockets Layer" or SSL) using with secure
  applications (e.g., HTTPS) can be built. However, current transport
  protocols on the internet provide no bandwidth or delay
  guarantees. Bandwidth and delay is "best effort".

* If several sockets open on a machine, how do we know which socket a
  message is destined to (note: IP address takes care of delivering
  the message to a particular machine, but doesn't help beyond
  that). Answer: port numbers. 16-bit identifier that uniquely
  identifies the several open sockets on a machine. More on this
  later.

* Rest of the lecture, we will study the most popular application on
  the Internet. The World Wide Web: developed by Tim Berners Lee in
  the 1990s. The "killer app" for the Internet. The web consists of
  "web pages" connected by links, so you can browse a lot of
  information easily. The file transfer / download text / email
  existed before the web. The invention of the web was organizing
  information using hyperlinks, so that it is easy to use.

* Web page - HTML file, contains embedded images, videos, javascripts
  etc. Web pages are hosted on web servers (e.g., Apache). Web clients
  or browsers request web pages by specifying a "URL" or clicking on
  links in other web pages. URL has two parts: domain name of server
  (which is resolved to IP address), and the path of the file in the
  server's working directory. The URL can also optionally embed some
  data fields to fill in forms, any additional parameters that the
  server can look at etc.

* The communication protocol used by web clients and servers is Hyper
  Text Transfer Protocol (HTTP). Example: user types URL or clicks on
  a link. Browser resolves domain name to IP address, opens TCP
  connections, sends a HTTP GET "rquest" message to server (specifying
  the page to get). Browser sends a HTTP "response" saying OK, and
  then transfer the web page. Browser renders the HTML to display the
  page. This request/response message exchange is called the HTTP
  protocol. By adhering to this protocol, any compliant browser can
  communicate with any web server.

* Typically, a HTML page has multiple "embedded objects". Need to
  "GET" all of those as well. Browsers do all of this
  automatically. When multiple objects to GET, do browsers do them
  serially or in parallel? The answer is both. Typically, a browser
  opens multiple TCP connections (say, 4). Sends GET requests on these
  connections in parallel. Once a request finishes, it sends the next
  one on the same connection. Why this way? TCP connection setup has
  an initial cost, so want to reuse the same connection for next few
  requests also ("persistent" TCP connections). Also, multiple
  parallel/concurrent TCP connections lead to better utilization of
  bandwidth.

* Walk through example of how a web page is downloaded with/without
  persistent connections, with/without parallel connections.

* HTTP requests format: Starts with a request line. Typically GET
  (requesting a web page) or POST (posting some information) request,
  mentioning a URL and HTTP version. This main request line is
  followed by a variable number of "HTTP headers" specifying various
  things (e.g., whether persistent connections are preffered or not,
  browser model, preferred language etc). Then there is an empty line
  followed by an optional body message. 

* HTTP response has a response code (OK, error etc), followed by HTTP
  headers (metadata about the response like length, last-modified
  etc), followed by the actual data requested. Common HTTP response
  codes: 2xx indicates success (e.g., 200 OK), 3xx indicates
  redirection (301 Moved Permanently), 4xx is client error (400 Bad
  Request, 404 Not Found), 5xx Server error (503 Service Unavailable).

* Let's take a minute to ponder the difference between the protocol
  used by the application (HTTP: consists of requests and responses)
  vs. the actual data transmitted by the protocol (HTML-based web
  pages with various types of embedded objects). HTTP does not specify
  what the data should be, just specifies the protocol for
  communication. Generating the application data itself is a
  complicated matter (e.g., how do you encode audio and video files
  etc). The HTTP protocol itself is oblivious to this part (e.g., you
  can transfer and view a text file in your browser). That is, the
  HTTP application layer protocol is a small part of the "network
  application" ecosystem consisting of web servers, web clients,
  HTML-based and other types of application data etc.

* Note that HTTP is stateless. Every request from a client is
  independent and self-contained. Server does not have to maintain any
  state, and can forget about the client after sending a response.

* If HTTP is stateless, how does Amazon or Google recognize you even
  when you visit their webpages after a break? Answer: using a
  mechanism called HTTP cookies. A cookie is a separate identifier
  sent with HTTP requests. The first time you visit a webpage, the
  server creates a unique ID (cookie) for you and returns it as part
  of the HTTP response. Every subsequent HTTP request to that site
  will carry the cookie, so that the server can identify you. A local
  cookie database exists in your browser, a bigger database exists at
  the server.

* HTTP caching: if several people from the same organization request
  the same web page, why should we transfer it over and over again? We
  can have an intermediate server (usually called proxy server) that
  can "cache" the page and serve it on subsequent requests. Proxy
  server parses the HTTP request, checks the cache. If the page is
  found in cache, it serves from cache. Else it requests the
  server. Note that a proxy server "splits" the HTTP and TCP
  connections, client-proxy and proxy-server. (Is splitting
  necessary?) Proxy can do other tasks also like content filtering
  etc.

* Caching also happens in browsers. Browser checks before issuing request.

* Some HTTP headers help in implementing caching and cache
  validation. Most responses have a "Last-modified" header that
  specifies when the page was last modified at the server. Client can
  do a conditional GET request, where it specifies the "last-modified"
  time of the cached page it has. If the page hasn't been modified at
  the server, server sends a "304 Not Modified", else it sends the
  latest page. Server can expire pages after a while using the
  "Expires" header. Content freshness can also be checked using the
  "ETag" header (which is a type of checksum of the page). Client
  returns the ETag it has and server resends if page modified.

* FTP - precursor to HTTP. Conceptually similar. You connect to a FTP
  server, and can "get" or "put" files, after which data transfer
  happens. There are some differences: (1) FTP sends control and data
  through separate TCP connections. (2) FTP keeps state over a
  session, e.g., user's current working directory etc. HTTP subsumes a
  lot of the simple FTP functionality.

* Now, we will study how HTTP has advanced. The version of HTTP we
  studied (1.1) had features such as persistent and parallel
  connections. Subsequently, Google pioneeered a new protocol called
  SPDY, that has made its way to HTTP 2.0. The main goal was to lower
  latency of accessing web sites.

* Problems with HTTP which served as motivation for SPDY:

- HTTP can only send one request per connection (no parallelism in the
  absence of parallel connections). HTTP pipelining options exists
  (where you send multiple requests at once), but the multiple
  requests are served in FIFO order, leading to head of line blocking
  (that is, the first response blocks all other responses). So HTTP
  pipelining is seldom used.

- All requests are only client initiated. The server cannot help even
  if it knows the client needs a resoruce.

- Lots of redundancy in the HTTP headers. For example, most headers
  like "User Agent" do not change and don't have to be repeated.

* Solution: SPDY: layer between HTTP and SSL/TCP. The HTTP protocol
  still remains same. Some interesting features:

- Multiplexed streams. Multiple requests sent over same TCP
  connection. Prioritization in requests exist so that no head of line
  blocking happens for high priority requests.

- Server push (can indicate it is sending an object via a new header,
  and then push the object), or server hint (just hinting the client
  to download it)

- HTTP header compression

* Does it improve latency? Some research has found that the single TCP
  connection may sometimes get stuck in a bad state of congestion,
  especially in cellular networks. HTTP is better because only one of
  of N connections suffers. Other research says latency improves.

* Recent solution by Google: QUIC Quick UDP Internet Connection. Send
  HTTP requests over UDP. Do reliability and congestion control inside
  the application.

* Other approaches to HTTP latency: domain sharding (use multiple
  domains, so more parallel connections), redesign webpages so that
  important files get downloaded first, compress Javascript, CSS,
  iamge files. People have been working on these for a long time.
