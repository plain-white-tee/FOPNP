# Notes and files for Foundations of Python Network Programming

### Chapter 2 - UDP

`udp_local.py` UDP Server and Client on the Loopback interface<br>
In this file, both client and server create sockets that will use UDP over TCP/IP with the line `socket.socket(socket.AF_INET, socket.SOCK_DGRAM)`<br>
Note the tuple used to bind the IP address and port number for both the server to listen on and the client to send to: `('127.0.0.1', port)`

#### Promiscuous Clients and Unwelcome Replies

`udp_local.py` is dangerous because it never checks that the reply is actually coming from the server.<br>
Start the server and then pause it with `ctrl-z`. Then start the client which causes it to wait for the reply from the suspended server.<br>
Start a new python session in a new terminal:

```
import socket
sock = socket.socket(socket.AF_INTET, socket.SOCK_DGRAM)
sock.sendto('FAKE'.encode('ascii'), ('127.0.0.1', client_port))
```

This is enough to cause the client to accept the 'FAKE' server response.

#### Unreliability, Backoff, Blocking, and Timeouts

`udp_remote.py` simulates a server that randomly drops packets and a client that exponentially backs off resending packets. It will wait a maximum of 2 seconds and then give up.<br>
`python udp_remote.py server ""` starts the server listening on any local interface.<br>
`python udp_remote.py client localhost` starts the client sending to localhost.

#### Connecting UDP Sockets

`udp_remote.py` uses `connect()` with the `send()` call to avoid having to do `sendto()` with an explicit address tuple every time.<br>
This also eliminates the promiscuity problem since once `connect()` has been run, the OS will discard incoming packets to the port whose return address doesn't match the address that was connected.

The two ways to write UDP clients that are careful about return addresses of received packets:<br>
    Use `sendto()` to send to a specific address and use `recvfrom()` while checking the return address against a list of servers you've sent requests to.<br>
    Use `connect()` with `send()` and `recv()` and let the OS filter unwanted packets. This only works for speaking to one server at a time.

Neither of these two approaches are a form of security. Packets can still be spoofed so the client will accept them.

#### Request IDs: A Good Idea

Added ID's to requests and responses helps avoid requests and responses getting out of sync and helps avoid spoofing.

#### Binding to Interfaces

Binding to an interface (ex: 192.168.0.1) limits which external hosts can communicate with the server, but programs running locally can communicate with any of the machines interfaces they want.

The IP stack thinks in terms of UDP 'socket names' which are always a pair linking an IP address and port number.

#### UDP Fragmentation

`big_sender.py` checks for MTU and prints an error if the packet exceeds it.

#### Socket Options

Options are accessed through the methods `getsockopt()` and `setsockopt()`<br>
Check the man pages for `socket`, `udp` and `tcp`.

Options are OS specific. Common ones are:<br>
`SO_BROADCAST`: allows broadcast UDP packets to be sent and received<br>
`SO_DONTROUTE`: only allows packets to be sent to hosts on subnets the computer is directly connected to (ie. no gateway needed)<br>
`SO_TYPE`: when passed to `getsockopt()` returns socket type - `SOCK_DGRAM` or `SOCK_STREAM`

#### Broadcast

`udp_broadcast.py` shows how to configure a socket to send on the broadcast address.<br>
`python3 udp_broadcast.py client "<broadcast>"` after the server is started, the client can send to the broadcast address and the server will receive it, and so will any other servers running on the same subnet.

### Chapter 3 - TCP

It takes three packets to setup a TCP connection - SYN, SYN-ACK, ACK.<br>
Another series are required to close a connection - FIN, FIN-ACK, ACK or a pair of FIN and ACK packets in each direction.

Active TCP sockets are described by the four-part coordinates: `(local_ip, local_port, remote_ip, remote_port)`<br>
This is how the OS keeps track of every active connections source and destination.

`tcp_sixteen.py` shows a simple TCP client and server.<br>
Unlike UDP, where `connect()` is only a local operation that takes place in memory, with TCP, `connect()` is a network operation that starts the three-way handshake.

`send()` and `recv()` are different in TCP. In UDP they either send or receive a datagram, which either arrives or doesn't. But TCP sends data in streams, and can split data into different size packets and reassemble them on the receiving end.

`send()` might not be able to send all of the data you give it. If it can't it will return the number of bytes sent. So `send()` needs to be inside of a loop to make sure it has sent everything. Something like this:<br>
```
bytes_sent = 0
while bytes_sent < len(message):
    message_remaining = message[bytes_sent:]
    bytes_sent += s.send(message_remaining)
```

`socket` provies the `sendall()` method, which handles all of this.<br>
But, there is no equivalent wrapper for `recv()` which is why it has to be inside of a loop. In this case, the `recvall()` loop.

#### One Socket per Conversation


