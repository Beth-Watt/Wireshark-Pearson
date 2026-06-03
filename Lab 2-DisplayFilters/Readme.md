# Lab 2 — Display Filters

**File:** `Lab 2-DisplayFilters.pcapng`  
**Packets:** 1,550

This lab was all about building display filters — how to write them, how to combine them, and how to use them to cut through noise and find what actually matters.

---

## Searching for Slow TCP Conversations

`tcp.time_delta > 10` — only 2 packets came back. One was an FTP request and one was a TCP Keep-Alive, both with gaps over 10 seconds. Good filter for hunting slow conversations fast.

---

## Finding Only TCP SYNs

`tcp.flags.syn == 1` returns 428 packets but that includes SYN/ACKs. To get only the pure SYNs with no ACK — `tcp.flags.syn == 1 && tcp.flags.ack == 0` — that brought it down to 365. Small difference in the filter, very different result.

---

## Membership Operator

Instead of chaining a bunch of `or` statements for ports, you can use `tcp.port in {21,80,443,445}` — 245 packets. Way cleaner to read and write. One thing to watch: the range syntax uses two dots not three. Typo'd it the first time and the filter bar turned red.

---

## Contains vs Matches

`frame contains "txt"` is case sensitive and searches raw bytes — found a TFTP file called `rfc1350.txt` plus some TCP packets.

`frame matches "mail"` is case insensitive and uses regex — pulled back 25 packets across POP, DNS, TFTP and BROWSER traffic. Useful when you're not sure exactly how something is spelled or cased in the capture.

---

## Filtering HTTP GETs by Version

`http.request.method == "GET" && http.request.version == "HTTP/1.1"` — 10 packets. You can get pretty specific combining field filters like this.

---

## TLS Handshakes

`tls.handshake.type in {1,2}` — 4 packets, Client Hello and Server Hello across both TLSv1.2 and TLSv1.3. Same membership operator works across different field types, not just ports.

---

## Removing Noise

This is where it got interesting. Instead of filtering for something, you filter out everything you don't care about.

`!(arp or dns or (tcp.port in {21,23,443}))` stripped out the clutter and left mostly HTTP traffic.

Pushing it further — `!(arp or dns or smtp or tftp or udp or http or (tcp.port in {21,23,443}))` — left only 2 packets: a DCERPC request and a TCP retransmission. That retransmission would have been buried in 1,550 packets without this approach.

`not dns && not tcp.flags.syn == 1` — 1,120 packets left. Knocking out DNS and all SYN packets together clears a lot of background noise at once.

---

## Key Takeaways

The main difference between capture filters and display filters is that display filters capture all packets first and then let you filter through them after the fact. Capture filters only grab the traffic that matches the filter — so if you didn't set it right, that traffic is just gone. I prefer capturing everything and using display filters to search through it. You can't search for what you don't have.

Conversation filters are more effective than following a TCP stream because following a stream only shows you unencrypted plaintext information. Conversation filters give you the full picture.

Time based filters like `tcp.time_delta > 10` are a quick way to surface slowness — if something took longer than it should have, that gap shows up immediately and gives you somewhere to start.

When it comes to removing noise, the operators `!` (not), `||` (or), and `&&` (and) do the heavy lifting. The approach that clicked for me — build the filter around what you want to remove, wrap it in a `!`, and what's left is exactly what you need. Something like `!(arp or dns or (tcp.port in {21,23,443}))` — negate the noise, keep the signal.
