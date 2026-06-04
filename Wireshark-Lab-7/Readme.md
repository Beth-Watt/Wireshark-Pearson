# Lab 7: TCP Performance Problems

## What This Lab Is About

TCP performance problems are tricky because when something feels slow, it's easy to blame the app, the network, or the client — but it's not necessarily any of those. It really comes down to how TCP is passing and consuming data.

Capture file: `Lab_7-TCPIssues.pcapng`

---

## Concepts to Know Going In

**TCP Window Size (`Win=`)** — how much space the receiver currently has in its buffer. The sender can't send more than this. Add it as a column in Wireshark so you can watch it change:
`TCP Header → Window → Add as Column`

**[TCP Window Full]** — the sender has maxed out the receiver's buffer. Nothing else can go through until space opens up.

**[TCP Zero Window]** — the receiver's buffer is completely full. Transmission stops. The sender has to wait for a Window Update before it can send anything again.

---

## What I Did

### Scroll through and scan for problems
Look for black lines with red text — that's Wireshark flagging something. The Info column will show `[TCP Window Full]` or `[TCP Zero Window]` where things go wrong.

### Packet 47
Server (`157.240.11.22`) → Client (`10.2.15`) — `[TCP Window Full]`, 11680 bytes. Client's buffer is full, server can't send more.

### Packet 49
Same thing again — server → client, `[TCP Window Full]` on port 443 → 58322.

### Packet 50
Client → Server, Window = `0` — `[TCP Zero Window]`. Buffer is completely exhausted, transmission halts.

### Packet 51
`[TCP Window Update]` — client tells the server it has space again, transmission resumes.

### Packet reassembly note
When you're capturing internally (on the client or server side), Wireshark may show packet reassembly at the TCP layer. That's normal — it's just TCP combining segments before handing them up to the application.

---

## What I Found

- **Packet 47** — TCP Window Full (server → client) — client buffer saturated
- **Packet 49** — TCP Window Full (server → client) — still saturated
- **Packet 50** — TCP Zero Window (client → server) — transmission halted
- **Packet 51** — TCP Window Update (client → server) — buffer cleared, back to sending

The whole conversation was only about 1.3 seconds. The client was just getting congested at the transport layer — not a network issue, not the app's fault. The client couldn't keep up with the data coming in.

---

## Things to Remember

- **TCP Window Full** = sender hit the limit, waiting on the receiver
- **TCP Zero Window** = receiver is maxed out, everything stops
- **TCP Window Update** = receiver cleared space, sender can go again
- Always add the Window Size column — you'll miss a lot without it
- A short conversation with these events = congestion on the client side, not the path

---

*Source: Wireshark 101 — Pearson Course*
