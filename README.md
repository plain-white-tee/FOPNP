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


