Introduction to socket programming
====================================

* Applcation layer interfaces with the lower layers via the socket
  interface. For example, a web server opens a socket, browser opens a
  socket, both sockets are linked to provide a datapipe
  abstraction. That is, whatever data is pushed in at one end comes
  out the other and vice versa. For example, all the HTTP request and
  response messages seen in last lecture and sent and received through
  sockets on browsers and web servers. 

* Any user space process can invoke the socket system call to create a
  socket. Sockets belong to two families: Unix domain sockets (for
  process to process communication on the same machine) and Internet
  sockets (what we will use). Internet sockets are further of three
  main types: stream socket (uses TCP to send the message given to the
  socket), datagram socket (uses UDP), and raw socket (sends the given
  packet as-is without any additional processing, don't worry about these).

* Every socket has a socket address: the IP address and a port
  number. Port number is a 16-bit number used to distinguish sockets
  on the same machine. Servers open sockets on well known ports, so
  that clients know how to contact them. For example, web servers
  listen on port 80, 20/21 for FTP data/control, 22 for SSH, 23
  Telnet, 25 SMTP (email) etc. Ports 0-1023 are reserved for these
  well-known services.

* Inside a process / application, open sockets are referenced by a
  number called the socket descriptor (much like a file descriptor
  when you open files). Whenever you have to refer to a socket to the
  kernel in a system call, you must quote this number. The socket file
  descriptor / handle is obtained when any process opens a socket
  using the "socket" system call.

* After a server creates a socket (by specifying its type, family etc),
  it "binds" the socket to a particular well-known IP address and port
  number.  After server binds to a particular port, it issues the
  "listen" system call to tell the lower layers to start listening for
  incoming requests. Whenever an incoming request arrives on the
  server's socket, it must do an "accept" system call. If the server
  calls accept, it will block till a request arrives.

* Clients create sockets, but need not pick a port number. Client
  sockets are assigned a random unused port number that is not
  reserved by the system. Once a client socket is created, the client
  "connects" to the server socket by specifying the server IP and
  port. This connection involves the three-way TCP handshake for TCP
  sockets, and nothing for UDP sockets.

* When the client connects, the server accepts the connection, a new
  socket is created at the server for communication with this
  client. The original listening socket continues to listen for new
  requests, and the new socket is used to send and receive from a
  particular client.

* Once the client and server sockets have been connected, both
  endpoints can read and write from sockets, much like they do from
  files. For example, you call read(socket number,...) and this call
  returns when there is some data to be read on the socket. Similarly,
  the write call writes data into a socket.

* Summary: main system calls for TCP communication: socket (for
  creation), bind, listen, accept (at server), connect (at client),
  read, write (at both). Demo: show a simple socket program. 

* UDP communication proceeds similarly, but without the connection
  establishment phase.

* Handling multiple clients concurrently at the server: the accept and
  read system calls are "blocking" (i.e., the server does not make
  progress until some data arrives), so how can a server serve
  multiple clients at the same time, without blocking for any one of
  them?

- Non-blocking sockets, along with polling or event-driven I/O (system
  calls like select, epoll).

- Blocking sockets, but have multiple threads/processes, one per
  active connection.


* Overview of the programming assignment 1.
