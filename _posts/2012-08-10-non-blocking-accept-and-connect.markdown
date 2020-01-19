---
author: Cloud Fundoo
comments: true
date: 2012-08-10 15:41:58+00:00
layout: post
slug: non-blocking-accept-and-connect
title: Non-blocking accept() and connect()
wordpress_id: 254
tags:
- Client
- Linux
- Non Blocking
- server
- sockets
---

I was trying to do accept() and listen() in a non-blocking way. Pseudo code is below.

**Non Blocking Accept()**

1. Check whether listening fd is ready for read, by using select()
2. Once ready issue accept(), this will return a new socket fd

**Non Blocking Connect()**

1. Issue connect on socket fd
2. If it is returning EINPROGRESS, check whether fd is ready for write using select()
3. If it is ready check socket state using getsockopt()
4. If no errors, then socket connection is established

TCP connection establishment is a three-way handshake, see below. Actually it is a 4-way process, but usually server will combine both SYN and ACK in a single packet, so three way handshake.

![3 way handshake"]({{ site.baseurl }}/assets/3-way-handshake.jpg "3 way handshake"){:class="img-responsive"}

Client initiates the connection request by sending the SYN, this is what happens when it calls connect(), then it waits for ACK+SYN from server, when client gets ACK+SYN packets, it sends ACK packet and moves the socket to ESTABLISHED state, see TCP state diagram below.

![TCP States]({{ site.baseurl }}/assets/tcp_states1.jpg "TCP States"){:class="img-responsive"}

Only point at which client has to wait is when it waits for ACK+SYN packet from server, this is indicated by EINPROGRESS return code from connect() call. When  fd returns from select(), indicating ready for write, it means either some error happended or ACK is send(3-way handshake done) by client in response to ACK+SYN packet from server, we can check for error using getsockopt() call, if no errors then client socket is in ESTABLISHED state, ready for send/receive packets. Client need not call another connect() to ACK the ACK+SYN packet from server, it will be done by the stack/kernel.

On the server side, I was on the impression that accept() is the one which is sending ACK+SYN packet to client, so accept() pseudo code may not work properly, it may not eliminate one delay (sending ACK+SYN and waiting for ACK from client)

![TCP Queues]({{ site.baseurl }}/assets/tcp-queues.jpg "TCP Queues"){:class="img-responsive"}

Actually accept() works in a different way, stack/kernel is the one doing the 3-way handshake and then new connection availability is notified to the application(server). TCP stack maintains two queues, one incomplete connection queue and one completed connection queue. Connection which are in SYN_RCVD state(i.e. send ACK+SYN packet in response to SYN from client) are in incomplete connection queue, connections which are in incomplete connection queue are moved to completed connection queue when it receives ACK from client in response to ACK+SYN. Accept() will be successful only if there is any connection in completed connection queue, select() will return ready for read only if there is any connection in completed connection queue.

So Accept() is not the one which does 3-way handshake, but it simply returns new socket fd, if there any connections in completed connection queue, so our pseudo code will work correctly.
