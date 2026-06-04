# Lab 6 – TCP Resets

---

## Scenario

A user opened a browser to go out to the internet and started experiencing bad, intermittent connectivity — not persistent, just random. First few tabs worked fine, then all of a sudden tabs stopped working, even though a few continued to work. The pcap was captured while troubleshooting this random intermittent internet connectivity problem.

---

## What Looked Suspicious Right Away

The first thing that stood out was **53.235.100.201 → 192.168.1.1** — 277 packets, 139 A→B, about 71KB, all over port 443. There were also 12 two-packet conversations between 192.168.1.1 and 40.147.222.88, and an Ethernet stream with 868 packets at Stream ID 0, meaning it started at the very beginning of the capture.

- **53.235.100.201** had packet size limited during capture and red bars → RST, ACK (0 segment length)
- A lot of 2-packet conversations on port 443 — that's not normal

---

## Digging Into the Conversations

After setting a display filter for `ip.addr == 53.235.100.201`, the conversations view only pulled up the ones involving that IP — which is exactly what you want.

192.168.1.1 is making repeated connection attempts to 53.235.100.201 on port 443 using multiple different source ports (49979, 49983, 49986, 49987, 49996, 50001). Most of those conversations are only 2 packets. The pattern: SYN goes out → RST comes back → connection dies → tries again on a new port. The top row with 277 packets was actually a successful connection — everything after that failed with just 2 packets.

**Coloring rules in play:**
- Light green = new connection attempt at a new port
- Red row = RST,ACK — connection immediately rejected, connection was refused

So either the whole internet decided to reset new connections from this user (not likely), or some device along the path is sending resets on behalf of the device trying to connect.

---

## Finding the Source of the Resets

**Filter:** `tcp.flags.reset==1` → filtered down to 107 packets (11.9% of 900 total)

Pulled up Packet 19 and checked two things:

- **Under IP:** TTL = 64 → the reset is coming from an unrouted source, not traveling across the internet to get here
- **Under Ethernet II:** Source = **Sonicwall_9d:c4:d0**

The firewall is originating the reset.

---

## Root Cause — Firewall Misconfiguration

There was a setting that capped the number of TCP connections per user. The reason that setting exists is to prevent the firewall from getting overwhelmed — rather than trying to track stateful connections with thousands of entries, it gives you a ceiling on new connections you can establish as a client.

The Sonicwall was set at **100** connections, which was too low for the traffic volume on this network. The firewall was constantly hitting that cap, sending resets on new connection attempts while keeping existing ones alive — which is exactly why the connectivity was intermittent. Some tabs kept working because they were already established before the cap was hit. New ones kept getting reset.

**Fix:** Firewall connection limit bumped to **500** — problem went away.

---

## Key Takeaways

TCP shuts down connections two ways:
1. **FIN** — after a successful conversation, clean close
2. **Reset** — abrupt, something killed it

When you see RST,ACK with Seq=1, Ack=1, Win=0, Len=0 coming back immediately after a SYN, that's not the remote server rejecting you — that's something in the middle doing it. The TTL and the Ethernet source MAC are your clues for figuring out what in the middle is responsible.

---

## Tools / Filters Used

- Isolate IP traffic: `ip.addr == 53.235.100.201`
- Find all resets: `tcp.flags.reset==1`
- Identify reset source: Packet details → Ethernet II → Source MAC
- Confirm local origin: IP layer → TTL = 64
- Conversation overview: Statistics → Conversations → TCP tab

---

## Skills Demonstrated

Filtered by IP to focus on the traffic that actually matters. Identified RST patterns across multiple source ports and traced where the reset was coming from using TTL and MAC address. Connected firewall connection limits to an intermittent connectivity problem and used Wireshark coloring rules to spot it faster.

---

*Wireshark101 — Lab 6: TCP Resets | Lab 6-TCPResets.pcapng*
