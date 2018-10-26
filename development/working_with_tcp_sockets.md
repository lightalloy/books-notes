## Chapter 1. Your First Socket
```ruby
require 'socket'
socket = Socket.new(Socket::AF_INET, Socket::SOCK_STREAM)
```
Creates a socket of type `STREAM` in the `INET` domain.  
STREAM - communicating via stream (provided by TCP).  
DGRAM - datagram, refers to a UDP socket.  

### Understanding Endpoints
When 2 sockets need to communicate they need to know where to find each other.  
Sockets use Ip-addresses to route messages to specific hosts.  

Ipv4 - 4 numbers <= 255
Ipv6

### Loopbacks
A virtual interface.
Any data sent to the loopback interface is immediately received on the same interface.
`localhost`, `127.0.0.1`

### Ports
Each socket must bind to a unique port number.
Defaults - http - 80, ftp - 21

### Second Socket
Creating a socket in the Ipv6 domain:
```ruby
require 'socket'
socket = Socket.new(:INET6, :STREAM)
```

## Chapter 2. Establishing Connections
TCP connections are made between 2 points.
One socket is a initiator (client), and another one is a listener (server).

## Chapter 3. Server Lifecycle
A server socket lifecycle: create-bind-listen-accept-close.

### Bind
Bind a port, low-level implementation:
```ruby
# First, create a new TCP socket.
socket = Socket.new(:INET, :STREAM)
# Create a C struct to hold the address for listening.
addr = Socket.pack_sockaddr_in(4481, '0.0.0.0')
# Bind to it.
socket.bind(addr)
```
`Errno::EADDRINUSE` is raised if the port is in use already.

#### What port should I bind to?
- Don't bind to ports in the 0-1024 range (reserved for system use)
- and 49,000-65,535 range (used by services who need port for temporary purposes)
- 1025-48,999 are ok
Look at the IANA list of registered ports and make sure your choice doesn't conflict.

#### What address should I bind to?
Most of the time you'll want `0.0.0.0`

### Servers Listen
```ruby
# listen to the incoming connections
socket.listen(5)
```
The argument is the number of pending connections your server will tolerate.
This list of connections is called `the listen queue`.
If the listen queue is full when the new client arrives, the `Errno::ECONNREFUSED` will be raised.
The number of pending connections can't be bigger then `Socket::SOMAXCONN`.
You may want to set it to maximum number: `server.listen(Socket::SOMAXCONN)`

### Server Accepts
```ruby
server = Socket.new(:INET, :STREAM)
#...
# bind, listen...
# Accept a connection.
connection, _ = server.accept
```
Accept is blocking call.
`accept` pops the next pending connection of the listen queue.
Accept returns an Array.
```ruby
server.accept
=> [#<Socket:fd 10>, #<Addrinfo: 127.0.0.1:41522 TCP>]`
```
The first - the connection, the second - Addrinfo (represents a host and port)

Each connection is represented by a new `Socket` object, so the server socket remains untouched and continues to accept connections.

#### Connection Addresses
remote_address - second return value from `accept` and `remote_address` of the connection.
local address - `connection.local_address`

Each TCP connection is defined by this unique grouping of local-host, local-port, remote-host, and remote-port. You cannot have two
simultaneous connections to a remote host if the local ports and remote ports are identical.

### The Accept Loop
```ruby
# create socket, bind, listen, ...
# Enter an endless loop of accepting and
# handling connections.
loop do
  connection, _ = server.accept
  # handle connection
  connection.close
end
```

### Servers Close
It's important to close connections because of the:
- resource usage (get rid of the stuff you don't need immediately, not relying on GC)
- os limit of open file descriptors

You can also close only writing of reading with `connection.close_write` or `connection.close_read`

`shutdown` - another type of closing, it'll also shut down the copies of the connection (made with `Socket#dup`, which is not a very common way to copy connections, usually `Process#fork` will be used)

### Ruby Wrappers
`servers = Socket.tcp_server_sockets(4481)` - returns 2 sockets, 1 to listen to Ipv4, the other - to listen to the Ipv6

#### Connection handling
```ruby
# Create the listener socket.
server = TCPServer.new(4481)
# returns an instance of `TCPServer`, which has some differences with the `Socket` interface.

# Enter an endless loop of accepting and
# handling connections.
Socket.accept_loop(server) do |connection|
   # handle connection
  connection.close
end
```

You can pass multiple listening sockets when using `Socket.accept_loop`

Wrapping all into one:
```ruby
Socket.tcp_server_loop(4481) { |conn| conn.close }
```

## Chapter 4. Client Lifecycle
- create - bind - connect - close

### Client binds
If the socket omits the call to `bind` (the client doesn't usually call bind) it will be assigned to the port in the ephemeral range

Clients don't call bind because they don't need to be accessible from a known port number - a client can connect to the server from any port.

### Clients Connect
```ruby
socket = Socket.new(:INET, :STREAM)
# Initiate a connection to google.com on port 80.
remote_addr = Socket.pack_sockaddr_in(80, 'google.com')
socket.connect(remote_addr)
```

If the client connects to the server which is not ready to accept connections, it will wait for a while and then raise `Errno::ETIMEDOUT` if the server remains unresponsive.

### Ruby Wrappers
```ruby
socket = TCPSocket.new('google.com', 80)
# with a block
Socket.tcp('google.com', 80) do |connection|
  connection.write "GET / HTTP/1.1\r\n"
  connection.close
end
```

## Chapter 5. Exchanging Data
It can be helpful to think of a TCP connection as a series of tubes connecting a local socket to a remote socket, which can send and receive chunks of data.

### Strams
TCP has a stream-based nature. That's why we pass `:STREAM` option when creating a socket. No `:STREAM` option => no tcp-socket.

A TCP connection provides an ordered stream of communication with no beginning and no end.

## Chapter 6. Sockets Can Read
### Simple reads:
```ruby
# inside a tcp_server_loop
puts connection.read
```
But this way the server will never finish reading the data and never exit.

### Read Length
```ruby
while data = connection.read(one_kb) do
 puts data
end
```
You can read a certain amount of data, but if the client responds with less than that amount, the server will continue waiting.

### The EOF Event
```ruby
client.write('gekko')
client.close # close the socket => send the EOF
```

### Partial Reads
You pass the maximum length of data to the `readpartial`
```ruby
Socket.tcp_server_loop(4481) do |connection|
  begin
  # Read data in chunks of 1 hundred kb or less.
    while data = connection.readpartial(one_hundred_kb) do
    puts data
  end
  rescue EOFError
end
```

## Chapter 7. Sockets Can Write
```ruby
Socket.tcp_server_loop(4481) do |connection|
  # Simplest way to write data to a connection.
  connection.write('Welcome!')
  connection.close
end
```

## Chapter 8. Buffering
### Write Buffers
When you `write` successfully it doesn't mean that the data is sent. It only means that the data was passed to the OS kernel.
The kernel will decide whether to send it as is or combine it with other data.
Buffering is helpful for perfomance, cause the network is very slow.
So you can `write` as much as you need and the kernel will figure out the rest.
(unless you send large files or bigdata, then you need to split it cause of the RAM)

### Read Buffers
The optimal size to read dependa on the data you read.
Mongrel, Unicorn, Puma, Passenger, and Net::HTTP do `readpartial(1024 * 16)` (16kb)
redis-rb reads 1kb

## Chapter 9. Our First Client/Server
## Chapter 10. Socket Options
### SO_TYPE
Socket type
```ruby
socket = TCPSocket.new('google.com', 80)
# Get an instance of Socket::Option representing the type of the socket.
opt = socket.getsockopt(Socket::SOL_SOCKET, Socket::SO_TYPE)
opt.int == Socket::SOCK_STREAM #=> true
opt.int == Socket::SOCK_DGRAM #=> false

# same
opt = socket.getsockopt(:SOCKET, :TYPE)
```

### SO_REUSE_ADDR
- it's OK for another socket to bind to the same local address that the server is using if it's currently in the TCP TIME_WAIT state.

### TIME_WAIT state
- when you `close` the socket, but there is some data in the buffers.
If another socket tries to bind on the same address, the `Errno::EADDRINUSE` will be raised.
But if you pass `SO_REUSE_ADDR`, you'll allow you to bind to an address still in use by another socket in the TIME_WAIT state.

```ruby
server.setsockopt(:SOCKET, :REUSEADDR, true)
server.getsockopt(:SOCKET, :REUSEADDR) #=> true
```

## Chapter 11. Non-blocking IO
### Non-blocking Reads
`read` - blocks until it receives EOF
`readpartial` - returns available data immediately, blocks until then
`read_nonblock` - never blocks

`Errno::EAGAIN` will be raised if there is no data available to read.

Calling `IO.select` with an Array of sockets as the first argument will block until one of the sockets becomes readable. So `retry` will be called when there's available data to read.
```ruby
begin
  connection.read_nonblock(4096)
rescue Errno::EAGAIN
  IO.select([connection])
  retry
end
```
Using `IO.select` gives the flexibility to monitor multiple sockets simultaneously or periodically check for readability while doing other work.

`read_nonblock` method first checks the buffers. If there's some data, it returns immediately.
If not, it asks the kernel for the data (using `select(2)`). Reads, if data is available. If not available - `read` blocks, `read_nonblock` raises an exception.

### Non-blocking Writes
`writes` - writes all data available
`write_nonblock` - may return a partial write

Write blocks when:
- The receiving end of the TCP connection has not yet acknowledged
receipt of pending data.
- The receiving end of the TCP connection cannot yet handle more data.

### Non-blocking Accept
Accept just pops a connection off of the listen queue.
If there's nothing on the queue, `accept` will block, `accept_nonblock` will raise `EAGAIN`

### Non-blocking Connect
If connect_nonblock cannot make an immediate connection to the remote host, then it actually lets the operation continue in the background and raises `Errno::EINPROGRESS`

## Chapter 12. Multiplexing Connections
- working with multiple active sockets at the same time.
Prevent ourselves from getting stuck blocking a particular socket.

`IO.select(for_reading, for_writing, for_writing)`
first argument - an Array of IO objects which you want to read from
second argument - an Array of IO objects which you want to write to
third argument - an Array of IO objects for which you are interested in exceptional conditions

returns an Array of Arrays Arrays.
first element - array of IO objects that can be read from w/o blocking
second - array of IO objects that can be written to without blocking
third - array of IO objects which have applicable exceptional conditions.

`IO.select` is blocking (sync method call).
Blocks until the status of one of the passed `IO` objects changes.

Fourth argument - timeout, prevents from blocking infinitely.
If the timeout is reached before any of the IO statuses have changed, `IO.select` will return `nil` .

You can also pass plain Ruby objects to IO.select , so long as they respond to `to_io` and return an IO object

### Events Other Than Read/Write

### High Performance Multiplexing
There're more performant ways to do multiplexing with lots of connections besides `IO`. Most OS kernels have other methods than `select`, e.g. `epoll`(Linux) or `kqueue`.
E.g. `nio4r` gem.

## Chapter 13. Nagle's algorithm
- optimization applied to all TCP connections by default
Applicable to apps which don't do buffering and send small amounts of data at a time.

If you're working with a protocol like HTTP, where the request/response are large enough, this algorithm will take no effect. It's benefitial for the specific situations like implementing a telnet.

Every Ruby web server disables Nagle's algorithm by default: `server.setsockopt(Socket::IPPROTO_TCP, Socket::TCP_NODELAY, 1)`

## Chapter 14. Framing Messages
Client/server should have an agreed-upon way to frame the beginning and end of messages, so that they wouldn't have to open a new connection each time.

Reusing connections across multiple messages is the same concept behind the familiar keep-alive feature of HTTP.

### Using newlines
`request = connection.gets`

One real-world protocol that uses newlines to frame messages is HTTP.

### Using A Content Length
The sender of the message first denotes the size of their message, packs that into a fixed-width integer representation and sends that over the connection, immediately followed by the message itself.

Using this method the client/server always communicate using the same number of bytes for the message size.

## Chapter 15. Timeouts
`IO.select([connection], nil, nil, timeout)` - select with timeout.

These IO.select based timeout mechanisms are commonly used, even in Ruby's standard library, and offer more stability than something like native socket timeouts.

## Chapter 16. DNS Lookups
```ruby
socket = TCPSocket.new('google.com', 80)
```
Ruby needs to do the DNS lookup, which may be slow and it can block your entire Ruby process.

### MRI and the GIL
GIL - when 1 thread is active, the others are blocked.
But it understands blocking IO, when a thread is doing a blocking IO, another thread can continue execution.

But any time that a library uses the C extension API, the GIL blocks any other code from execution.
Ruby uses a C extension for DNS lookups, so it won't release the GIL!

`resolv` is a pure-ruby implementation.
`require 'resolv-replace'` - monkeypatches the Socket classes to use 'resolv'.


## Chapter 17. SSL Sockets
SSL provides a mechanism for exchanging data securely over sockets using public key cryptography.
You can upgrade "plain" sockets to secure SSL sockets.
A single socket can't do both SSL and non-SSL communication.
SSL-certificates.

The SSL source would provide you with the cert and key.
```ruby
# Create the TCP client socket.
socket = TCPSocket.new('0.0.0.0', 4481)
ssl_socket = OpenSSL::SSL::SSLSocket.new(socket)
ssl_socket.connect
ssl_socket.read
```
The server builds the SSL context and sets the `verify_mode`.

## Chapter 18. Network Architecture Patterns
### The Muse
One 'control' socket is used for sending FTP commands and their arguments between server and client.
Each time a file transfer needs to be made, a new TCP socket is used.
`connection.respond` writes response out on the connection.
FTP uses a `\r\n` to signify message boundaries, just like HTTP.

## Chapter 19. Serial
1) Client connects.
2) Client/server exchange requests and responses.
3) Client disconnects.
4) Back to step #1.

Advantage - simplicity.
Disadvantage - no concurrency, slow response

## Chapter 20. Process per Connection
After accepting a connection, the server will fork a child process whose sole purpose will be the handling of that new connection. The child process handles the connection, then exits.

`fork` - create a new process at runtime. A new process (child) will be an exact copy if the original (parent). Then they can go their separate ways and do whatever they need to do.

At any given time there'll be a single parent process waiting to accept connections and many children handling individual connections.

`Process.detach(pid)` after `fork` - stop caring about the child process, its resources will be fully cleaned up when it exits

### Considerations
Advantages: simplicity, complete separation (no locks or race condition).
Disadvantag: no upper bound for the forked processes. May be a problem for the large number of clients.

Examples - shotgun, inetd

## Chapter 21. Thread per Connection
Like Process Per Connection, but spawn a thread instead of spawning a process.

### Threads vs. Processes
Spawning: threads are much cheaper to spawn, they share memory.
Synchronizing: need mutexes, locks, and synchronizing access between threads when working with data structures that will be accessed by multiple threads.
Processes don't need them, cause they have their own copy of everything.
Parallelism: theads' parallelism is influenced by the MRI GIL.

### Considerations
Advantages: simplicity, no need to care about synchronizing (cause each connection is handled by a single, independent thread)
Lighter on resources.
Disadvantage: the number of threads can grow and grow until the system is overwhelmed.
Less parralel execution cause of the GIL.

Examples: webrick, mongrel.

## Chapter 22. Preforking
Forks a bunch of processes when the server boots up, before any connections arrive.
1) Main server process creates a listening socket.
2) Main server process forks a horde of child processes.
3) Each child process accepts connections on the shared socket and handles them
independently.
4) Main server process keeps an eye on the child processes.

The kernel balances the load and ensures that one, and only one, copy of the socket will be able to accept any particular connection.

Forwards an INT signal (interruption) received by the parent to its child processes, cleans up children.
```ruby
trap(:INT) {
  # kill child processes
}
```
In case a child exits (unusial situation):
```ruby
loop do
  pid = Process.wait
  $stderr.puts "Process #{pid} quit unexpectedly"
  child_pids.delete(pid)
  child_pids << spawn_child
end
```
Some preforking servers (e.g. unicorn) take more active role in caring in monitoring their children. E.g. they can see if a child process takes a long time to process a request. In this case the parent can kill the child process and spawn a new one.

### Considerations
Prevents too many processes from being spawned (not like in the "process per connection").
Advantage over a threaded pattern -- complete separation.
Disadvantage: more processes => more memory (they're expensive!)

Example - unicorn.

## Chapter 23. Thread Pool
Spawn a number of threads when the server boots and defer connection handling to each independent thread.
Same as "preforking", but with threads.

Example - puma.

## Chapter 24. Evented (Reactor)
1. The server monitors the listening socket for incoming connections.
2. Upon receiving a new connection it adds it to the list of sockets to monitor.
3. The server now monitors the active connection as well as the listening socket.
4. Upon being notified that the active connection is readable the server reads a chunk of data from that connection and dispatches the relevant callback.
5. Upon being notified that the active connection is still readable the server reads another chunk and dispatches the callback again.
6. The server receives another new connection; it adds that to the list of sockets to monitor.
7. The server is notified that the first connection is ready for writing, so the response is written out on that connection.

The Evented pattern is single-threaded so it needs to represent each independent connection with an object so they don't end up trampling on each other's state.

The connection stores the actual underlying IO object in its @client instance variable and makes it accessible to the world with an `attr_reader`.

When the Reactor reads data from the client connection, checks to see if it's received a complete request. If it has then it asks the `@handler` to build the response and assigns that to `@response`.

When the client connection is ready to be written to, it writes what it can from the `@response` out to the client connection

The `@handles` Hash looks something `{6 => #<FTP::Evented::Connection:xyz123>}`where the keys are file descriptor numbers and the values are Connection objects.

In the main loop:
- ask each of the active connections if they want to be monitored for reading or writing
- grab a reference to the underlying IO object for each of the eligible connections.
- pass these IO instances to `IO.select` with no timeout
- sneak the @control_socket into the connections to monitor for reading so it can detect new incoming client connections

Handle readable sockets:
- `@control_socket` is readable => it's a new client connection; `accept` and add `Connection` to `@handles`
- other readable socket => read the data and trigger the `on_data` method of
the appropriate Connection
- 'writable' sockets => call `on_writable`

### Considerations
Advantages:
- is able to handle extremely high levels of concurrency, thouthands of connections
- no need to worry about the syncronization
Disadvantage:
- never block the Reactor
E.g you need to retreive data from the remote server and the link is slow. That would block the Reactor!
You need to make sure that any blocking IO is handled by the Reactor itself.
The slow remote connection would need to be encapsulated inside its own subclass of `Connection` that defined its own on_data and on_writable methods.

It requires you to rethink all of the IO that your app does.

Examples:
- EventMachine
- Celluloid::IO
- Twisted

## Chapter 25. Hybrids
### nginx
- an extremely high-performance network server written in C, is often used in the Ruby world as an HTTP proxy in front of web application servers, but it can speak HTTP, SMTP, and others.

nginx uses the Preforking pattern, but in each of the forked processes it uses an Evented pattern.

### Puma
The puma rubygem provides "a Ruby web server built for concurrency".
At a high level Puma uses a Thread Pool to provide concurrency.

Puma's request handling is always done by a pool of threads (thead pool). This is supported by a reactor that monitors any persistent connections (`keep-alive`).

### EventMachine
- an event-driven I/O library.
At its core EventMachine uses the Evented pattern. There is a single-
threaded event loop that can handle network events on many concurrent connections. For the long-running or blocking operations it uses a Thread Pool.

#### (My) Questions
- What options can you pass when creating a socket in ruby?
- What's a loopback?
- What are the 2 points of the TCP connection?
- What's the server socket lifecycle?
- What port should you choose to bind a server socket?
- What is the argument for `socket.listen`, how do you decide it's value?
- What does the server accept do?
- What arguments does it return?
- What is the client lifecycle?
- Why doesn't client usually call bind?
- What happens if the socket doesn't call bind?
- How does the server read? What problems may appear?
- What's the difference between `readpartial` and `read`?
- Why buffer?
- How do you find out the amount of the data to send (at once)?
- -||- to read?
- what's the difference between `read`, `read_partial` and `read_nonblock`?
- when `EAGAIN` is raised?
- What's the benefit of using `IO.select`
- When would a `read` block and what does `read_nonblock` do instead?
- When would a `write` block?
- What's multiplexing?
- How does `IO.select` work?
- What arguments does `IO.select` accept?
- What does `IO.select` return?
- What happens if the socket receives EOF, Accept or Connect while being monitored?
- When is Nagle's algorithm useful?
- How to signal the end of one message and the beginning of another if you use one connection to send multiple messages (how to frame)?
- How to use `IO.select` with timeout?
- What network architecture patterns are there? (a server which handles multiple connections)
- How is Reactor pattern different from the others?
- What patterns does `nginx` uses at it's core?
- -||- `puma`? How does it handle keep-alive and non-keep-alive connections?