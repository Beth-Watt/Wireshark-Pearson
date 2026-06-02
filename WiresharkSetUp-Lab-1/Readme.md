# Wireshark101 — Learning to Read Networks

A personal lab series working through real packet captures to get comfortable with Wireshark. Nothing fancy — just hands-on time with actual traffic, building habits that make troubleshooting faster.

---

## What This Is

I put this together to document my process as I work through network analysis from the ground up. The goal isn't to cover every feature Wireshark has — it's to build a workflow that's actually useful when something is broken and you need answers fast.

The labs use real `.pcapng` files, and the notes reflect what I actually ran into, not a cleaned-up version of it.

---

## Setup — Before the Labs

Before jumping into packet analysis, there's some one-time Wireshark configuration worth doing. It makes a big difference in how readable everything is.

### Custom Profile

I created a dedicated **Wireshark101** profile rather than touching the Default one. Keeps everything clean and isolated.

### Columns

Adding a **Delta** column was the first thing that made packet analysis click for me. Seeing the time between packets at a glance is way more useful than staring at absolute timestamps trying to do mental math.

### Coloring Rules

This one I didn't expect to matter as much as it does. Being able to scan through hundreds of packets and immediately spot what's worth looking at — just from color — saves a lot of time. Without it you're reading every single Info field trying to find something wrong.

The rules I have active:

- **Bad TCP** — `tcp.analysis.flags && !tcp.analysis.window_update && !tcp.analysis.keep_alive`
- **TCP SYNs** — `tcp.flags.syn == 1`
- **TCP RST** — `tcp.flags.reset eq 1`
- **HTTP** — `http || tcp.port == 80 || http2`

### Filter Buttons

Added a **Bad TCP** button to the filter bar so I'm not retyping that filter constantly. Label is `Bad TCP`, filter is `tcp.analysis.flags`.

---

## Lab 1 — Slow Network

**File:** `Pre-Lab-SlowNetwork.pcapng`  
**Traffic:** HTTP between `172.16.0.13` and `10.0.0.100`  
**Packets:** 123

### What I was looking for

The capture name says "slow network," so the question is: where's the delay, and what's causing it?

### What the delta column showed

Packet 7 jumped out immediately. The client sent the HTTP GET in packet 4 at 0.001087s and didn't get the HTTP 200 OK back until packet 7 — a **20.586026 second** wait. Everything before and after that gap was normal, fast ACKs, clean TCP handshake, no retransmissions.

### Conclusion

This isn't a network problem. The packets got there fine. The server just took over 20 seconds to process the request and send a response. That's a slow application, not a slow network — and that distinction matters a lot when you're trying to figure out what to fix. If you misread it as a network problem you could spend weeks looking in the wrong place.
