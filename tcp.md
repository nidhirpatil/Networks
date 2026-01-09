# TCP (Transmission Control Protocol): A Comprehensive Guide

## Table of Contents
1. [Introduction](#introduction)
2. [TCP Characteristics](#tcp-characteristics)
3. [TCP Header Structure](#tcp-header-structure)
4. [TCP Connection Establishment](#tcp-connection-establishment)
5. [TCP Connection Termination](#tcp-connection-termination)
6. [TCP Reliability Mechanisms](#tcp-reliability-mechanisms)
7. [TCP Flow Control](#tcp-flow-control)
8. [TCP Congestion Control](#tcp-congestion-control)
9. [TCP Ports and Sockets](#tcp-ports-and-sockets)
10. [TCP vs UDP](#tcp-vs-udp)
11. [TCP States](#tcp-states)
12. [Common TCP Issues](#common-tcp-issues)
13. [Best Practices](#best-practices)

---

## Introduction

### What is TCP?

**TCP (Transmission Control Protocol)** is a connection-oriented, reliable transport-layer protocol that provides ordered, error-checked delivery of data between applications running on hosts communicating over an IP network. TCP is one of the core protocols of the Internet Protocol Suite.

![TCP Overview](images/tcp-overview.png)
*Figure 1: TCP provides reliable data delivery between applications*

### TCP in the Network Stack

TCP operates at the **transport layer** (Layer 4) of the OSI model:

```
┌─────────────────────────────────────┐
│   Application Layer (HTTP, FTP)    │  Layer 7
├─────────────────────────────────────┤
│   Presentation Layer                │  Layer 6
├─────────────────────────────────────┤
│   Session Layer                     │  Layer 5
├─────────────────────────────────────┤
│   Transport Layer (TCP, UDP)        │  Layer 4 ← TCP
├─────────────────────────────────────┤
│   Network Layer (IP)                │  Layer 3
├─────────────────────────────────────┤
│   Data Link Layer (Ethernet)        │  Layer 2
├─────────────────────────────────────┤
│   Physical Layer                    │  Layer 1
└─────────────────────────────────────┘
```

*Figure 2: TCP in the OSI model*

### TCP History

- **1974**: TCP designed by Vint Cerf and Bob Kahn
- **1981**: RFC 793 published (TCP specification)
- **1980s**: TCP becomes standard for reliable internet communication
- **1990s**: TCP congestion control algorithms developed
- **2000s**: TCP improvements (SACK, window scaling)
- **2010s**: TCP Fast Open, Multipath TCP

### Why TCP Exists

TCP solves problems that IP alone cannot handle:

1. **Reliability**: IP is unreliable - packets can be lost, duplicated, or arrive out of order
2. **Ordering**: IP doesn't guarantee packet order
3. **Flow Control**: Prevents overwhelming the receiver
4. **Congestion Control**: Prevents network congestion
5. **Connection Management**: Establishes and maintains connections

---

## TCP Characteristics

### Key Features

#### 1. Connection-Oriented
- Establishes connection before data transfer
- Maintains connection state
- Terminates connection when done

#### 2. Reliable Delivery
- Guarantees data delivery
- Detects and retransmits lost packets
- Ensures data integrity

#### 3. Ordered Delivery
- Delivers data in order
- Reorders out-of-order packets
- Maintains sequence numbers

#### 4. Flow Control
- Prevents sender from overwhelming receiver
- Uses sliding window mechanism
- Receiver controls data rate

#### 5. Congestion Control
- Prevents network congestion
- Adapts to network conditions
- Multiple algorithms (Reno, CUBIC, BBR)

#### 6. Full-Duplex Communication
- Simultaneous bidirectional communication
- Both sides can send and receive
- Independent data streams

#### 7. Byte-Stream Service
- Treats data as continuous byte stream
- No message boundaries
- Application must handle framing

![TCP Characteristics](images/tcp-characteristics.png)
*Figure 3: Key characteristics of TCP*

### TCP vs Other Protocols

| Feature | TCP | UDP | IP |
|---------|-----|-----|-----|
| **Reliability** | Yes | No | No |
| **Ordering** | Yes | No | No |
| **Connection** | Connection-oriented | Connectionless | Connectionless |
| **Flow Control** | Yes | No | No |
| **Congestion Control** | Yes | No | No |
| **Overhead** | High | Low | Low |
| **Speed** | Slower | Faster | Fastest |

---

## TCP Header Structure

### TCP Header Format

The TCP header is 20 bytes (minimum) and can be up to 60 bytes with options:

```
 0                   1                   2                   3
 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|          Source Port          |       Destination Port        |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|                        Sequence Number                        |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|                    Acknowledgment Number                      |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|  Data |           |U|A|P|R|S|F|                               |
| Offset| Reserved  |R|C|S|S|Y|I|            Window             |
|       |           |G|K|H|T|N|N|                               |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|           Checksum            |         Urgent Pointer        |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|                    Options                    |    Padding    |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
```

*Figure 4: TCP header structure*

### TCP Header Fields

#### Source Port (16 bits)
- Port number of the sender
- Range: 0-65535
- Ephemeral ports: 49152-65535 (client-side)

#### Destination Port (16 bits)
- Port number of the receiver
- Well-known ports: 0-1023 (HTTP: 80, HTTPS: 443, SSH: 22)

#### Sequence Number (32 bits)
- Sequence number of first byte in segment
- Used for ordering and reliability
- Random initial value (ISN - Initial Sequence Number)

#### Acknowledgment Number (32 bits)
- Next expected sequence number
- Only valid when ACK flag is set
- Confirms receipt of data

#### Data Offset (4 bits)
- Length of TCP header in 32-bit words
- Minimum: 5 (20 bytes)
- Maximum: 15 (60 bytes)

#### Reserved (6 bits)
- Reserved for future use
- Must be zero

#### Control Flags (6 bits)
- **URG (Urgent)**: Urgent pointer is valid
- **ACK (Acknowledgment)**: Acknowledgment number is valid
- **PSH (Push)**: Push buffered data immediately
- **RST (Reset)**: Reset the connection
- **SYN (Synchronize)**: Synchronize sequence numbers
- **FIN (Finish)**: No more data from sender

#### Window Size (16 bits)
- Size of receive window
- Flow control mechanism
- Maximum: 65535 bytes (can be scaled)

#### Checksum (16 bits)
- Error detection for header and data
- Includes pseudo-header (IP addresses, protocol, length)

#### Urgent Pointer (16 bits)
- Points to urgent data
- Only valid when URG flag is set

#### Options (Variable length)
- Maximum segment size (MSS)
- Window scaling
- Timestamp
- SACK (Selective Acknowledgment)
- Padding to align to 32-bit boundary

![TCP Header Fields](images/tcp-header-fields.png)
*Figure 5: Detailed TCP header fields*

### TCP Options

#### Maximum Segment Size (MSS)
- Largest segment size sender can receive
- Negotiated during connection establishment
- Default: 536 bytes (IPv4) or 1220 bytes (IPv6)

#### Window Scaling
- Extends window size beyond 65535 bytes
- Multiplier: 2^scale_factor
- Allows larger windows for high-speed networks

#### Selective Acknowledgment (SACK)
- Acknowledges non-contiguous data blocks
- Improves performance with packet loss
- Allows faster recovery

#### Timestamp
- Used for:
  - Round-trip time measurement
  - Protection against wrapped sequence numbers (PAWS)

---

## TCP Connection Establishment

### Three-Way Handshake

TCP establishes a connection using a three-way handshake:

```
Client                                    Server
  │                                         │
  │─── SYN (seq=x) ────────────────────────>│
  │                                         │
  │<── SYN-ACK (seq=y, ack=x+1) ───────────│
  │                                         │
  │─── ACK (seq=x+1, ack=y+1) ────────────>│
  │                                         │
  │<════ Connection Established ═══════════>│
```

*Figure 6: TCP three-way handshake*

### Detailed Handshake Steps

#### Step 1: Client sends SYN
- Client sends SYN packet
- Sets initial sequence number (ISN)
- Random value for security
- Example: `SYN, seq=1000`

#### Step 2: Server sends SYN-ACK
- Server receives SYN
- Allocates resources
- Sends SYN-ACK:
  - Sets its own ISN
  - Acknowledges client's SYN (ack=client_seq+1)
- Example: `SYN-ACK, seq=5000, ack=1001`

#### Step 3: Client sends ACK
- Client receives SYN-ACK
- Sends ACK to confirm
- Connection established
- Example: `ACK, seq=1001, ack=5001`

### Initial Sequence Number (ISN)

- Random value to prevent:
  - Sequence number prediction attacks
  - Connection hijacking
- Modern implementations use:
  - Cryptographically secure random number generator
  - Clock-based component
  - Incremental counter

### Connection States

During handshake, connection goes through states:

**Client States:**
1. CLOSED → SYN_SENT → ESTABLISHED

**Server States:**
1. CLOSED → LISTEN → SYN_RECEIVED → ESTABLISHED

![TCP Handshake States](images/tcp-handshake-states.png)
*Figure 7: State transitions during three-way handshake*

### SYN Flood Attack

**Problem:** Attacker sends many SYN packets without completing handshake

**Impact:** Server resources exhausted, legitimate connections denied

**Mitigation:**
- SYN cookies
- Rate limiting
- Firewall rules
- Connection timeouts

---

## TCP Connection Termination

### Four-Way Handshake

TCP terminates connection using a four-way handshake:

```
Client                                    Server
  │                                         │
  │─── FIN (seq=x) ────────────────────────>│
  │                                         │
  │<── ACK (ack=x+1) ──────────────────────│
  │                                         │
  │<── FIN (seq=y) ────────────────────────│
  │                                         │
  │─── ACK (ack=y+1) ──────────────────────>│
  │                                         │
  │<════ Connection Closed ═══════════════>│
```

*Figure 8: TCP four-way handshake*

### Detailed Termination Steps

#### Step 1: Client sends FIN
- Client wants to close connection
- Sends FIN packet
- Enters FIN_WAIT_1 state
- Example: `FIN, seq=5000`

#### Step 2: Server sends ACK
- Server receives FIN
- Sends ACK immediately
- Enters CLOSE_WAIT state
- Can still send data
- Example: `ACK, ack=5001`

#### Step 3: Server sends FIN
- Server finished sending data
- Sends FIN packet
- Enters LAST_ACK state
- Example: `FIN, seq=8000`

#### Step 4: Client sends ACK
- Client receives FIN
- Sends ACK
- Enters TIME_WAIT state
- Waits 2MSL (Maximum Segment Lifetime)
- Then enters CLOSED state
- Example: `ACK, ack=8001`

### TIME_WAIT State

**Purpose:**
- Ensures final ACK is received
- Prevents old segments from interfering with new connections
- Allows duplicate segments to expire

**Duration:** 2 × MSL (Maximum Segment Lifetime)
- Typically: 30 seconds to 2 minutes
- MSL: Maximum time a segment can exist in network

**Why Important:**
- If final ACK is lost, server retransmits FIN
- Client must be able to respond
- Prevents port reuse issues

### Simultaneous Close

Both sides can initiate close simultaneously:

```
Client                                    Server
  │                                         │
  │─── FIN (seq=x) ────────────────────────>│
  │<── FIN (seq=y) ────────────────────────│
  │                                         │
  │─── ACK (ack=y+1) ─────────────────────>│
  │<── ACK (ack=x+1) ──────────────────────│
  │                                         │
  │<════ Connection Closed ═══════════════>│
```

*Figure 9: Simultaneous close*

---

## TCP Reliability Mechanisms

### How TCP Ensures Reliability

TCP uses several mechanisms to ensure reliable data delivery:

#### 1. Sequence Numbers
- Each byte has a sequence number
- Tracks data sent and received
- Enables ordering and duplicate detection

#### 2. Acknowledgments
- Receiver acknowledges received data
- Cumulative ACK: acknowledges all bytes up to N
- Selective ACK (SACK): acknowledges non-contiguous blocks

#### 3. Retransmission
- Sender retransmits unacknowledged data
- Triggered by:
  - Timeout (RTO - Retransmission Timeout)
  - Duplicate ACKs (Fast Retransmit)

#### 4. Checksum
- Detects corrupted data
- Calculated over header and data
- Discards corrupted segments

#### 5. Duplicate Detection
- Sequence numbers identify duplicates
- Discards duplicate segments
- Prevents duplicate data delivery

### Sequence Number Example

```
Sender sends:
Segment 1: seq=1000, len=1000 bytes (covers 1000-1999)
Segment 2: seq=2000, len=1000 bytes (covers 2000-2999)
Segment 3: seq=3000, len=1000 bytes (covers 3000-3999)

Receiver acknowledges:
ACK 2000: Received bytes 1000-1999
ACK 3000: Received bytes 2000-2999
ACK 4000: Received bytes 3000-3999
```

*Figure 10: Sequence numbers and acknowledgments*

### Retransmission Mechanisms

#### Retransmission Timeout (RTO)

**How it works:**
1. Sender transmits segment
2. Starts retransmission timer
3. If ACK received before timeout: Cancel timer
4. If timeout: Retransmit segment

**RTO Calculation:**
- Based on RTT (Round-Trip Time)
- Adaptive algorithm
- Exponential backoff on retransmission

**RTT Measurement:**
- Sample RTT: Time for segment + ACK
- Smoothed RTT: Weighted average
- RTT variance: Deviation from mean

#### Fast Retransmit

**Trigger:** Three duplicate ACKs

**How it works:**
1. Receiver receives out-of-order segment
2. Sends duplicate ACK for expected sequence
3. Sender receives 3 duplicate ACKs
4. Immediately retransmits missing segment
5. Doesn't wait for timeout

```
Sender sends: 1, 2, 3, 4, 5
Receiver receives: 1, 2, 4, 5 (3 is lost)
Receiver sends: ACK 3, ACK 3, ACK 3
Sender: Fast retransmit segment 3
```

*Figure 11: Fast retransmit mechanism*

#### Selective Acknowledgment (SACK)

**Problem:** Cumulative ACK only acknowledges contiguous data

**Solution:** SACK acknowledges non-contiguous blocks

**Example:**
```
Received: segments 1, 2, 4, 5, 7
Cumulative ACK: ACK 3 (only acknowledges 1-2)
SACK: ACK 3, SACK blocks: [4-5], [7-7]
```

![TCP Reliability](images/tcp-reliability.png)
*Figure 12: TCP reliability mechanisms*

---

## TCP Flow Control

### What is Flow Control?

Flow control prevents the sender from overwhelming the receiver. The receiver controls how much data the sender can transmit.

### Receive Window

The receiver advertises its **receive window** (rwnd) size:

- **Window Size**: Amount of buffer space available
- **Advertised in**: TCP header Window field
- **Dynamic**: Changes as buffer fills/empties

### Sliding Window Protocol

TCP uses a sliding window for flow control:

```
Sender Window:
┌─────────────────────────────────────────┐
│ Sent & ACKed │ Sent, Not ACKed │ Window │
│              │                 │        │
└─────────────────────────────────────────┘
              ▲                 ▲
              │                 │
         Window Start      Window End
```

*Figure 13: TCP sliding window*

### Window Size Calculation

```
Available Window = Advertised Window - (Last Byte Sent - Last Byte ACKed)
```

**Example:**
- Advertised window: 10,000 bytes
- Last byte sent: 5,000
- Last byte ACKed: 2,000
- Available window: 10,000 - (5,000 - 2,000) = 7,000 bytes

### Zero Window

**Problem:** Receiver buffer is full

**Solution:** Receiver advertises window size = 0

**Behavior:**
- Sender stops sending data
- Sender probes with zero-window probe
- Waits for window update

### Window Scaling

**Problem:** 16-bit window field limits window to 65,535 bytes

**Solution:** Window scaling option

**How it works:**
- Scale factor: 0-14 (multiplier: 2^scale_factor)
- Effective window: Window × 2^scale_factor
- Example: Window=65535, Scale=4 → Effective=1,048,560 bytes

![TCP Flow Control](images/tcp-flow-control.png)
*Figure 14: TCP flow control mechanism*

---

## TCP Congestion Control

### What is Congestion Control?

Congestion control prevents the sender from overwhelming the network. It adapts transmission rate based on network conditions.

### Congestion Window (cwnd)

- Sender maintains congestion window
- Limits amount of unacknowledged data
- Separate from receive window
- Effective window = min(cwnd, rwnd)

### Congestion Control Phases

#### 1. Slow Start
- Starts with cwnd = 1 MSS
- Doubles cwnd for each ACK received
- Exponential growth
- Continues until:
  - cwnd >= ssthresh (slow start threshold)
  - Packet loss detected

#### 2. Congestion Avoidance
- Linear growth: cwnd += MSS × (MSS/cwnd) per ACK
- More conservative than slow start
- Continues until packet loss

#### 3. Fast Retransmit / Fast Recovery
- Triggered by duplicate ACKs
- Retransmits lost segment
- Reduces cwnd
- Enters congestion avoidance

### Congestion Control Algorithms

#### TCP Reno
- Classic algorithm
- Slow start → Congestion avoidance
- Fast retransmit on 3 duplicate ACKs
- Fast recovery: cwnd = cwnd/2 + 3 MSS

#### TCP NewReno
- Improvement over Reno
- Better handling of multiple losses
- Continues fast recovery until all losses recovered

#### TCP CUBIC
- Default in Linux
- Cubic function for window growth
- Better for high-speed networks
- Less aggressive than Reno

#### TCP BBR (Bottleneck Bandwidth and Round-trip propagation time)
- Google's algorithm
- Estimates bandwidth and RTT
- Maintains queue at optimal level
- Better throughput and latency

### Congestion Indicators

#### Packet Loss
- Indicates network congestion
- Detected by:
  - Timeout
  - Duplicate ACKs

#### Explicit Congestion Notification (ECN)
- Router marks packets when congested
- Receiver echoes ECN to sender
- Sender reduces rate before loss

### Congestion Control Example

```
Time    Event                    cwnd    ssthresh
─────────────────────────────────────────────────
0       Start                    1       64
1       ACK received            2       64
2       ACK received            4       64
3       ACK received            8       64
4       ACK received            16       64
5       ACK received            32       64
6       ACK received            64       64
7       Enter congestion avoid  65       64
8       ACK received            66       64
9       Packet loss detected    33       33
10      Fast retransmit         33       33
```

*Figure 15: Congestion control example*

![TCP Congestion Control](images/tcp-congestion-control.png)
*Figure 16: TCP congestion control phases*

---

## TCP Ports and Sockets

### Ports

A **port** is a 16-bit number (0-65535) that identifies a specific process or service.

#### Port Categories

**Well-Known Ports (0-1023)**
- Reserved for system services
- Examples:
  - 20/21: FTP
  - 22: SSH
  - 23: Telnet
  - 25: SMTP
  - 53: DNS
  - 80: HTTP
  - 443: HTTPS

**Registered Ports (1024-49151)**
- Assigned by IANA
- Used by applications
- Examples:
  - 3306: MySQL
  - 5432: PostgreSQL
  - 8080: HTTP alternate

**Dynamic/Private Ports (49152-65535)**
- Ephemeral ports
- Used by clients
- Assigned dynamically

### Sockets

A **socket** is the combination of:
- IP address (identifies host)
- Port number (identifies process)
- Protocol (TCP/UDP)

**Format:** `IP:Port`
- Example: `192.0.2.1:80`
- IPv6: `[2001:db8::1]:443`

### Socket Pair

A TCP connection is identified by a **socket pair**:
- Client socket: `Client_IP:Client_Port`
- Server socket: `Server_IP:Server_Port`

**Example:**
- Client: `192.0.2.100:49152`
- Server: `192.0.2.1:80`
- Connection: `192.0.2.100:49152 ↔ 192.0.2.1:80`

### Socket States

Sockets go through various states during connection lifecycle:

- **CLOSED**: No connection
- **LISTEN**: Server waiting for connection
- **SYN_SENT**: Client sent SYN
- **SYN_RECEIVED**: Server received SYN
- **ESTABLISHED**: Connection active
- **FIN_WAIT_1**: Client sent FIN
- **FIN_WAIT_2**: Client received ACK for FIN
- **CLOSE_WAIT**: Server received FIN
- **LAST_ACK**: Server sent FIN
- **TIME_WAIT**: Waiting for 2MSL
- **CLOSING**: Simultaneous close

![TCP Socket States](images/tcp-socket-states.png)
*Figure 17: TCP socket state diagram*

---

## TCP vs UDP

### Comparison Table

| Feature | TCP | UDP |
|---------|-----|-----|
| **Connection** | Connection-oriented | Connectionless |
| **Reliability** | Reliable | Unreliable |
| **Ordering** | Ordered delivery | No ordering |
| **Flow Control** | Yes | No |
| **Congestion Control** | Yes | No |
| **Overhead** | High (20+ bytes header) | Low (8 bytes header) |
| **Speed** | Slower | Faster |
| **Use Cases** | HTTP, FTP, Email | DNS, Video streaming, Gaming |
| **Error Detection** | Checksum | Checksum (optional) |
| **Retransmission** | Yes | No |

### When to Use TCP

- **Reliability required**: File transfer, email, web browsing
- **Order matters**: Database replication, file synchronization
- **Flow control needed**: Large data transfers
- **Can tolerate latency**: Not real-time

### When to Use UDP

- **Speed critical**: Real-time gaming, video streaming
- **Loss acceptable**: Live video, audio
- **Low overhead needed**: DNS queries
- **Broadcast/multicast**: Network discovery

### TCP vs UDP Headers

**TCP Header:** 20-60 bytes
```
Source Port | Dest Port | Seq Num | Ack Num | Flags | Window | Checksum | Options
```

**UDP Header:** 8 bytes
```
Source Port | Dest Port | Length | Checksum
```

![TCP vs UDP](images/tcp-vs-udp.png)
*Figure 18: TCP vs UDP comparison*

---

## TCP States

### Complete State Diagram

```
                    CLOSED
                      │
                      ▼
                   LISTEN
                      │
                      │ (SYN received)
                      ▼
              SYN_RECEIVED
                      │
                      │ (ACK sent)
                      ▼
                ESTABLISHED
                      │
        ┌─────────────┼─────────────┐
        │             │             │
        │ (FIN sent)  │ (FIN recv)  │ (FIN recv)
        ▼             ▼             ▼
    FIN_WAIT_1   CLOSE_WAIT    FIN_WAIT_1
        │             │             │
        │ (ACK recv)  │ (FIN sent)  │ (ACK sent)
        ▼             ▼             ▼
    FIN_WAIT_2   LAST_ACK    FIN_WAIT_2
        │             │             │
        │ (FIN recv)  │ (ACK recv)  │ (FIN recv)
        ▼             ▼             ▼
     TIME_WAIT      CLOSED      TIME_WAIT
        │                         │
        │ (2MSL timeout)          │ (2MSL timeout)
        ▼                         ▼
      CLOSED                    CLOSED
```

*Figure 19: Complete TCP state diagram*

### State Descriptions

#### CLOSED
- No connection exists
- Initial and final state

#### LISTEN
- Server waiting for connection
- Passive open

#### SYN_SENT
- Client sent SYN
- Waiting for SYN-ACK

#### SYN_RECEIVED
- Server received SYN
- Sent SYN-ACK
- Waiting for ACK

#### ESTABLISHED
- Connection active
- Data can be exchanged

#### FIN_WAIT_1
- Sent FIN
- Waiting for ACK or FIN

#### FIN_WAIT_2
- Received ACK for FIN
- Waiting for FIN

#### CLOSE_WAIT
- Received FIN
- Application closing
- Will send FIN

#### LAST_ACK
- Sent FIN
- Waiting for final ACK

#### TIME_WAIT
- Received FIN
- Sent ACK
- Waiting 2MSL

#### CLOSING
- Both sides closing simultaneously
- Waiting for ACK

---

## Common TCP Issues

### 1. Connection Timeout

**Symptoms:**
- Connections fail to establish
- Slow connection setup

**Causes:**
- Network congestion
- Firewall blocking
- Server overload
- SYN flood attack

**Solutions:**
- Check network connectivity
- Review firewall rules
- Increase server capacity
- Implement SYN cookies

### 2. High Latency

**Symptoms:**
- Slow data transfer
- High RTT

**Causes:**
- Network congestion
- Long distance
- Slow network links
- Buffer bloat

**Solutions:**
- Optimize routing
- Use CDN
- Implement BBR algorithm
- Tune buffer sizes

### 3. Packet Loss

**Symptoms:**
- Retransmissions
- Slow throughput
- Connection drops

**Causes:**
- Network congestion
- Link errors
- Buffer overflow
- MTU issues

**Solutions:**
- Reduce congestion
- Fix network links
- Increase buffer sizes
- Adjust MTU

### 4. Connection Reset (RST)

**Symptoms:**
- Connections abruptly closed
- "Connection reset by peer" errors

**Causes:**
- Application error
- Firewall rules
- Idle timeout
- Port scan detection

**Solutions:**
- Check application logs
- Review firewall configuration
- Adjust timeout values
- Verify security policies

### 5. TIME_WAIT Accumulation

**Symptoms:**
- Port exhaustion
- Cannot bind to port
- "Address already in use" errors

**Causes:**
- Many short-lived connections
- TIME_WAIT state (2MSL wait)

**Solutions:**
- Connection pooling
- Reuse sockets (SO_REUSEADDR)
- Increase port range
- Reduce TIME_WAIT duration (carefully)

### 6. Window Size Issues

**Symptoms:**
- Slow transfer despite bandwidth
- Zero window advertisements

**Causes:**
- Small receive buffer
- Slow application processing
- Network issues

**Solutions:**
- Increase receive buffer (SO_RCVBUF)
- Optimize application
- Check network conditions
- Enable window scaling

### Troubleshooting Commands

#### netstat
```bash
# Show TCP connections
netstat -an | grep tcp

# Show listening ports
netstat -tuln

# Show connection statistics
netstat -s
```

#### ss (Socket Statistics)
```bash
# Show TCP connections
ss -t

# Show listening ports
ss -tln

# Show connection statistics
ss -s
```

#### tcpdump
```bash
# Capture TCP packets
tcpdump -i eth0 tcp

# Capture specific port
tcpdump -i eth0 port 80

# Capture with details
tcpdump -i eth0 -n -v tcp
```

#### Wireshark
- GUI tool for packet analysis
- Filter: `tcp`
- Analyze handshakes, retransmissions, flow control

---

## Best Practices

### For Application Developers

#### 1. Use Connection Pooling
- Reuse connections
- Reduce overhead
- Avoid TIME_WAIT accumulation

#### 2. Set Appropriate Timeouts
- Connection timeout
- Read/write timeout
- Idle timeout

#### 3. Handle Errors Gracefully
- Retry logic
- Exponential backoff
- Circuit breaker pattern

#### 4. Optimize Buffer Sizes
- Match application needs
- Consider network conditions
- Use window scaling

#### 5. Monitor Connection Health
- Track connection metrics
- Monitor retransmissions
- Alert on anomalies

### For Network Administrators

#### 1. Tune TCP Parameters
- Buffer sizes
- TIME_WAIT duration
- Keepalive settings

#### 2. Monitor Network Health
- Packet loss rates
- RTT measurements
- Congestion indicators

#### 3. Implement Quality of Service (QoS)
- Prioritize important traffic
- Limit bandwidth per connection
- Shape traffic

#### 4. Configure Firewalls Properly
- Allow established connections
- Rate limit new connections
- Monitor for attacks

#### 5. Use Appropriate Congestion Control
- Choose algorithm for network
- Monitor performance
- Adjust as needed

### TCP Tuning Parameters

#### Linux sysctl Parameters

```bash
# Increase buffer sizes
net.core.rmem_max = 16777216
net.core.wmem_max = 16777216
net.ipv4.tcp_rmem = 4096 87380 16777216
net.ipv4.tcp_wmem = 4096 65536 16777216

# Enable window scaling
net.ipv4.tcp_window_scaling = 1

# Enable SACK
net.ipv4.tcp_sack = 1

# Reduce TIME_WAIT
net.ipv4.tcp_tw_reuse = 1
net.ipv4.tcp_fin_timeout = 30

# Congestion control
net.ipv4.tcp_congestion_control = bbr
```

### Performance Optimization

#### 1. Enable TCP Fast Open (TFO)
- Reduces handshake overhead
- Sends data in SYN packet
- Requires application support

#### 2. Use Multiple Connections
- Parallel connections for better throughput
- Balance with server capacity
- Consider HTTP/2 multiplexing

#### 3. Optimize for Your Use Case
- Low latency: Reduce buffers
- High throughput: Increase buffers
- Many connections: Connection pooling

#### 4. Monitor and Adjust
- Measure performance
- Identify bottlenecks
- Tune parameters
- Test changes

---

## Advanced Topics

### TCP Fast Open (TFO)

**Purpose:** Reduce connection establishment overhead

**How it works:**
1. First connection: Normal handshake + TFO cookie
2. Subsequent connections: Send data in SYN packet
3. Server validates cookie and processes data

**Benefits:**
- Faster connection setup
- Lower latency
- Better user experience

**Limitations:**
- Requires application support
- Security considerations
- Not all networks support

### Multipath TCP (MPTCP)

**Purpose:** Use multiple paths simultaneously

**Benefits:**
- Increased throughput
- Better reliability
- Load balancing

**Use Cases:**
- Mobile devices (WiFi + Cellular)
- Data centers (multiple links)
- High availability

### TCP BBR

**Purpose:** Better congestion control for high-speed networks

**How it works:**
- Estimates bandwidth and RTT
- Maintains optimal queue size
- Reduces latency

**Benefits:**
- Higher throughput
- Lower latency
- Better fairness

### TCP Offloading

**Purpose:** Reduce CPU load

**Types:**
- **TSO (TCP Segmentation Offload)**: Hardware segments large packets
- **LRO (Large Receive Offload)**: Hardware combines small packets
- **Checksum Offload**: Hardware calculates checksums

**Benefits:**
- Lower CPU usage
- Better performance
- Requires NIC support

---

## Real-World Examples

### Example 1: Web Browser Connection

```
1. Browser resolves DNS: www.example.com → 192.0.2.1
2. Browser initiates TCP connection:
   SYN: Client → Server (seq=1000)
   SYN-ACK: Server → Client (seq=5000, ack=1001)
   ACK: Client → Server (seq=1001, ack=5001)
3. Browser sends HTTP request over TCP
4. Server sends HTTP response over TCP
5. Browser closes connection:
   FIN: Client → Server
   ACK: Server → Client
   FIN: Server → Client
   ACK: Client → Server
```

### Example 2: File Transfer (FTP)

```
1. Control connection established (port 21)
2. Client requests file transfer
3. Data connection established (port 20)
4. File data transferred over TCP
5. Data connection closed
6. Control connection remains open
```

### Example 3: Database Connection

```
1. Application connects to database
2. TCP handshake completes
3. Database authentication over TCP
4. Queries/results exchanged over TCP
5. Connection kept alive (persistent)
6. Connection closed when done
```

---

## Conclusion

TCP is a fundamental protocol that provides reliable, ordered, and error-checked delivery of data over IP networks. Understanding TCP is essential for:

- **Network Engineers**: Designing and troubleshooting networks
- **Application Developers**: Building reliable applications
- **System Administrators**: Optimizing system performance
- **Security Professionals**: Protecting against attacks

### Key Takeaways

- TCP is connection-oriented and reliable
- Three-way handshake establishes connections
- Four-way handshake terminates connections
- Sequence numbers and ACKs ensure reliability
- Flow control prevents receiver overload
- Congestion control prevents network overload
- Proper tuning improves performance
- Monitoring helps identify issues

### Further Reading

- [RFC 793](https://tools.ietf.org/html/rfc793): Transmission Control Protocol
- [RFC 5681](https://tools.ietf.org/html/rfc5681): TCP Congestion Control
- [RFC 7323](https://tools.ietf.org/html/rfc7323): TCP Extensions for High Performance
- [RFC 7414](https://tools.ietf.org/html/rfc7414): A Roadmap for TCP Specification Documents
- [TCP/IP Illustrated, Volume 1](https://en.wikipedia.org/wiki/TCP/IP_Illustrated): Comprehensive TCP reference

---

*Last Updated: May 2024*

