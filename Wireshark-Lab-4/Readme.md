# Lab 4 – Network Congestion

## Overview

This lab analyzes a packet capture showing network congestion using the file `NetworkCongestion.pcapng`. The goal was to identify the signs of congestion and understand how TCP responds when the network can't keep up.

- [Lab 4-NetworkCongestion.png](Lab%204-NetworkCongestion.png)

---

## What Network Congestion Looks Like in Wireshark

Black and red rows in the packet list are the first indicator that something is wrong. These colors flag retransmissions and duplicate ACKs — both signs that packets are being lost in both directions.

When congestion occurs, network devices can't process data fast enough, which causes packet loss. TCP detects the loss and responds by slowing down transmission and waiting for congestion to clear before retransmitting. This creates a cycle of delayed transmissions that compounds the problem.

---

## Key Indicators

- **TCP Retransmissions** — packets sent again because no acknowledgment was received
- **TCP Duplicate ACKs** — the receiving end signaling it already received that data
- **Packets lost in both directions** — confirms the issue is network-wide, not one-sided

---

## How to Investigate Further

Use the Statistics menu and navigate to **Conversations** to get a broader view. Look for top talkers, review IP conversations, and examine TCP sessions. Ask:

- Is a backup happening in the middle of the day?
- Is one server talking to another at an unusual time?

Unexpected traffic patterns often explain congestion.

---

## Key Takeaway

Congestion causes network devices to fall behind, leading to packet loss, which forces TCP to keep retransmitting, which adds more traffic, which makes congestion worse. Identifying the source of unusual traffic is the first step to resolving it.
