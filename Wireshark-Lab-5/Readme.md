# Lab 5 – Slow Application Response

## Overview

This lab analyzes a packet capture showing slow application response. The goal was to distinguish between network delay and application delay, and understand how TCP keeps a connection alive while waiting for a slow server to respond.

---

## Key Concepts from This Lab

### TCP Options to Look For

Always check the first three TCP options at the start of a capture:

- **Maximum Segment Size:** 1460 bytes
- **Selective Acknowledgment (SACK):** controls how much unacknowledged data is allowed
- **Receive Window:** servers don't need a large receive window, clients do. The receive window controls how much data can be received at once before passing it to the application.

### TTL: Time To Live

TTL is an indicator of how far away a server is, specifically how many hops a packet can travel before it dies. Common starting TTL values are 64, 128, and 255. A TTL of 111 most likely started at 128, meaning the server is approximately 17 hops away.

---

## What the Packet Capture Showed

- **Packet 6:** GET request completed successfully
- **Packet 7:** Client waits 45 seconds confirming it is alive. The TCP connection is keeping itself active while the slow application does its work.
- **Packet 8:** Server responds in .079 seconds confirming it is alive. The TCP connection is keeping itself active while the slow application does its work.
- **Packet 9:** 43 seconds later
- **Packet 10:** Server responds in .095 seconds. Not a network problem.
- **Packet 11:** 18 seconds later, server finally returns actual payload HTTP/1.1 200 OK

---

## Key Finding

The delay between Packet 4 and Packet 11 is 108 seconds. That delay is on the application, not the network. The network responded in milliseconds. 100 milliseconds is not a major problem. The application was simply slow to fulfill the request.

---

## How to Assign Blame Correctly

To determine whether the problem is the network or the server, follow these steps:

1. Set filters for TCP connections
2. Watch the handshake establish, time it and note how long it takes
3. Watch the client send the request to the server and time how long the server takes to respond

---

## What Was the Server Doing?

Unknown from this capture alone. The next step would be to look at the server side and obtain another packet capture from that end. The delay could be a problem with the server itself.

---

## Key Takeaway

The client tried to keep TCP alive. It was not a client delay, and not a network delay since the network responded in .095 seconds. The 108 second gap between packets 4 and 11 is an application delay. TCP kept the connection alive the entire time waiting for the slow application to respond.

---

## Source

Packet capture file provided by the Wireshark101 course by Pearson.
