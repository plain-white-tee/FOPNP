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

