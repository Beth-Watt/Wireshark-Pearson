# Lab 3 – TCP Retransmissions & Packet Loss

Top 5 Network Problems series — capture file: `Lab 3-TCP Retrans.pcapng`

---

## The point of this lab

Figure out how to spot TCP retransmissions in Wireshark and calculate how many packets actually went missing. TCP will retransmit every single byte until it gets acknowledged — this lab shows what that looks like in a capture.

---

## TCP stuff to know going in

The handshake (SYN, SYN-ACK, ACK) is where sequence numbers get set up. Everything else builds on that.

- **Sequence numbers** — how TCP tracks what's been sent and received
- **MSS (Max Segment Size)** — max payload size, negotiated during the handshake. In this capture it's 1460 bytes.
- **Window Scale** — multiplies the receive window, matters more on high-latency links
- **SACK (Selective ACK)** — receiver tells the sender exactly which segments arrived so only the missing ones get resent, not everything

The ACK number is always the other side's seq number +1. That's TCP saying "got everything up to X, send me X+1." When there's a gap in seq numbers, that's how TCP knows something went missing.

---

## First thing to do in Wireshark — add these columns

Add relative Sequence Number and Acknowledgment Number columns before doing anything else. Makes the math way easier to follow.

`Edit → Preferences → Columns`

![Adding relative sequence and ACK number columns in Wireshark](Lab_3-AddColumnsRelativeSeq_Ack.png)

---

## Finding the gap — 9 missing packets

Packets 52–54 are where it gets interesting:

| Packet | Seq # | What's happening |
|--------|-------|------|
| 52 | 45261 | last normal packet |
| 53 | 46721 | ACK from receiver |
| 54 | 59861 | **[TCP Previous segment not captured]** — gap |
| 55 | 46721 | **[TCP Dup ACK 53#1]** — receiver asking for what it missed |

Seq jumped from 46721 to 59861. To find out how many packets that gap represents:

```
59861 - 46721 = 13140 bytes missing
13140 ÷ 1460  = 9 packets missing/dropped
```

9 packets dropped between packet 52 and 54.

![Packets 52-55 showing the gap and TCP Previous segment not captured](Lab_3-TCP_Retrans-PacketNotCaptured.png)

![Zoomed view of the 9 missing packets gap](Lab_3-9MissingPackets.png)

---

## What happened after the gap

- **Packet 54** — `[TCP Previous segment not captured]` — Wireshark never saw what should have come before this
- **Packets 55, 57, 59** — Dup ACKs (`53#1`, `#2`, `#3`) — receiver keeps asking for the same thing
- **Packet 58** — `[TCP Window Full]` — receiver's buffer filled up waiting
- **Packet 60** — `[TCP Fast Retransmission]` — 3 dup ACKs triggered the retransmit without waiting for a timeout

Packet 60 is the first retransmission. 9 packets total got resent. TCP doesn't give up.

---

## Set/Unset Time Reference

Right-click any packet → `Set/Unset Time Reference` (or `Ctrl+T`). That packet shows as `*REF*` and all the times after it reset relative to it.

Set the reference on packet 53, then scroll to packet 79 — **191ms** spent on retransmissions. That's 191ms your application was sitting and waiting.

![Set/Unset time reference on packet 53 showing retransmission window](Lab_3-Set_UnsetTimeRef.png)

---

## Why packet loss actually matters

It kills application performance. Not "slows it down a little" — can completely bog it down. When TCP is retransmitting, the app is stalled waiting for data that should already be there.

What to look for in Wireshark: dup ACKs, retransmissions, out-of-order segments. Black rows and red text = start digging.

When you find it, walk the network path. Check every switch and router for:
- FCS/CRC errors (Ethernet layer — FCS is the field in the frame, CRC is the algorithm that calculates it; both show up in interface counters on switches/routers and usually point to physical layer issues like a bad cable, bad port, or duplex mismatch)
- Discards — input *and* output, either side can be the problem

---

## Files

- `Lab 3-TCP Retrans.pcapng` — the capture
- `Lab_3-AddColumnsRelativeSeq_Ack.png`
- `Lab_3-TCP_Retrans-PacketNotCaptured.png`
- `Lab_3-9MissingPackets.png`
- `Lab_3-Set_UnsetTimeRef.png`
