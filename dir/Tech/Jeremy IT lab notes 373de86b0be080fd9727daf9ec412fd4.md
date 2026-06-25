# Jeremy IT lab notes

# Day 1 (Network Devices)

---

# Day 2 (Interfaces and Cables)

---

# Day 3 (How the TCP/IP Model Actually Works)

---

# Day 4 (Intro to the CLI)

---

# Day 5 (Ethernet LAN Switching (Part 1))

## Ethernet LAN Switching — Part 1

**Key Takeaways**

- Ethernet operates at Layer 1 (Physical) and Layer 2 (Data Link) of the OSI model
- MAC addresses are 48-bit physical addresses; the first 24 bits = OUI, last 24 bits = device identifier
- Switches learn MAC addresses by reading the **source** MAC address of received frames
- Unknown unicast frames are **flooded** out all ports except the receiving port
- Known unicast frames are **forwarded** only to the correct port
- Dynamic MAC addresses are removed after **5 minutes of inactivity** on Cisco switches

---

### OSI Layer Review

**Layer 1 — Physical**

- Defines physical characteristics: voltage levels, cables, connectors, transmission distances
- Converts digital bits into electrical or radio signals

**Layer 2 — Data Link**

- Provides node-to-node connectivity (PC ↔ switch, switch ↔ router)
- Uses Layer 2 addressing (MAC addresses) — separate from Layer 3 IP addresses
- Detects and may correct Physical Layer errors
- Switches operate at Layer 2

---

### What is a LAN?

A **Local Area Network (LAN)** is a network contained within a relatively small area (office, home).

- **Switches** expand a LAN but do not separate them
- **Routers** connect separate LANs
- Two switches connected to each other = **one LAN**
- Two switches connected to different router interfaces = **two separate LANs**

---

### Ethernet Frame Structure

**PDU (Protocol Data Unit) at Layer 2 = Frame**

### Header Fields

| Field | Size | Purpose |
| --- | --- | --- |
| **Preamble** | 7 bytes | Alternating 1s and 0s (10101010 ×7) — synchronizes receiver clock |
| **SFD** (Start Frame Delimiter) | 1 byte | Pattern 10101011 — signals end of preamble, start of frame |
| **Destination MAC** | 6 bytes | Layer 2 address of the receiving device |
| **Source MAC** | 6 bytes | Layer 2 address of the sending device |
| **Type / Length** | 2 bytes | ≤1500 = length of payload; ≥1536 = type of encapsulated packet (0x0800 = IPv4, 0x86DD = IPv6) |

### Trailer Field

| Field | Size | Purpose |
| --- | --- | --- |
| **FCS** (Frame Check Sequence) | 4 bytes | Cyclic Redundancy Check (CRC) — detects transmission errors |

**Total header + trailer size = 26 bytes**

---

### MAC Addresses

- **48 bits (6 bytes)** in length — written as 12 hexadecimal characters
- Also called **BIA (Burned-In Address)** — assigned at manufacturing, not configurable
- **Globally unique** (no two devices share the same MAC address)

**Structure:**

| Portion | Bits | Purpose |
| --- | --- | --- |
| **OUI** (Organizationally Unique Identifier) | First 24 bits (3 bytes) | Assigned to the device manufacturer |
| **Device Identifier** | Last 24 bits (3 bytes) | Unique to the specific device |

**Example:** `E8BA.7011.2874` → OUI = `E8BA.70`, Device ID = `11.2874`

---

### Hexadecimal Basics

- Decimal uses 10 digits (0–9); Hexadecimal uses **16 digits** (0–9, A–F)
- A=10, B=11, C=12, D=13, E=14, F=15
- Each column represents a power of 16 (vs. power of 10 in decimal)
- Prefix `0x` indicates a hexadecimal value (e.g., `0x0800`)

---

### How Switches Learn and Forward Frames

### MAC Address Table

Switches maintain a **MAC address table** that maps MAC addresses to interfaces. Entries are built dynamically — no manual configuration required.

**Rule:** A switch learns MAC addresses by reading the **source MAC address** of every frame it receives, then records which interface that frame arrived on.

### Frame Types

| Type | Description | Switch Action |
| --- | --- | --- |
| **Unicast** | Destined for a single device | Forward or flood depending on table |
| **Unknown Unicast** | Destination MAC not in MAC address table | **Flood** out all ports except receiving port |
| **Known Unicast** | Destination MAC is in the MAC address table | **Forward** out the correct port only |

### Step-by-Step Process (Single Switch)

1. PC1 sends a frame to PC2 → frame arrives at SW1
2. SW1 reads **source MAC** → records PC1's MAC + interface in MAC address table
3. SW1 checks **destination MAC** → not in table = **Unknown Unicast** → **floods** frame
4. PC3 receives and **drops** the frame (wrong destination MAC)
5. PC2 receives and **processes** the frame
6. PC2 replies to PC1 → SW1 reads source MAC → learns PC2's MAC + interface
7. SW1 checks destination MAC (PC1) → now in table = **Known Unicast** → **forwards** directly

### Step-by-Step Process (Two Switches)

- Each switch independently learns MAC addresses from source fields
- A switch records the **incoming interface**, not the device's actual location
- The interface entry means "I can reach this MAC via this port" — not necessarily directly connected
- Both switches flood unknown unicast frames; both learn from replies

### MAC Address Aging

- Dynamic MAC address entries are removed after **5 minutes of inactivity**
- If the device sends traffic again, the switch relearns the MAC address automatically











---

# Day 6 (Ethernet LAN Switching (Part 2))

Here are your structured study notes for Notion:

---

# 📡 CCNA Day 6 — Ethernet LAN Switching (Deep Dive)

---

## ✅ Key Takeaways

- The **Ethernet header + trailer = 18 bytes**; minimum frame size is **64 bytes**, requiring a minimum payload of **46 bytes** (padding added if needed)
- **ARP (Address Resolution Protocol)** resolves IP addresses to MAC addresses — ARP Request is **broadcast**; ARP Reply is **unicast**
- **Ping** uses **ICMP Echo Request / Reply** to test reachability; the first ping often fails due to ARP lookup delay
- Switches build a **MAC address table** dynamically; entries age out after **5 minutes**
- The `show mac address-table` command displays four fields: **VLAN, MAC Address, Type, Ports**
- Switches flood **broadcast** and **unknown unicast** frames; they only forward **known unicast** frames

---

## 1. Ethernet Frame Details

### Structure

- **Preamble + SFD** — sent with every frame but generally *not* counted as part of the Ethernet header
- **Ethernet Header** = Destination MAC + Source MAC + Type = **14 bytes**
- **Ethernet Trailer** (FCS) = **4 bytes**
- **Total Header + Trailer = 18 bytes**

### Minimum Frame Size

- **Minimum Ethernet frame size: 64 bytes** (excludes Preamble + SFD)
- **Minimum payload size: 46 bytes** (64 − 18 = 46)
- If payload < 46 bytes → **padding bytes (0x00) are added**
    - Example: 34-byte payload → 12 bytes of padding added

### EtherType Values

| Protocol | EtherType |
| --- | --- |
| IPv4 | 0x0800 |
| IPv6 | 0x86DD |
| ARP | 0x0806 |

---

## 2. ARP — Address Resolution Protocol

### What It Does

- Resolves a known **Layer 3 address (IP)** → unknown **Layer 2 address (MAC)**
- Required because switches operate at **Layer 2** and need MAC addresses, not IPs

### Two ARP Messages

| Message | Type | Direction |
| --- | --- | --- |
| **ARP Request** | **Broadcast** (FFFF.FFFF.FFFF) | Sent to all hosts on the network |
| **ARP Reply** | **Unicast** | Sent only to the requesting device |

### ARP Request Process (Example: PC1 → PC3)

1. PC1 knows PC3's IP (`192.168.1.3`) but not its MAC
2. PC1 sends **ARP Request** with destination MAC = `FFFF.FFFF.FFFF`
3. SW1 learns PC1's MAC → adds to MAC address table (dynamic)
4. SW1 **floods** the frame out all interfaces except the receiving one
5. PC2 and PC4 **ignore** it (destination IP doesn't match)
6. PC3 recognizes its own IP → sends **ARP Reply**

### ARP Reply Process

1. PC3 sends unicast ARP Reply to PC1 (it already knows PC1's MAC from the request)
2. SW2 learns PC3's MAC → adds to MAC address table
3. SW2 **forwards** the frame (known unicast — no flooding)
4. PC1 receives reply → stores PC3's MAC in its **ARP table**

### ARP Table

- **Windows/macOS/Linux command:** `arp -a`
- **Cisco IOS command:** `show arp`
- Columns: Internet Address | Physical Address | Type
    - **Static** = default entry, not learned via ARP
    - **Dynamic** = learned via ARP Request/Reply

---

## 3. Ping (ICMP)

### What It Does

- Tests **reachability** between two hosts
- Measures **round-trip time**

### Two ICMP Messages

| Message | Direction |
| --- | --- |
| **ICMP Echo Request** | Sent from source host |
| **ICMP Echo Reply** | Sent back from destination |

### Cisco IOS Ping Behavior

- Default: sends **5 ICMP Echo Requests**, 100 bytes each
- `.` = failed ping | `!` = successful ping
- **First ping often fails** due to ARP lookup delay
- Command: `ping <ip-address>`
    - Example: `ping 192.168.1.3`
    - Custom size: `ping 192.168.1.3 size 36`

---

## 4. MAC Address Table on Cisco Switches

### Viewing the Table

```
show mac address-table
```

> Note: older IOS uses `show mac-address-table` (with leading hyphen)
> 

### Table Fields

| Field | Description |
| --- | --- |
| **VLAN** | Default is VLAN 1 |
| **MAC Address** | The learned hardware address |
| **Type** | `dynamic` (learned) or `static` (manual) |
| **Ports** | Interface the MAC was learned on |

### MAC Address Aging

- **Dynamic MACs are removed after 5 minutes** of inactivity (aging)

### Clearing the MAC Address Table

| Command | Effect |
| --- | --- |
| `clear mac address-table dynamic` | Clears **all** dynamic MACs |
| `clear mac address-table dynamic address <mac>` | Clears a **specific MAC** |
| `clear mac address-table dynamic interface <int>` | Clears all MACs on a **specific interface** |

---

## 5. Frame Forwarding Behavior Summary

| Frame Type | Switch Action |
| --- | --- |
| **Known Unicast** | Forward out the specific interface in the MAC table |
| **Unknown Unicast** | Flood out all interfaces except the one it was received on |
| **Broadcast** | Flood out all interfaces except the one it was received on |

---

## 6. Quick Command Reference

| Command | Platform | Purpose |
| --- | --- | --- |
| `arp -a` | Windows/macOS/Linux | View ARP table |
| `show arp` | Cisco IOS | View ARP table |
| `show mac address-table` | Cisco IOS | View MAC address table |
| `clear mac address-table dynamic` | Cisco IOS | Clear all dynamic MACs |
| `ping <ip>` | Cisco IOS | Send ICMP Echo Requests |

---

## 7. Quiz Review

| # | Question | Answer |
| --- | --- | --- |
| 1 | What are the `00` padding bytes in a packet capture? | Padding added to meet 46-byte minimum payload |
| 2 | Which message is sent to all hosts on the local network? | **ARP Request** (broadcast) |
| 3 | What fields are in `show mac address-table`? | VLAN, MAC Address, Type, Ports |
| 4 | What frame types does a switch flood? | **Broadcast** and **Unknown Unicast** |
| 5 | Command to clear dynamic MACs on a specific interface? | `clear mac address-table dynamic interface <int-id>` |

---

# Day 7 (IPv4 Addressing (Part 1))

# 🌐 CCNA Day 7 — Introduction to IPv4 Addressing

---

## ✅ Key Takeaways

- **Routers** (Layer 3) separate networks; **switches** (Layer 2) only expand them — broadcasts do not cross routers
- IPv4 addresses are **32 bits**, written in **dotted decimal** format (4 octets of 8 bits each)
- An IP address has two parts: the **network portion** and the **host portion**, defined by the **prefix length** (e.g. /24) or **netmask** (e.g. 255.255.255.0)
- IPv4 classes: **Class A (/8)**, **Class B (/16)**, **Class C (/24)** are the three usable classes
- The **network address** (host bits all 0s) and **broadcast address** (host bits all 1s) **cannot be assigned to hosts**
- The **127.x.x.x** range is reserved for **loopback** — not part of Class A's usable range

---

## 1. OSI Model — Layer 3 Review

- **Layer 2 (Data Link)** — uses **MAC addresses**; handles switching within a LAN
- **Layer 3 (Network)** — uses **IP addresses**; handles routing *between* different networks
- Layer 3 responsibilities:
    - Connectivity between hosts on **different networks**
    - **Logical addressing** (IP addresses assigned by configuration, not burned in)
    - **Path selection** — choosing the best route across complex networks
- **Routers** are Layer 3 devices

---

## 2. How Routers Separate Networks

- **Switches** connect and expand a single LAN — broadcasts reach all connected devices
- **Routers** divide networks — **broadcasts do not cross a router**
- Each router interface belongs to a different network and needs its own IP address

### Example Topology

| Device | IP Address |
| --- | --- |
| PC1 | 192.168.1.1/24 |
| PC2 | 192.168.1.2/24 |
| R1 G0/0 | 192.168.1.254/24 |
| PC3 | 192.168.2.1/24 |
| PC4 | 192.168.2.2/24 |
| R1 G0/1 | 192.168.2.254/24 |
- A broadcast from PC1 reaches PC2 and R1's G0/0 — but **stops at the router**, never reaching PC3 or PC4

---

## 3. IPv4 Address Structure

### Dotted Decimal Format

- IPv4 = **32 bits** total, split into **4 octets** of 8 bits each
- Written as four decimal numbers separated by periods: `192.168.1.254`
- Each octet ranges from **0 to 255**

### Binary Representation

- Computers process addresses in **binary (base 2)**
- Each bit position doubles in value moving left: `128 | 64 | 32 | 16 | 8 | 4 | 2 | 1`

| Decimal | Binary |
| --- | --- |
| 192 | 11000000 |
| 168 | 10101000 |
| 1 | 00000001 |
| 254 | 11111110 |

> **Octet** = one 8-bit group within an IP address (the standard term you'll see on the exam)
> 

---

## 4. Binary ↔ Decimal Conversion

### Binary → Decimal

1. Write bit values above each digit: `128 | 64 | 32 | 16 | 8 | 4 | 2 | 1`
2. Add up values where a `1` appears

**Example:** `10001111` = 128 + 8 + 4 + 2 + 1 = **143**

### Decimal → Binary

1. Write out bit values: `128 | 64 | 32 | 16 | 8 | 4 | 2 | 1`
2. Starting from 128, subtract each value if possible — write `1`; if not, write `0`

**Example:** 207 → 128✓ → 64✓ → 32✗ → 16✗ → 8✓ → 4✓ → 2✓ → 1✓ = **11001111**

> **Range of one octet:** 0 (`00000000`) to 255 (`11111111`)
> 

---

## 5. Network vs. Host Portion

- The **prefix length** (e.g. `/24`) defines how many bits are the **network portion**
- Remaining bits = **host portion**

| Prefix | Network Bits | Host Bits | Example |
| --- | --- | --- | --- |
| /8 | First 8 bits (1 octet) | Last 24 bits | `12`.128.251.23 |
| /16 | First 16 bits (2 octets) | Last 16 bits | `154.78`.111.32 |
| /24 | First 24 bits (3 octets) | Last 8 bits | `192.168.1`.254 |
- Hosts on the **same network** share the same network portion; only the host portion differs

---

## 6. IPv4 Address Classes

| Class | First Octet Bits | First Octet Range | Prefix | Netmask | Networks | Hosts/Network |
| --- | --- | --- | --- | --- | --- | --- |
| **A** | 0xxx | 0–126 | /8 | 255.0.0.0 | 128 | ~16.7 million |
| **B** | 10xx | 128–191 | /16 | 255.255.0.0 | ~16,000 | ~65,000 |
| **C** | 110x | 192–223 | /24 | 255.255.255.0 | ~2 million | 254 |
| D | 1110 | 224–239 | — | — | Multicast (reserved) | — |
| E | 1111 | 240–255 | — | — | Experimental (reserved) | — |

> **Class A ends at 126**, not 127 — the 127.x.x.x range is reserved for loopback
> 

---

## 7. Loopback Addresses

- **Reserved range:** `127.0.0.0` – `127.255.255.255`
- Used to test the **local device's own network stack**
- Traffic sent to any 127.x.x.x address is processed locally — never sent to the network
- Round-trip time = **0 ms** (traffic never leaves the device)
- Most common loopback address: **`127.0.0.1`**

---

## 8. Netmasks

Two equivalent ways to express the prefix length:

| Notation | Class A | Class B | Class C |
| --- | --- | --- | --- |
| **Prefix (slash)** | /8 | /16 | /24 |
| **Dotted Decimal Netmask** | 255.0.0.0 | 255.255.0.0 | 255.255.255.0 |
- **Network portion → all 1s**
- **Host portion → all 0s**
- Cisco devices use **dotted decimal netmask** notation; Juniper uses slash notation

---

## 9. Special Addresses — Network & Broadcast

### Network Address

- Host portion = **all 0s**
- Identifies the **network itself** — cannot be assigned to a host
- **First** address in the network
- Example: `192.168.1.0/24`
- First **usable** host address = network address + 1 → `192.168.1.1`

### Broadcast Address

- Host portion = **all 1s**
- Used to send traffic to **all hosts** on the local network — cannot be assigned to a host
- **Last** address in the network
- Example: `192.168.1.255/24`
- Last **usable** host address = broadcast address − 1 → `192.168.1.254`
- When a packet is sent to the broadcast IP, the destination MAC = **`FFFF.FFFF.FFFF`**

### Usable Host Range (Class C Example: 192.168.1.0/24)

| Address | Value | Assignable? |
| --- | --- | --- |
| Network address | 192.168.1.0 | ❌ No |
| First usable | 192.168.1.1 | ✅ Yes |
| Last usable | 192.168.1.254 | ✅ Yes |
| Broadcast address | 192.168.1.255 | ❌ No |

---

## 10. Quick Reference — Binary Bit Values

| Bit Position | 8 | 7 | 6 | 5 | 4 | 3 | 2 | 1 |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
| **Decimal Value** | 128 | 64 | 32 | 16 | 8 | 4 | 2 | 1 |

> Memorize this row — it's the foundation of all IP address binary conversion on the CCNA exam.
> 

---

# Day 8 (IPv4 Addressing (Part 2))

# 🌐 CCNA Day 8 — IPv4 Addressing Part 2 & Cisco Router Configuration

---

## ✅ Key Takeaways

- Maximum usable hosts formula: **2ⁿ − 2** (where n = number of host bits)
- **Network address** (host bits all 0s) and **broadcast address** (host bits all 1s) are never assignable to hosts
- First usable = **network address + 1**; Last usable = **broadcast address − 1**
- Cisco router interfaces are **administratively down by default** — must use `no shutdown` to enable
- Three essential interface commands: `ip address`, `no shutdown`, `description`
- Three essential show commands: `show ip interface brief`, `show interfaces`, `show interfaces description`

---

## 1. IPv4 Address Classes — Clarifications

### Class A Range Clarification

- Full binary range: 0–127, but **both 0 and 127 are reserved**
    - **0.x.x.x** — reserved (cannot be assigned)
    - **127.x.x.x** — reserved for loopback
- Safe to remember: Class A = **0–127**, but **usable range is 1–126**

### Class Summary Table

| Class | First Octet Range | Prefix | Network Bits | Host Bits |
| --- | --- | --- | --- | --- |
| **A** | 0–127 (usable: 1–126) | /8 | 8 | 24 |
| **B** | 128–191 | /16 | 16 | 16 |
| **C** | 192–223 | /24 | 24 | 8 |
| D | 224–239 | — | Multicast (reserved) | — |
| E | 240–255 | — | Experimental (reserved) | — |

---

## 2. Calculating Network Values

### Maximum Hosts Per Network

**Formula: 2ⁿ − 2** (n = number of host bits; subtract 2 for network + broadcast addresses)

| Class | Host Bits | Total Addresses | Max Usable Hosts |
| --- | --- | --- | --- |
| **A** | 24 | 16,777,216 | **16,777,214** |
| **B** | 16 | 65,536 | **65,534** |
| **C** | 8 | 256 | **254** |

### Network Address

- Host portion = **all 0s**
- **Cannot** be assigned to a host
- Is the **first** address in the network
- Example: `192.168.1.0/24`

### Broadcast Address

- Host portion = **all 1s**
- **Cannot** be assigned to a host
- Is the **last** address in the network
- Example: `192.168.1.255/24`

### First & Last Usable Addresses

- **First usable** = Network address + 1
- **Last usable** = Broadcast address − 1

### Quick Reference Examples

| Network | Class | Network Address | Broadcast | First Usable | Last Usable | Max Hosts |
| --- | --- | --- | --- | --- | --- | --- |
| 10.0.0.0/8 | A | 10.0.0.0 | 10.255.255.255 | 10.0.0.1 | 10.255.255.254 | 16,777,214 |
| 172.16.0.0/16 | B | 172.16.0.0 | 172.16.255.255 | 172.16.0.1 | 172.16.255.254 | 65,534 |
| 192.168.1.0/24 | C | 192.168.1.0 | 192.168.1.255 | 192.168.1.1 | 192.168.1.254 | 254 |

---

## 3. Configuring IP Addresses on Cisco Routers

### Key Difference: Routers vs. Switches

- **Cisco router interfaces** → **administratively down by default** (shutdown applied automatically)
- **Cisco switch interfaces** → NOT administratively down; status depends on whether a cable is connected

### Step-by-Step Configuration

```
R1> enable
R1# configure terminal
R1(config)# interface gigabitethernet0/0
R1(config-if)# ip address 10.255.255.254 255.0.0.0
R1(config-if)# no shutdown
```

### Interface Command Shortcuts

| Full Command | Common Shortcut |
| --- | --- |
| `configure terminal` | `conf t` |
| `interface gigabitethernet0/0` | `int g0/0` |
| `ip address` | `ip add` |
| `no shutdown` | `no shut` |

> You can switch directly between interfaces without typing `exit` first — just type the next `int` command
> 

### Subnet Mask Reference

| Prefix | Subnet Mask (Dotted Decimal) |
| --- | --- |
| /8 | 255.0.0.0 |
| /16 | 255.255.0.0 |
| /24 | 255.255.255.0 |

### Configuring Interface Descriptions (Optional but Recommended)

```
R1(config-if)# description ##Connected to SW1##
```

- No strict formatting rules — use whatever helps identify the interface's purpose
- Hashtags or labels help make descriptions stand out in output

---

## 4. Essential Show Commands

### `show ip interface brief`

```
R1# show ip interface brief
```

Displays a summary of all interfaces. Key columns:

| Column | Meaning |
| --- | --- |
| **Interface** | Interface name |
| **IP-Address** | Assigned IP (or "unassigned") |
| **OK?** | Legacy field — ignore |
| **Method** | How IP was set (`manual`, `unset`, `DHCP`) |
| **Status** | **Layer 1** status (`up` / `administratively down` / `down`) |
| **Protocol** | **Layer 2** status (`up` / `down`) |
- `administratively down` = interface disabled by `shutdown` command
- You will **never** see Status = `down` with Protocol = `up` (reverse is possible)
- After `no shutdown` + cable connected → both columns should show **`up`**

---

### `show interfaces [interface]`

```
R1# show interfaces gigabitethernet0/0
```

- Shows detailed **Layer 1, 2, and some Layer 3** info
- Key fields:
    - **Hardware** — interface type and **MAC address**
    - **BIA** (Burned-In Address) — the physical MAC; listed separately in case a custom MAC has been configured
    - **Internet address** — the IP address with prefix length
    - Line status and protocol status

---

### `show interfaces description`

```
R1# show interfaces description
```

- Shows Status, Protocol, and the **Description** column
- Useful for quickly identifying what each interface connects to

---

## 5. Status Column Interpretation

| Status | Protocol | Meaning |
| --- | --- | --- |
| `up` | `up` | ✅ Fully operational |
| `administratively down` | `down` | ❌ Shutdown command applied |
| `down` | `down` | ❌ No cable / physical issue |
| `up` | `down` | ⚠️ Layer 1 OK, Layer 2 issue |

---

## 6. Quiz Reference — Finding Network Values

**Given any IP/prefix, derive these 5 values:**

1. **Network address** → set all host bits to 0
2. **Max hosts** → 2ⁿ − 2
3. **Broadcast address** → set all host bits to 1
4. **First usable** → network address + 1
5. **Last usable** → broadcast address − 1

### Worked Examples

| IP Address | Network Address | Broadcast | First Usable | Last Usable | Max Hosts |
| --- | --- | --- | --- | --- | --- |
| 43.109.23.12/8 | 43.0.0.0 | 43.255.255.255 | 43.0.0.1 | 43.255.255.254 | 16,777,214 |
| 129.221.23.13/16 | 129.221.0.0 | 129.221.255.255 | 129.221.0.1 | 129.221.255.254 | 65,534 |
| 209.211.3.22/24 | 209.211.3.0 | 209.211.3.255 | 209.211.3.1 | 209.211.3.254 | 254 |
| 2.71.209.233/8 | 2.0.0.0 | 2.255.255.255 | 2.0.0.1 | 2.255.255.254 | 16,777,214 |
| 155.200.201.141/16 | 155.200.0.0 | 155.200.255.255 | 155.200.0.1 | 155.200.255.254 | 65,534 |

---

# Day 9 (Switch Interfaces)

---

# Day 10 (IPv4 Header)

---

# Day 11 (Routing Fundamentals (Part 1))

---

# Day 11 (Static Routing (Part 2))

---

# Day 12 (The Life of a Packet)

---

# Day 13 (Subnetting (Part 1))

---

# Day 14 (Subnetting (Part 2))

---

# Day 15 (Subnetting (Part 3 - VLSM))

---

# Day 16 (VLANs (Part 1))

**🎯 Key Takeaways**

- A **LAN** is formally defined as a **single broadcast domain** — the group of devices that receive a broadcast frame (destination MAC `FFFF.FFFF.FFFF`) sent by any one member. **Switches flood** broadcasts out all interfaces except the source; **routers never forward** broadcasts between networks.
- Splitting a network into Layer 3 **subnets alone doesn't isolate broadcast traffic** — devices in different subnets but on the same switch are still in the same Layer 2 broadcast domain unless the switch is also segmented.
- **VLANs (Virtual LANs)** logically split a single physical switch/broadcast domain into multiple separate broadcast domains at **Layer 2** — solving both **performance** (less unnecessary broadcast traffic) and **security** (limits which hosts can directly reach each other) problems.
- VLANs are configured **per switch interface**; a host's VLAN membership is determined by the VLAN assigned to the port it connects to.
- Switches **never forward traffic directly between VLANs** (not even within the same subnet) — inter-VLAN traffic must pass through a **router** for inter-VLAN routing.
- **VLAN 1** and **VLANs 1002–1005** exist by default on every Cisco switch and **cannot be deleted**; all interfaces are in VLAN 1 unless reconfigured.
- Assigning an interface to a VLAN number that doesn't exist yet will **automatically create that VLAN**.
- Key commands: `switchport mode access`, `switchport access vlan [#]`, `vlan [#]` + `name [name]`, and `show vlan brief` to verify.

---

**1. What Is a LAN? (Broadcast Domains)**

- **LAN (Local Area Network)** = a **single broadcast domain**.
- **Broadcast domain:** the set of devices that receive a broadcast frame (destination MAC = all Fs) sent by any member of the group.
- **Switch behavior:** floods broadcast frames out of **all interfaces except the one it was received on**.
- **Router behavior:** **does not forward** broadcast frames to other connected networks — this is what defines the boundary of a broadcast domain.
- **Worked example:** in a network with 3 switches each connected to a router, plus a direct router-to-router link, there are **4 separate broadcast domains** — one per switch/router-interface group, plus one for the router-to-router connection (which counts as its own broadcast domain even with only two devices).

---

2. The Problem: Subnets Alone Aren't Enough

- Scenario: a company LAN (`192.168.1.0/24`) has Engineering, HR, and Sales departments all on one switch, all in the same broadcast domain.
- **Performance issue:** a broadcast from any department reaches *every* device on the LAN (and the router), wasting bandwidth.
- **Security issue:** because all hosts are in the same LAN, they can reach each other **directly**, bypassing any security policies configured on the router/firewall — those policies only apply to traffic that actually passes through the router.

### Attempted Fix: Layer 3 Segmentation (Subnets Only)

- Splitting into subnets per department (e.g., `192.168.1.0/26`, `192.168.1.64/26`, `192.168.1.128/26`) requires the router to have **one interface per subnet**.
- **Unicast traffic between subnets** now correctly routes through R1 (source/destination MAC rewritten by the router), allowing security policy enforcement on that traffic.
- **However:** broadcast and unknown-unicast frames are still a problem. The switch only understands **Layer 2** — it has no awareness of the Layer 3 subnet boundaries, so a broadcast frame (even one intended only for one subnet) still gets **flooded to the entire switch**, including all departments and the router.
- **Conclusion:** subnetting alone splits Layer 3 addressing but does **not** split the Layer 2 broadcast domain — all hosts remain in the same LAN.
- Buying a separate physical switch per department would solve this but is **expensive and inflexible** for most organizations.

---

## 3. The Solution: VLANs (Layer 2 Segmentation)

- **VLANs (Virtual LANs)** let you logically divide a single physical switch (and its broadcast domain) into multiple **separate** broadcast domains — without needing separate physical hardware.
- Example assignment: Engineering → **VLAN 10**, HR → **VLAN 20**, Sales → **VLAN 30**.
- VLAN membership is configured **on switch interfaces**; whichever VLAN an interface is assigned to determines the VLAN of the connected end host.
- **Behavior:** the switch treats each VLAN as its own separate LAN. Broadcast and unknown-unicast frames arriving on an interface in one VLAN are only flooded to **other interfaces in that same VLAN** — never across VLANs.
- **Inter-VLAN communication:** even if two hosts in different VLANs happen to be in the same IP subnet, the switch will **never forward traffic directly between VLANs**. Traffic must always be sent to a router for **inter-VLAN routing** (other inter-VLAN routing methods exist and are covered in a later video).

---

## 4. VLAN Configuration on Cisco Switches

### Default State

- `show vlan brief` displays all VLANs and their assigned interfaces.
- **VLAN 1** (named `default`) — all switch interfaces belong here unless reconfigured.
- **VLANs 1002–1005** — reserved for legacy FDDI/Token Ring technologies (not needed for CCNA).
- ⚠️ **VLAN 1 and VLANs 1002–1005 exist by default and cannot be deleted.**

### Assigning Interfaces to a VLAN (Access Ports)

```
interface range g1/0 - g1/3
 switchport mode access
 switchport access vlan 10
```

- **`interface range`** — configures multiple interfaces at once.
- **`switchport mode access`** — sets the port as an **access port**: a switchport belonging to a single VLAN, typically connecting to end hosts (gives them "access" to the network). Ports connected to end hosts default to access mode via autonegotiation, but it's best practice to configure this explicitly rather than relying on autonegotiation.
- **`switchport access vlan [#]`** — assigns the interface to the specified VLAN. If that VLAN doesn't already exist, the switch **automatically creates it** (confirmed by the message: *"% Access VLAN does not exist. Creating vlan [#]"*).
- **Trunk ports** (which carry multiple VLANs) are a separate, more advanced topic covered in the next video — this lesson focuses only on access ports.

### Naming VLANs

```
vlan 10
 name ENGINEERING
```

- The `vlan [#]` command both **creates** a VLAN (if it doesn't exist) and enters VLAN configuration mode.
- `name [name]` assigns a descriptive name (otherwise VLANs have a generic default name).
- Verify with `show vlan brief` — names will now display as configured (e.g., ENGINEERING, HR, SALES).

### Verification Example

- A `ping 255.255.255.255` (broadcast) sent from a host in VLAN 10 will only reach **other hosts in VLAN 10** — confirming that broadcast traffic is now contained within each VLAN.

---

## 5. Summary of VLAN Purpose & Behavior

- VLANs are configured **per interface**, logically separating hosts at **Layer 2** even if they're physically connected to the same switch.
- Hosts in different VLANs are placed into **separate broadcast domains**, even though they share the same physical hardware.
- Switches **never** forward traffic directly between VLANs — it must pass through a router (or other Layer 3 device) for inter-VLAN routing.
- **Two main benefits:**
    - **Performance:** reduces unnecessary broadcast/flooded traffic, lowering congestion.
    - **Security:** prevents devices in different VLANs from communicating directly, ensuring all inter-VLAN traffic can be inspected/controlled at the router.

---

## 6. Quiz Review

**Q1:** How many broadcast domains exist in a network with no VLANs configured (all hosts in default VLAN 1)?

- **Answer: 6.** Each router interface and everything connected to it forms one broadcast domain.

**Q2:** How many broadcast domains exist once VLANs are configured?

- **Answer: 5.** One broadcast domain per configured VLAN, plus one for the router-to-router connection.

**Q3:** What happens if you assign a switch interface to a VLAN that doesn't exist yet?

- **Answer: B — the switch will create the VLAN automatically.**

**Q4:** If PC3 sends a broadcast message within its VLAN (VLAN 20, containing PC3, another PC, and the router), how many devices receive it?

- **Answer: 3** — the switch, the router, and the other PC in the same VLAN. (Without VLANs, all PCs on the switch would have received it.)

**Q5:** After creating VLANs 10, 20, and 30, how many total VLANs appear in `show vlan brief`?

- **Answer: C — 8.** The default VLAN 1 plus VLANs 1002–1005 (5 default VLANs total) plus the 3 newly created VLANs = 8.

---

# Day 17 (VLANs (Part 2))

# 🔀 VLANs Part 2 — Trunk Ports & Router on a Stick (ROAS)

**Source:** Jeremy's IT Lab – Free CCNA Course (Day 17)
**Exam Topics:** VLAN Trunking · 802.1Q Encapsulation · Inter-VLAN Routing (ROAS)

---

## ⚡ Key Takeaways

- **Trunk ports** carry traffic from **multiple VLANs** over a single physical interface — essential when VLANs scale beyond a few
- **802.1Q (dot1q)** is the industry-standard trunking protocol; ISL is Cisco-proprietary and essentially obsolete
- The **dot1q tag** (4 bytes) is inserted between the Source MAC and Type/Length fields of the Ethernet frame
- The **VID field** (12 bits) in the dot1q tag identifies the VLAN — supports VLANs **1 to 4094**
- **Native VLAN** frames are sent **untagged** over a trunk — must match on both ends or traffic problems occur
- By default, **all VLANs (1–4094)** are allowed on a trunk; limit this for security and performance
- **Router on a Stick (ROAS)** uses router **subinterfaces** to route between multiple VLANs over a single physical link
- `show interfaces trunk` shows trunk port details; `show vlan brief` shows **access ports only** — not trunks

---

## 1. 🔁 Why Trunk Ports?

### The Problem with Access-Port-Only Designs

- With few VLANs: a separate physical link per VLAN between switches/routers is workable
- With many VLANs: requires too many physical interfaces — **routers often don't have enough ports**
- Wasted interfaces, wasted cabling, poor scalability

### The Solution: Trunk Ports

- A **trunk port** carries traffic from **multiple VLANs** over a **single physical interface**
- Also called a **tagged port** (vs. access ports = **untagged ports**)
- Frames sent over trunks are **tagged** with the VLAN ID so the receiving device knows which VLAN each frame belongs to

---

## 2. 🏷️ VLAN Tagging — 802.1Q (dot1q)

### Two Trunking Protocols

| Protocol | Type | Status |
| --- | --- | --- |
| **ISL (Inter-Switch Link)** | Cisco proprietary | ❌ Obsolete — rarely supported on modern Cisco devices |
| **IEEE 802.1Q (dot1q)** | Industry standard | ✅ Use this — required for CCNA |

### dot1q Tag Placement

- Inserted **between the Source MAC address and the Type/Length fields** of the Ethernet frame
- Tag is **4 bytes (32 bits)** total

### dot1q Tag Structure

`[ TPID - 16 bits ] [ TCI: PCP (3 bits) | DEI (1 bit) | VID (12 bits) ]`

| Field | Size | Purpose |
| --- | --- | --- |
| **TPID** (Tag Protocol Identifier) | 16 bits | Always set to **0x8100** — signals this is a dot1q-tagged frame |
| **PCP** (Priority Code Point) | 3 bits | Class of Service (CoS) — prioritizes traffic in congested networks |
| **DEI** (Drop Eligible Indicator) | 1 bit | Marks frames that can be dropped if network is congested |
| **VID** (VLAN ID) | 12 bits | **Identifies the VLAN** — the most important field |

### VLAN ID (VID) Range

- 12 bits → 2¹² = **4096 total values**
- VLANs **0 and 4095** are reserved → **usable range: 1 to 4094**

### VLAN Ranges

| Range | Name | Notes |
| --- | --- | --- |
| **1 – 1005** | Normal VLANs | Supported on all switches |
| **1006 – 4094** | Extended VLANs | Not supported on some older switches |

---

## 3. 🌐 Native VLAN

- **dot1q feature** (ISL does not have this)
- Default native VLAN = **VLAN 1** on all trunk ports
- Must be configured **per trunk port** — not a global setting
- Frames in the native VLAN are sent **untagged** over the trunk
- When a switch receives an **untagged frame on a trunk**, it assumes it belongs to the **native VLAN**

### ⚠️ Native VLAN Mismatch

- If native VLANs don't match between two switches, traffic **may be misassigned to the wrong VLAN**
- Switches will still forward traffic but **problems will occur**
- **Always ensure native VLAN matches on both ends of a trunk**

### Security Best Practice

- Change native VLAN from the default (VLAN 1) to an **unused VLAN** to reduce unnecessary traffic and improve security

---

## 4. ⚙️ Trunk Port Configuration

### Step 1 — Set Encapsulation (if required)

> Only needed on switches that support both dot1q and ISL (encapsulation defaults to "auto").
Switches that only support dot1q skip this step.
> 

`SW1(config-if)# switchport trunk encapsulation dot1q`

### Step 2 — Set Port to Trunk Mode

`SW1(config-if)# switchport mode trunk`

### Step 3 — Configure Allowed VLANs

| Command | Effect |
| --- | --- |
| `switchport trunk allowed vlan 10,30` | Allow **only** VLANs 10 and 30 |
| `switchport trunk allowed vlan add 20` | **Add** VLAN 20 to existing list |
| `switchport trunk allowed vlan remove 20` | **Remove** VLAN 20 from list |
| `switchport trunk allowed vlan all` | Allow **all** VLANs 1–4094 (default state) |
| `switchport trunk allowed vlan except 1-5,10` | Allow all VLANs **except** specified ones |
| `switchport trunk allowed vlan none` | Allow **no** VLANs (blocks all traffic) |

### Step 4 — Change Native VLAN (security best practice)

`SW1(config-if)# switchport trunk native vlan 1001`

### Verification Commands

| Command | Shows |
| --- | --- |
| `show interfaces trunk` | Trunk interfaces, encapsulation, mode, native VLAN, allowed VLANs |
| `show vlan brief` | Access port VLAN assignments **only** — trunk ports do NOT appear here |

### show interfaces trunk — Key Output Fields

- **Mode:** `on` = manually configured as trunk
- **Encapsulation:** `dot1q`
- **Native VLAN:** configured native VLAN
- **VLANs allowed on trunk:** full allowed list
- **VLANs allowed and active in management domain:** allowed VLANs that **exist** on the switch

> 💡 A VLAN must exist on the switch to appear in the "allowed and active" section, even if it's in the allowed list.
> 

---

## 5. 🍢 Router on a Stick (ROAS)

### What Is ROAS?

- A method of **inter-VLAN routing** using a **single physical interface** on the router
- The one physical cable between router and switch looks like a "stick" on topology diagrams
- Uses **subinterfaces** on the router — logical divisions of one physical interface

### When to Use ROAS

- When only one physical interface is available between the router and switch
- More efficient than using a separate physical interface per VLAN

### How It Works

- The switch port connecting to the router is configured as a **trunk**
- The router physical interface is divided into **subinterfaces** (e.g., G0/0.10, G0/0.20, G0/0.30)
- Each subinterface is assigned a **VLAN tag** and **IP address** (acts as default gateway for that VLAN)

### ROAS Configuration (Router Side)

`! Enable the physical interface
R1(config)# interface g0/0
R1(config-if)# no shutdown

! Subinterface for VLAN 10
R1(config)# interface g0/0.10
R1(config-if)# encapsulation dot1q 10
R1(config-if)# ip address 192.168.1.62 255.255.255.192

! Subinterface for VLAN 20
R1(config)# interface g0/0.20
R1(config-if)# encapsulation dot1q 20
R1(config-if)# ip address 192.168.1.126 255.255.255.192

! Subinterface for VLAN 30
R1(config)# interface g0/0.30
R1(config-if)# encapsulation dot1q 30
R1(config-if)# ip address 192.168.1.190 255.255.255.192`

### Key ROAS Rules

- Subinterface number **does not have to match** the VLAN number — but it **should** for clarity
- `encapsulation dot1q <vlan-id>` tells the router to treat frames tagged with that VLAN as arriving on this subinterface
- Frames **leaving** a subinterface are tagged with the VLAN configured on it
- The **physical interface itself has no IP address** — only subinterfaces do
- The switch side needs **no additional configuration** beyond setting the port as a trunk with correct VLANs allowed

### Verification

`R1# show ip interface brief       ! Shows physical interface + all subinterfaces
R1# show ip route                 ! Shows connected/local routes per subinterface`

### ROAS Inter-VLAN Routing Flow (Example)

`PC (VLAN10) → SW2 → [tagged VLAN10] → R1 G0/0.10
R1 routes to VLAN30 subnet → [tagged VLAN30] → SW2 → SW1 → destination PC (VLAN30)`

---

## 6. 📝 Quiz Answers (Self-Test)

| # | Question | Answer |
| --- | --- | --- |
| 1 | Command to send VLAN10 frames **untagged** on a trunk? | `switchport trunk native vlan 10` |
| 2 | Command to return allowed VLANs to **default state**? | `switchport trunk allowed vlan all` |
| 3 | `switchport mode trunk` is rejected — what fixes it? | `switchport trunk encapsulation dot1q` (set encapsulation first) |
| 4 | Which field of the 802.1Q tag identifies the VLAN? | **VID** (VLAN ID) — 12 bits |
| 5 | VLAN10 is in allowed list but missing from "active" section of `show interfaces trunk` — why? | **VLAN10 doesn't exist on the switch** — must be created first |

---

## 7. 🔑 Critical Reminders

- **Trunk = tagged port; Access = untagged port** — know both terms
- **dot1q TPID = 0x8100** — this is how a switch identifies a dot1q-tagged frame
- **VID = 12 bits → VLANs 1–4094** (0 and 4095 reserved)
- **Native VLAN must match** on both ends of a trunk — mismatch causes traffic to be assigned to wrong VLAN
- **Default native VLAN = VLAN 1** — best practice is to change it to an unused VLAN
- **Default allowed VLANs on trunk = all (1–4094)** — restrict for security
- **`show vlan brief` does NOT show trunk ports** — use `show interfaces trunk` instead
- **ROAS subinterface number should match VLAN number** (not required, but strongly recommended)
- The **physical interface in ROAS has no IP address** — only subinterfaces get IPs
- ISL is effectively dead — **always use dot1q**

---

# Day 18 (VLANs (Part 3))

# 🔀 VLANs Part 3 — Native VLAN on Router, Layer 3 Switching & SVIs

**Source:** Jeremy's IT Lab – Free CCNA Course (Day 18)
**Exam Topics:** Inter-VLAN Routing · Layer 3 Switching · SVIs · Native VLAN on Router

---

## ⚡ Key Takeaways

- Two methods to configure native VLAN on a router: **`encapsulation dot1q <vlan> native`** on a subinterface, OR assign IP directly to the **physical interface** (no subinterface needed)
- Native VLAN frames are **untagged** — confirmed by Wireshark: no dot1q header on native VLAN traffic
- **Layer 3 (Multilayer) switches** can both switch AND route — they have their own routing table
- **`ip routing`** must be enabled on a multilayer switch or inter-VLAN routing will NOT work
- **`no switchport`** converts a switch physical interface into a Layer 3 routed port (can assign IP)
- **SVIs (Switch Virtual Interfaces)** are virtual Layer 3 interfaces per VLAN — used as default gateways for hosts
- An SVI will only come **up/up** if: VLAN exists + at least one port in the VLAN is up + VLAN is not shut + SVI is not shut
- Layer 3 switching is the **preferred inter-VLAN routing method** in large networks — traffic doesn't need to leave the switch

---

## 1. 🔌 Native VLAN on a Router (ROAS)

### Why Use Native VLAN?

- Frames in the native VLAN are **untagged** → slightly smaller frames → marginally more efficient
- Security best practice is still to set native VLAN to an **unused VLAN** — but know how to configure it if needed

### Method 1 — Use a Subinterface with `native` keyword

`R1(config)# interface g0/0.10
R1(config-subif)# encapsulation dot1q 10 native
R1(config-subif)# ip address 192.168.1.62 255.255.255.192`

- The router treats untagged frames as belonging to this subinterface's VLAN
- Frames sent in this VLAN leave **untagged**

### Method 2 — Configure IP on the Physical Interface (no subinterface)

`R1(config)# no interface g0/0.10        ! Delete the subinterface
R1(config)# interface g0/0
R1(config-if)# ip address 192.168.1.62 255.255.255.192`

- No `encapsulation dot1q` command needed
- Physical interface handles the native VLAN automatically
- All other VLANs still use subinterfaces as normal

### Comparison

| Feature | Method 1 (Subinterface + native) | Method 2 (Physical interface) |
| --- | --- | --- |
| Subinterface required | ✅ Yes | ❌ No |
| `encapsulation dot1q` required | ✅ Yes (with `native`) | ❌ No |
| Behavior | Same — untagged frames = native VLAN | Same — untagged frames = native VLAN |

---

## 2. 🔬 Wireshark Analysis — dot1q Tag in Action

### Tagged Frame (non-native VLAN traffic: VLAN20 → R1)

Key fields visible in the Ethernet header:

- **Type: 802.1Q Virtual LAN** — present before the normal Type field
- **TPID = 0x8100** — identifies this as a dot1q-tagged frame
- **PCP = 0** — no special CoS priority
- **DEI = 0** — not marked as drop-eligible
- **VLAN ID = 20** — identifies the VLAN
- Normal **IPv4 Type field** follows after the dot1q tag

### Untagged Frame (native VLAN traffic: R1 → SW2, VLAN10)

- **No dot1q header present** — native VLAN frames are sent with a standard Ethernet header only
- Both R1 and SW2 know that untagged frames = VLAN10 (native VLAN configured on both sides)

---

## 3. 🔀 Layer 3 (Multilayer) Switching

### What Is a Multilayer Switch?

- Can perform both **Layer 2 switching** AND **Layer 3 routing**
- Has its own **routing table** (just like a router)
- Can be assigned IP addresses on interfaces
- Supports **routed ports** and **SVIs (Switch Virtual Interfaces)**

### Three Inter-VLAN Routing Methods Compared

| Method | Interfaces Used | Best For | Limitation |
| --- | --- | --- | --- |
| **Separate router interfaces** (Day 16) | One per VLAN | Very small networks | Runs out of router ports quickly |
| **Router on a Stick (ROAS)** (Day 17) | Single trunk | Small-medium networks | All routed traffic flows through one link — potential congestion |
| **Layer 3 Switch + SVIs** (Day 18) | SVIs (virtual) | Large networks ✅ Preferred | Requires multilayer switch |

### Why Layer 3 Switching Is Preferred for Large Networks

- Inter-VLAN routing happens **inside the switch** — no need to send traffic out to a router and back
- Reduces latency and congestion on the uplink
- Scales well with many VLANs

---

## 4. ⚙️ Layer 3 Switch Configuration

### Step 1 — Enable IP Routing on the Switch

`SW2(config)# ip routing`

> ⚠️ **Critical:** Without this command, the switch will NOT route between VLANs. Do not forget it.
> 

### Step 2 — Configure a Routed Port (Physical Interface → Layer 3)

`SW2(config)# interface g0/1
SW2(config-if)# no switchport          ! Converts to Layer 3 routed port
SW2(config-if)# ip address 192.168.1.193 255.255.255.252
SW2(config-if)# no shutdown`

- `no switchport` removes the Layer 2 switchport behavior
- Interface now functions like a router interface
- Verify: `show interfaces status` → VLAN column shows **"routed"** instead of a VLAN number

### Step 3 — Configure a Default Route (toward the router)

`SW2(config)# ip route 0.0.0.0 0.0.0.0 192.168.1.194`

- All traffic destined outside the LAN is forwarded to R1

### Step 4 — Configure R1 (remove ROAS, add point-to-point IP)

`R1(config)# no interface g0/0.10
R1(config)# no interface g0/0.20
R1(config)# no interface g0/0.30
R1(config)# default interface g0/0     ! Resets to default settings
R1(config)# interface g0/0
R1(config-if)# ip address 192.168.1.194 255.255.255.252
R1(config-if)# no shutdown`

---

## 5. 🖧 SVI (Switch Virtual Interface) Configuration

### What Is an SVI?

- A **virtual Layer 3 interface** created for a specific VLAN on a multilayer switch
- Acts as the **default gateway** for hosts in that VLAN
- Replaces the need for a router interface or ROAS subinterface for inter-VLAN routing

### SVI Configuration

`SW2(config)# interface vlan 10
SW2(config-if)# ip address 192.168.1.62 255.255.255.192
SW2(config-if)# no shutdown            ! SVIs are shutdown by default

SW2(config)# interface vlan 20
SW2(config-if)# ip address 192.168.1.126 255.255.255.192
SW2(config-if)# no shutdown

SW2(config)# interface vlan 30
SW2(config-if)# ip address 192.168.1.190 255.255.255.192
SW2(config-if)# no shutdown`

> 💡 SVIs are **shutdown by default** — always use `no shutdown` after creating one.
> 

### Requirements for an SVI to Be Up/Up

All four conditions must be met:

| Condition | Detail |
| --- | --- |
| ✅ **VLAN must exist on the switch** | Creating an SVI does NOT auto-create the VLAN (unlike access ports) |
| ✅ **At least one port in the VLAN must be up/up** | Either an access port in the VLAN, OR a trunk port that allows the VLAN |
| ✅ **The VLAN itself must not be shut down** | VLAN can be shut independently of the SVI |
| ✅ **The SVI must not be shut down** | Use `no shutdown` — SVIs default to shutdown |

> ⚠️ If you create an SVI for a VLAN that doesn't exist yet, the switch will **NOT** auto-create the VLAN. You must create it manually.
> 

---

## 6. 📋 Verification Commands

| Command | What It Shows |
| --- | --- |
| `show ip route` | Routing table — connected/local routes for SVIs and routed ports |
| `show ip interface brief` | All interfaces including SVIs and their IP/status |
| `show interfaces status` | Interface status — routed ports show **"routed"** in VLAN column |
| `show running-config` | Full config — confirm subinterface, SVI, and routed port settings |

---

## 7. 🔄 Traffic Flow: Layer 3 Switching vs. ROAS

### ROAS Flow (old method)

`PC (VLAN20) → SW2 → R1 [tagged VLAN20] → SW2 [tagged VLAN10] → SW1 → PC (VLAN10)`

- Traffic leaves SW2, goes to R1, then comes back to SW2 — inefficient

### Layer 3 Switch + SVI Flow (new method)

`PC (VLAN20) → SW2 [routes internally] → SW2 trunk [tagged VLAN10] → SW1 → PC (VLAN10)`

- Traffic is routed **inside SW2** — never leaves the switch for inter-VLAN routing

### Traffic to the Internet (outside LAN)

`Any PC → SW2 (default route → R1) → R1 → Internet`

- SW2's default route forwards external traffic to R1

---

## 8. 📝 Quiz Answers (Self-Test)

| # | Question | Answer |
| --- | --- | --- |
| 1 | Two valid ways to configure native VLAN on a router in ROAS? | **`encapsulation dot1q <vlan> native`** on subinterface · OR **IP address on the physical interface** (no encapsulation command needed) |
| 2 | SVI for VLAN225 is down/down after `no shutdown` — two possible causes? | **VLAN225 doesn't exist on the switch** · **No interfaces in VLAN225 are up/up** |
| 3 | Command to configure a switch interface as a routed port? | **`no switchport`** |
| Boson | Native VLAN changed to VLAN44 — which traffic is untagged over the trunk? | **VLAN 44 traffic** — native VLAN traffic is always sent untagged |

---

## 9. 🔑 Critical Reminders

- **`ip routing`** = enables routing on the switch globally — **required for inter-VLAN routing to work**
- **`no switchport`** = converts a physical port to a Layer 3 routed port
- **SVIs are shutdown by default** — always `no shutdown` after creating one
- Creating an SVI **does NOT auto-create the VLAN** — must exist separately
- **Native VLAN = untagged** — changing native VLAN changes which traffic is untagged on that trunk
- Layer 3 switching is the **preferred method** for inter-VLAN routing in large networks
- Deleted subinterfaces show as **"deleted"** in `show ip interface brief` until the router is reloaded — this is normal
- `show interfaces status` → routed ports display **"routed"** in the VLAN column (not a VLAN number)

---

# Day 19 (DTP/VTP)

# 🔄 DTP & VTP — CCNA Study Notes

**Source:** Jeremy's IT Lab – Free CCNA Course (Day 19)
**Exam Topics:** DTP & VTP (removed from 200-301 list, but may still appear on exam)

> ⚠️ Both protocols are **Cisco proprietary** — only run on Cisco devices. Know their purpose and basic behavior, but **do not use them in production networks**.
> 

---

## ⚡ Key Takeaways

- **DTP (Dynamic Trunking Protocol)** — automatically negotiates trunk/access status between Cisco switches; **disabled by default on newer switches (dynamic auto)**; should be disabled for security
- **Older switches default to dynamic desirable**; newer switches default to **dynamic auto**
- DTP does NOT form a trunk with non-switch devices (routers, PCs) — must manually configure trunk for ROAS
- **VTP (VLAN Trunking Protocol)** — syncs VLAN database from a central server switch to client switches; rarely used in modern networks
- **VTP Revision Number** is critical — the switch with the highest revision number wins; adding an old switch with a high revision can **wipe all VLANs** from your network
- VTP only syncs the **VLAN database** — interface assignments (e.g., `switchport access vlan 10`) must still be configured manually
- **Two ways to reset revision number to 0:** change VTP domain name OR change to transparent mode
- VTP **transparent mode** forwards VTP advertisements but does NOT sync its own VLAN database

---

## 1. 🤝 DTP (Dynamic Trunking Protocol)

### What Is DTP?

- Cisco proprietary protocol that allows switches to **automatically negotiate** whether an interface should be an **access port or a trunk port**
- **Enabled by default** on all Cisco switch interfaces
- **Security risk** — should be disabled on all switchports in production; manually configure access or trunk instead

### DTP Administrative Modes

| Mode | Behavior |
| --- | --- |
| `switchport mode trunk` | Manually forces trunk; still sends DTP frames |
| `switchport mode access` | Manually forces access; **disables DTP** (stops sending DTP frames) |
| `switchport mode dynamic desirable` | **Actively tries** to form a trunk |
| `switchport mode dynamic auto` | **Passively waits** — will form a trunk if the other end is actively trying |

### Default DTP Mode by Switch Age

| Switch Type | Default Administrative Mode |
| --- | --- |
| **Older switches** | `dynamic desirable` (actively tries to trunk) |
| **Newer switches** | `dynamic auto` (passive) |

### DTP Negotiation Outcome Chart

| SW1 Mode | SW2 Mode | Result |
| --- | --- | --- |
| **Access** | Access | Access |
| **Access** | Dynamic Auto | Access |
| **Access** | Dynamic Desirable | Access |
| **Access** | Trunk | ❌ Misconfig — do NOT do this |
| **Dynamic Auto** | Dynamic Auto | Access |
| **Dynamic Auto** | Dynamic Desirable | **Trunk** |
| **Dynamic Auto** | Trunk | **Trunk** |
| **Dynamic Desirable** | Dynamic Desirable | **Trunk** |
| **Dynamic Desirable** | Trunk | **Trunk** |
| **Trunk** | Trunk | **Trunk** |
| **Trunk** | Access | ❌ Misconfig — do NOT do this |

> 💡 **Rule of thumb:** A trunk forms only when at least one side is actively trying (desirable or manual trunk). Two passive (auto) sides → access.
> 

### Disabling DTP

`! Option 1 — Disable DTP negotiation on a trunk port
SW1(config-if)# switchport nonegotiate

! Option 2 — Configuring access mode also stops DTP frames
SW1(config-if)# switchport mode access`

> ⚠️ `switchport mode trunk` alone does NOT stop DTP frames — you must also add `switchport nonegotiate`.
> 

### Trunk Encapsulation Negotiation via DTP

- Switches supporting both dot1q and ISL can use DTP to negotiate encapsulation
- Default encapsulation mode: `switchport trunk encapsulation negotiate`
- **ISL is preferred over dot1q** when both are supported — ISL will be selected
- DTP frames are sent in **VLAN1** (ISL) or the **native VLAN** (dot1q, default VLAN1)

### Key Verification Command

`SW1# show interfaces g0/0 switchport`

- **Administrative mode** = what you configured
- **Operational mode** = what is actually running (trunk or access)
- **Negotiation of trunking** = On (DTP active) or Off (DTP disabled)

---

## 2. 📋 VTP (VLAN Trunking Protocol)

### What Is VTP?

- Cisco proprietary protocol that **synchronizes the VLAN database** from a central server switch to client switches
- Designed for large networks — avoids having to configure VLANs on every switch individually
- **Rarely used in modern networks** — carries significant risk (see revision number danger below)
- VTP advertisements are sent **only on trunk ports** — not access ports
- VTP **only syncs the VLAN database** — interface-level configuration (e.g., `switchport access vlan 10`) must still be done manually on each switch

### VTP Versions

| Version | Notes |
| --- | --- |
| **VTPv1** | Default; does not support extended VLANs (1006–4094) |
| **VTPv2** | Adds Token Ring VLAN support — no practical reason to use over v1 today |
| **VTPv3** | Supports extended VLANs; clients store VLAN DB in NVRAM; most differences |

### VTP Modes

| Mode | Add/Modify/Delete VLANs? | Syncs to Server? | Stores DB in NVRAM? | Forwards Advertisements? |
| --- | --- | --- | --- | --- |
| **Server** (default) | ✅ Yes | ✅ Yes (if higher rev exists) | ✅ Yes | ✅ Yes |
| **Client** | ❌ No | ✅ Yes | ❌ No (v1/v2) / ✅ Yes (v3) | ✅ Yes |
| **Transparent** | ✅ Yes (local only) | ❌ No | ✅ Yes | ✅ Yes (same domain only) |

> 💡 **VTP Servers also act as VTP clients** — they will sync to another server with a higher revision number.
> 

### VTP Revision Number

- **Increments by 1** every time a VLAN is added, modified, or deleted on a server
- The switch with the **highest revision number** is considered to have the most current VLAN database
- All servers and clients will sync to the highest revision number in the domain

### ⚠️ The VTP Revision Number Danger

> If you connect an **old switch** with a **higher revision number** and the **same VTP domain name**, ALL switches in the domain will sync to that old switch's VLAN database — potentially **deleting all VLANs** and causing a network outage.
> 

**How to safely add an old switch to a network using VTP — reset revision to 0 first:**

`! Method 1 — Change to an unused VTP domain
SW(config)# vtp domain UNUSED_NAME

! Method 2 — Change to transparent mode
SW(config)# vtp mode transparent`

Both methods reset the revision number to **0**.

---

## 3. ⚙️ VTP Configuration

### Basic VTP Commands

`! Set VTP domain name
SW1(config)# vtp domain CISCO

! Change VTP mode
SW1(config)# vtp mode client
SW1(config)# vtp mode transparent
SW1(config)# vtp mode server       ! Default

! Change VTP version
SW1(config)# vtp version 2`

### VTP Domain Behavior

- Default domain name = **NULL** (no domain)
- If a switch with a NULL domain receives a VTP advertisement with a domain name, it will **automatically join that domain**
- VTP advertisements are only forwarded by transparent mode switches if the advertisement is in the **same VTP domain**

### Key Verification Command

`SW1# show vtp status`

Key fields to check:

- **VTP version running**
- **VTP domain name**
- **VTP operating mode** (server/client/transparent)
- **Configuration revision number**
- **Number of existing VLANs**
- **Maximum VLANs supported** (1005 for v1/v2; extended range for v3)

---

## 4. 📝 Quiz Answers (Self-Test)

| # | Question | Answer |
| --- | --- | --- |
| 1 | Old spare switch forms a trunk with SW1 after being reset — why? | Old switches default to **`dynamic desirable`**; newer switches default to `dynamic auto` → trunk is formed |
| 2 | SW2 should forward VTP ads to SW3 but NOT sync its own VLAN DB — which command? | **`vtp mode transparent`** |
| 3 | Two methods to reset VTP revision number to 0? | **Change VTP domain to an unused name** · **Change to VTP transparent mode** |
| DTP Chart | Auto + Desirable = ? | **Trunk** |
| DTP Chart | Auto + Auto = ? | **Access** |
| DTP Chart | Trunk + Access = ? | **❌ Misconfig** |

---

## 5. 🔑 Critical Reminders

- **Disable DTP in production** — use `switchport mode access` or `switchport nonegotiate` on trunks
- **DTP cannot form a trunk with a router or PC** — always manually configure `switchport mode trunk` for ROAS uplinks
- **Highest VTP revision number wins** — always reset an old switch's revision before connecting it to a VTP domain
- **VTP transparent mode:** forwards ads (same domain only), maintains its own independent VLAN DB, does NOT sync to server
- **VTP does NOT configure interfaces** — only syncs the VLAN database; port assignments must still be configured per switch
- `show interfaces <int> switchport` → check **administrative mode** (configured) vs **operational mode** (actual)
- `show vtp status` → check revision number, domain, mode, and VLAN count
- Changing VTP version increases the revision number and triggers a new round of advertisements

---

# Day 20 (Spanning Tree Protocol (Part 1))

# 🌳 CCNA Day 20 — Spanning Tree Protocol (STP) Part 1

---

## ✅ Key Takeaways

- **Redundant networks are essential**, but without STP, redundant Layer 2 paths cause **broadcast storms** and **MAC address flapping**
- STP (IEEE 802.1D) prevents loops by placing redundant ports in a **blocking state**
- The switch with the **lowest Bridge ID** becomes the **root bridge** — all its ports are **designated (forwarding)**
- Every other switch selects exactly **one root port** — the path with the lowest **root cost**
- Every collision domain must have exactly **one designated port** — the other port becomes **non-designated (blocking)**
- STP port cost by speed: 10 Mbps = **100**, 100 Mbps = **19**, 1 Gbps = **4**, 10 Gbps = **2**
- Bridge priority can only be changed in **increments of 4096**; default is **32768 + VLAN ID**

---

## 1. Network Redundancy

### Why It Matters

- Networks must run **24/7/365** — even brief downtime can be catastrophic for businesses
- If one component fails, **another must take over with little or no downtime**
- Redundancy should be implemented at **every possible point** in the network

### Poor vs. Good Network Design

- **Poor design** — single paths everywhere; one failure cuts off entire segments
- **Good design** — multiple paths between switches and routers so traffic can reroute around failures
- **Limitation:** Most PCs have a single NIC, so they connect to only one switch; **critical servers** often have multiple NICs for redundancy

---

## 2. The Problem: Layer 2 Loops & Broadcast Storms

### How a Broadcast Storm Forms

1. PC sends an **ARP request** (broadcast frame — destination MAC `FFFF.FFFF.FFFF`)
2. Each switch **floods** the frame out all interfaces except the one it arrived on
3. With redundant links and **no STP**, frames loop between switches **indefinitely**
4. Ethernet frames have **no TTL field** (unlike IP packets) — nothing stops the loop
5. Looping frames accumulate until the network is **too congested for legitimate traffic**

### MAC Address Flapping

- Every time a frame arrives, the switch updates its MAC address table with the **source MAC + incoming interface**
- Looping frames cause the same source MAC to appear on **different interfaces repeatedly**
- This constant updating is called **MAC address flapping** and destabilizes the MAC table

---

## 3. STP Introduction

### Key Facts

- **Standard:** IEEE 802.1D (Classic STP)
- Runs on switches from **all vendors**, enabled **by default**
- The term **"bridge"** in STP terminology = **switch** (bridges are obsolete; the term persists)

### How STP Prevents Loops

- Identifies redundant ports and places them in a **blocking state** (effectively disabled)
- Blocked ports act as **standby backups** — they transition to forwarding if an active port fails

### Two Port States

| State | Behavior |
| --- | --- |
| **Forwarding** | Sends and receives all normal traffic |
| **Blocking** | Only sends/receives **BPDUs** (Bridge Protocol Data Units); no regular traffic |

---

## 4. STP Process Overview

STP follows a **3-step process** to build a loop-free topology:

```
Step 1 → Elect Root Bridge
Step 2 → Select Root Ports (one per non-root switch)
Step 3 → Select Designated / Non-Designated Ports (one designated per collision domain)
```

---

## 5. Step 1 — Root Bridge Election

### Bridge ID Structure (Traditional)

| Field | Size |
| --- | --- |
| Bridge Priority | 16 bits |
| MAC Address | 48 bits |
- Default priority = **32768** on all switches
- **Lowest Bridge ID wins** → if priorities tie, **lowest MAC address** wins

### Updated Bridge ID (Modern Cisco — PVST)

| Field | Size |
| --- | --- |
| Bridge Priority | 4 bits |
| Extended System ID (VLAN ID) | 12 bits |
| MAC Address | 48 bits |
- **PVST** = Per-VLAN Spanning Tree — runs a **separate STP instance per VLAN**
- Default bridge priority in VLAN 1 = **32768 + 1 = 32769**
- Priority can only be changed in **increments of 4096** (the value of the least significant bit of the 4-bit priority field)

### Valid Bridge Priority Values

`0, 4096, 8192, 12288, 16384, 20480, 24576, 28672, 32768, 36864, 40960, 45056, 49152, 53248, 57344, 61440`*(VLAN ID is then added to whichever value you set)*

### Root Bridge Rules

- When a switch powers on, it **assumes it is the root bridge**
- It gives up that role only if it receives a **superior BPDU** (from a switch with a lower Bridge ID)
- Once topology converges, **only the root bridge generates BPDUs** — other switches forward them
- **All ports on the root bridge = Designated ports (Forwarding)**

---

## 6. Step 2 — Root Port Selection

- Every **non-root switch** selects exactly **one root port**
- Root ports are always in a **forwarding state**
- The port connected to a root port is always a **designated port**

### STP Port Cost by Interface Speed

| Speed | STP Cost |
| --- | --- |
| 10 Mbps (Ethernet) | **100** |
| 100 Mbps (FastEthernet) | **19** |
| 1 Gbps (GigabitEthernet) | **4** |
| 10 Gbps | **2** |

### Root Cost Calculation

- **Root cost** = sum of **outgoing** interface costs along the path to the root bridge
- The root bridge advertises cost **0**; each switch adds its own outgoing interface cost when forwarding BPDUs
- The **receiving interface cost is NOT counted** — only the outgoing interface

### Root Port Tiebreakers (in order)

| Priority | Tiebreaker |
| --- | --- |
| 1st | **Lowest root cost** |
| 2nd | **Lowest neighbor Bridge ID** |
| 3rd | **Lowest neighbor port ID** (not the local port — the neighbor's port) |

### Port ID Structure

- Port ID = **Port Priority** (default 128) + **Port Number** (e.g., G0/0 = 1, G0/1 = 2)
- Lower port number = lower port ID = wins the tiebreaker

---

## 7. Step 3 — Designated & Non-Designated Port Selection

- Every **collision domain** (link between two switches) must have **exactly one designated port**
- The other port becomes **non-designated (blocking)**

### Designated Port Selection (per collision domain)

| Priority | Criteria |
| --- | --- |
| 1st | Switch with **lowest root cost** → its port is designated |
| 2nd (tie) | Switch with **lowest Bridge ID** → its port is designated |
- The **losing switch's port** becomes **non-designated (blocking)**

---

## 8. Complete STP Process Summary

```
1. Elect Root Bridge
   └── Lowest Bridge ID wins
   └── All root bridge ports = Designated (Forwarding)

2. Select Root Ports (one per non-root switch)
   └── Lowest root cost → wins
   └── Tie: lowest neighbor Bridge ID → wins
   └── Tie: lowest neighbor port ID → wins
   └── Root ports = Forwarding
   └── Ports connected to root ports = Designated (Forwarding)

3. Select Designated / Non-Designated Ports
   └── One designated port per collision domain
   └── Lowest root cost → designated
   └── Tie: lowest Bridge ID → designated
   └── Other port = Non-Designated (Blocking)
```

---

## 9. STP Port Roles & States Reference

| Port Role | State | Where It Appears |
| --- | --- | --- |
| **Designated** | Forwarding | All root bridge ports; one per collision domain on non-root switches |
| **Root Port** | Forwarding | One per non-root switch (best path to root bridge) |
| **Non-Designated** | Blocking | Redundant ports — prevents loops |

---

## 10. Quiz Practice — Worked Examples

### How to Approach Any STP Topology Question

1. **Find the root bridge** — compare Bridge IDs (priority first, then MAC)
2. **Mark all root bridge ports as Designated**
3. **Find each switch's root port** — lowest root cost; use tiebreakers if needed
4. **Mark ports connected to root ports as Designated**
5. **For remaining links** — determine designated vs. non-designated using root cost then Bridge ID
6. **Verify** — every collision domain has exactly one designated port ✅

### Common Mistakes to Avoid

- The **neighbor's** port ID breaks ties for root port selection — not your own port
- Root cost counts **outgoing** interface costs only — not the incoming/receiving interface
- FastEthernet (cost 19) vs. GigabitEthernet (cost 4) can dramatically change which port is selected as root port

---

## 11. Key Terms Glossary

| Term | Definition |
| --- | --- |
| **STP** | Spanning Tree Protocol — prevents Layer 2 loops |
| **BPDU** | Bridge Protocol Data Unit — STP messages exchanged between switches |
| **Root Bridge** | The central switch in the STP topology; all ports forwarding |
| **Root Port** | A switch's best path toward the root bridge |
| **Designated Port** | The forwarding port on each collision domain segment |
| **Non-Designated Port** | A blocked port — prevents loops |
| **Root Cost** | Cumulative cost of outgoing interfaces along path to root bridge |
| **Bridge ID** | Priority + MAC address; used to elect the root bridge |
| **PVST** | Per-VLAN Spanning Tree — Cisco's STP implementation; separate instance per VLAN |
| **MAC Address Flapping** | Rapid MAC table updates caused by looping frames |
| **Broadcast Storm** | Network congestion caused by endlessly looping broadcast frames |

---

# Day 21 (Spanning Tree Protocol (Part 2))

# 🌳 CCNA Day 21 — Spanning Tree Protocol (STP) Part 2

---

## ✅ Key Takeaways

- STP has **4 active port states**: Blocking → Listening (15s) → Learning (15s) → Forwarding — total **30 seconds** to transition from blocking to forwarding
- A blocked port that stops receiving BPDUs waits up to **20 seconds (Max Age)** before reconverging — total worst-case transition time = **50 seconds**
- **Portfast** bypasses Listening and Learning states — use only on **end-host ports (access ports)**
- **BPDU Guard** shuts down a port if it receives a BPDU — combine with Portfast for protection
- Cisco's **PVST+** uses destination MAC `0100.0ccc.cccd`; standard STP uses `0180.c200.0000`
- STP timers on the **root bridge** govern the entire network — even if other switches are configured differently
- Use different root bridges per VLAN to achieve **STP load balancing**
- Bridge priority = **increments of 4096**; Port priority = **increments of 32**

---

## 1. STP Port States

### Overview — 5 States

| State | Type | Description |
| --- | --- | --- |
| **Blocking** | Stable | Non-designated ports; prevents loops |
| **Listening** | Transitional | 15 seconds; no traffic, no MAC learning |
| **Learning** | Transitional | 15 seconds; no traffic, **MAC learning begins** |
| **Forwarding** | Stable | Root and Designated ports; full operation |
| **Disabled** | N/A | Administratively shut down; not part of STP |

### Blocking State

- Applied to **non-designated ports**
- **Receives** STP BPDUs (stays aware of topology)
- Does **not forward** BPDUs
- Does **not** send/receive regular traffic
- Does **not** learn MAC addresses

### Listening State

- Entered by **Designated or Root ports** transitioning toward Forwarding
- Duration: **15 seconds** (Forward Delay timer)
- **Only** sends/receives STP BPDUs
- Does **not** send/receive regular traffic
- Does **not** learn MAC addresses

### Learning State

- Follows Listening state for **Designated or Root ports**
- Duration: **15 seconds** (Forward Delay timer)
- **Only** sends/receives STP BPDUs
- Does **not** send/receive regular traffic
- **Does learn MAC addresses** ← key difference from Listening

### Forwarding State

- Applied to stable **Root and Designated ports**
- Sends/receives **BPDUs**
- Sends/receives **regular traffic**
- **Learns MAC addresses**
- Operates as a normal switchport

### State Transition Rules

- **Forwarding → Blocking**: Can happen **immediately** (blocking never causes loops)
- **Blocking → Forwarding**: Must pass through **Listening then Learning** (30 seconds total)

---

## 2. STP Timers

### Three Timers

| Timer | Default | Purpose |
| --- | --- | --- |
| **Hello** | 2 seconds | How often the root bridge sends BPDUs |
| **Forward Delay** | 15 seconds | Duration of each transitional state (Listening + Learning = 30s total) |
| **Max Age** | 20 seconds | How long a port waits after missing BPDUs before reconverging |

> ⚠️ **STP timers are set on the root bridge and govern the entire network** — even if other switches are configured differently
> 

### Hello Timer — How BPDUs Flow

- Only the **root bridge** originates BPDUs (every 2 seconds)
- Other switches **forward** root bridge BPDUs **only out of their Designated ports**
- BPDUs are **not** forwarded out of Root ports or Non-designated ports

### Max Age Timer — Failure Detection

- Every time a port receives a BPDU, it **resets the Max Age timer to 20 seconds**
- If a port stops receiving BPDUs (e.g., upstream link failure), the timer counts down
- When Max Age reaches **0** → switch reevaluates root bridge and all port roles
- A previously blocking port may then transition: Blocking → Listening (15s) → Learning (15s) → Forwarding
- **Worst-case reconvergence time = 20 + 15 + 15 = 50 seconds**

### Message Age Field (BPDU)

- Starts at **0** at the root bridge
- Incremented by **1** each time a BPDU is forwarded by another switch
- Subtracted from Max Age when received — limits how far BPDUs can propagate

---

## 3. STP BPDU Structure

> You don't need to memorize every field for the CCNA, but understand the key ones.
> 

### Ethernet Header

- **Cisco PVST+** destination MAC: **`0100.0ccc.cccd`** ← memorize for exam
- **Standard STP** destination MAC: **`0180.c200.0000`** ← memorize for exam

### Key BPDU Fields

| Field | Value/Notes |
| --- | --- |
| Protocol Identifier | Always `0x0000` for STP |
| Protocol Version | `0` = Classic STP; different value for Rapid STP |
| BPDU Type | `0x00` = Configuration BPDU |
| Flags | Signal topology changes |
| **Root Identifier** | Priority + Extended System ID (VLAN) + Root Bridge MAC |
| **Root Path Cost** | `0` if sent by root bridge |
| **Bridge Identifier** | Sending switch's Bridge ID (same as Root ID if root bridge) |
| Port Identifier | `0x8002` = Priority 128 (0x80) + Port number 2 |
| **Timers** | Max Age, Hello, Forward Delay |
- **PVST** (older) = supports Cisco ISL trunking only
- **PVST+** (current) = supports **802.1Q (dot1q)** — what modern Cisco switches use

---

## 4. STP Optional Features (Toolkit)

### Portfast

**Purpose:** Allows a port to skip Listening and Learning states and go **directly to Forwarding**

- Use **only on access ports connected to end hosts** (PCs, printers, servers)
- **Never** enable on ports connected to other switches — can cause Layer 2 loops
- Even if configured on a trunk port, it will **not take effect**

**Configuration:**

```
! Interface level
SW(config-if)# spanning-tree portfast

! Global (applies to all access ports)
SW(config)# spanning-tree portfast default
```

---

### BPDU Guard

**Purpose:** Shuts down a port immediately if a BPDU is received — prevents loops caused by unauthorized switches

- Best used **alongside Portfast** on end-host ports
- If a BPDU arrives → port is placed in **err-disabled** state
- To re-enable: `shutdown` then `no shutdown` the interface
    - If the switch is still connected, the port will be immediately disabled again

**Configuration:**

```
! Interface level
SW(config-if)# spanning-tree bpduguard enable

! Global (applies to all Portfast-enabled ports)
SW(config)# spanning-tree portfast bpduguard default
```

> Note the command difference: interface-level uses `bpduguard enable`; global-level includes `portfast` in the command
> 

---

### Root Guard *(Know name + purpose)*

- If a port receives a **superior BPDU** (lower Bridge ID), the interface is **disabled**
- Prevents an unauthorized switch from becoming the root bridge
- Maintains your intended STP topology

### Loop Guard *(Know name + purpose)*

- If a port **stops receiving BPDUs**, it is **disabled** rather than transitioning to forwarding
- Protects against **unidirectional link failures** (link can send but not receive, or vice versa)

> For the CCNA exam: **know Portfast and BPDU Guard in depth**; Root Guard and Loop Guard — name and basic purpose only
> 

---

## 5. STP Configuration

### Set STP Mode

```
SW(config)# spanning-tree mode pvst          ! Classic STP (per-VLAN)
SW(config)# spanning-tree mode rapid-pvst    ! Rapid STP (default on modern Cisco)
```

> Modern Cisco switches run **Rapid PVST+ by default** — usually no need to change
> 

---

### Configure Root Bridge

```
! Make this switch the primary root bridge for VLAN 1
SW(config)# spanning-tree vlan 1 root primary
! → Sets priority to 24576 (or 4096 less than current lowest)

! Make this switch the secondary root bridge for VLAN 1
SW(config)# spanning-tree vlan 1 root secondary
! → Sets priority to 28672

! Manual priority configuration (must be in increments of 4096)
SW(config)# spanning-tree vlan 1 priority 4096
```

---

### Configure Port Cost and Priority (Per VLAN)

```
! Set spanning tree cost on an interface
SW(config-if)# spanning-tree vlan 1 cost <1-200000000>

! Set spanning tree port priority (increments of 32, range 0–224)
SW(config-if)# spanning-tree vlan 1 port-priority <0-224>
```

Use these to influence which port is selected as Root port or Designated port.

---

## 6. STP Load Balancing

- **PVST+** runs a **separate STP instance per VLAN** — each VLAN can have a different root bridge
- Blocking the **same link in every VLAN** wastes bandwidth on that redundant link
- **Solution:** Assign different root bridges for different VLANs → different links are blocked in different VLANs → bandwidth is utilized across all links

### Example Configuration

```
! On SW1 — Primary root for VLAN10, Secondary for VLAN20
SW1(config)# spanning-tree vlan 10 root primary
SW1(config)# spanning-tree vlan 20 root secondary

! On SW2 — Primary root for VLAN20, Secondary for VLAN10
SW2(config)# spanning-tree vlan 20 root primary
SW2(config)# spanning-tree vlan 10 root secondary
```

---

## 7. Complete STP Timers & States Reference

### Transition Timeline: Blocking → Forwarding

```
Blocking (stable)
   ↓  [Max Age counts down — 20 sec max]
Listening  ←── 15 seconds (Forward Delay)
   ↓
Learning   ←── 15 seconds (Forward Delay)
   ↓
Forwarding (stable)

Total worst case: 20 + 15 + 15 = 50 seconds
```

### What Each State Does

|  | Blocking | Listening | Learning | Forwarding |
| --- | --- | --- | --- | --- |
| Send/Receive BPDUs | Receive only | ✅ | ✅ | ✅ |
| Send/Receive traffic | ❌ | ❌ | ❌ | ✅ |
| Learn MAC addresses | ❌ | ❌ | ✅ | ✅ |

---

## 8. Key Commands Summary

| Command | Purpose |
| --- | --- |
| `spanning-tree portfast` | Enable Portfast on interface |
| `spanning-tree portfast default` | Enable Portfast on all access ports globally |
| `spanning-tree bpduguard enable` | Enable BPDU Guard on interface |
| `spanning-tree portfast bpduguard default` | Enable BPDU Guard on all Portfast ports globally |
| `spanning-tree mode [pvst/rapid-pvst]` | Set STP mode |
| `spanning-tree vlan X root primary` | Set switch as primary root for VLAN X |
| `spanning-tree vlan X root secondary` | Set switch as secondary root for VLAN X |
| `spanning-tree vlan X priority <value>` | Manually set bridge priority (multiples of 4096) |
| `spanning-tree vlan X cost <value>` | Set port cost for VLAN X |
| `spanning-tree vlan X port-priority <value>` | Set port priority for VLAN X (multiples of 32) |

---

## 9. Quiz Reference

| # | Question | Answer |
| --- | --- | --- |
| 8 | PC can't connect for ~30 sec after plugging in — two fixes? | **A (Portfast)** and **C (reduce Forward Delay timer)** |
| 9 | STP port ID = `0x8002` — what is the port priority? | **C — 128** (hex `0x80` = decimal 128) |
| 10 | Feature to prevent loops if user connects a switch to an access port? | **D — BPDU Guard** |
| Boson | `spanning-tree portfast default` — which ports use Portfast? | **D — All access ports** |

---

# Day 21 (PortFast (Part 1))

# 🌳 CCNA — STP Toolkit: PortFast (Deep Dive)

---

## ✅ Key Takeaways

- By default, a switch port takes **30 seconds** to start forwarding (15s Listening + 15s Learning) — even for end hosts
- **PortFast** bypasses Listening and Learning states, putting the port **immediately into Forwarding**
- PortFast should **only** be used on ports connected to **end hosts** — never on ports connected to other switches
- PortFast is **only active on access ports** by default — it won't activate on trunk ports unless explicitly configured with `portfast trunk`
- Modern Cisco switches automatically add the **EDGE** keyword to PortFast configs in the running-config
- Two config methods: **per-port** (`spanning-tree portfast`) or **global** (`spanning-tree portfast default` — applies to all access ports)
- `show spanning-tree interface <int> detail` confirms PortFast status

---

## 1. The Problem PortFast Solves

- When an end host connects to a switch port, the port becomes **up/up** physically but **cannot send or receive data** for 30 seconds
- The port is a **Designated port** transitioning through STP states:
    - **Listening** → 15 seconds
    - **Learning** → 15 seconds
    - **Forwarding** → finally able to pass traffic
- During this time, any frames the PC sends are **discarded by the switch**
- The user experiences "the internet doesn't work" with no explanation
- This delay is **unnecessary for end hosts** — Layer 2 loops can only form between switches, not between a switch and a PC

### Link Light Indicator (Physical Switch)

- **Amber/Orange** = Port is up but blocked by STP (Listening/Learning)
- **Green** = Port is in Forwarding state — traffic can pass

---

## 2. How PortFast Solves It

- When PortFast is enabled, the port **immediately enters Forwarding** when connected to a device
- **Bypasses** both the Listening and Learning states entirely
- The host can access the network **right away** — no 30-second wait
- Confirmed on a real switch: link light turns **green immediately** instead of amber for 30 seconds

---

## 3. PortFast Configuration

### Method 1 — Per-Port (Interface Config Mode)

```
SW(config-if)# spanning-tree portfast
```

- Enables PortFast on **that specific interface only**
- **Only active when the interface is in access mode** (not trunk mode)
- Will show a warning message upon configuration

> ⚠️ **Warning message:** "PortFast should only be enabled on ports connected to a single host. Connecting hubs, switches, bridges, etc. to this interface when PortFast is enabled can cause temporary bridging loops. Use with CAUTION."
> 

---

### Method 2 — Global (All Access Ports)

```
SW(config)# spanning-tree portfast default
```

- Enables PortFast on **all access ports** across the switch
- Does **not** activate on trunk ports
- To disable PortFast on a specific access port after using this command:

```
SW(config-if)# spanning-tree portfast disable
```

> ⚠️ **Note:** The warning message says "all interfaces" but it only activates on **access ports** — the warning is slightly misleading
> 

---

### Method 3 — Trunk Ports (Special Cases)

```
SW(config-if)# spanning-tree portfast trunk
```

Use when the trunk port connects to a device that **cannot cause Layer 2 loops**, such as:

- A **router** configured with router-on-a-stick
- A **virtualization server** with VMs in multiple VLANs (requires trunk link)

This command is **per-port only** — there is no global trunk version.

---

## 4. Verifying PortFast

```
SW# show spanning-tree interface g0/1 detail
```

Key lines to look for in output:

| Output | Meaning |
| --- | --- |
| `The port is in portfast edge mode` | PortFast configured per-port on access port |
| `The port is in portfast edge mode by default` | PortFast enabled via global command |
| `The port is in portfast edge trunk mode` | PortFast enabled on a trunk port |
| *(no mention of portfast)* | PortFast is not active on this port |

To view PortFast config for a specific interface:

```
SW# show running-config interface g0/1
```

---

## 5. PortFast Edge vs. PortFast Network

| Type | CCNA Topic? | Use Case |
| --- | --- | --- |
| **PortFast Edge** | ✅ Yes | End hosts, routers — covered in this video |
| **PortFast Network** | ❌ No | Used for Bridge Assurance — not a CCNA topic |

### The EDGE Keyword

- Modern Cisco switches **automatically append** `EDGE` to PortFast commands in the running-config
- You can use commands **with or without** EDGE — the result is identical

| Command You Type | What Appears in Running-Config |
| --- | --- |
| `spanning-tree portfast` | `spanning-tree portfast edge` |
| `spanning-tree portfast trunk` | `spanning-tree portfast edge trunk` |
| `spanning-tree portfast default` | `spanning-tree portfast edge default` |
| `spanning-tree portfast disable` | `spanning-tree portfast disable` *(no EDGE)* |

> ⚠️ **Packet Tracer note:** Current versions do **not** support the EDGE keyword — don't be confused if it doesn't work in labs
> 

---

## 6. PortFast Rules — What's Allowed

| Port Type | PortFast Allowed? | Command |
| --- | --- | --- |
| Access port → end host | ✅ Yes (recommended) | `spanning-tree portfast` |
| Access port → switch | ❌ No (risk of loop) | Do not configure |
| Trunk port → router (ROAS) | ✅ Yes (safe) | `spanning-tree portfast trunk` |
| Trunk port → virtualization server | ✅ Yes (safe) | `spanning-tree portfast trunk` |
| Trunk port → switch | ❌ No (risk of loop) | Do not configure |

---

## 7. Command Summary

| Command | Mode | Effect |
| --- | --- | --- |
| `spanning-tree portfast` | Interface | Enable PortFast on this access port |
| `spanning-tree portfast edge` | Interface | Same as above (explicit EDGE keyword) |
| `spanning-tree portfast trunk` | Interface | Enable PortFast on this trunk port |
| `spanning-tree portfast default` | Global | Enable PortFast on **all access ports** |
| `spanning-tree portfast disable` | Interface | Disable PortFast on a specific port |
| `show spanning-tree interface <int> detail` | Exec | Verify PortFast status on an interface |
| `show running-config interface <int>` | Exec | View running config for one interface |

---

## 8. Key Distinctions to Remember for the Exam

- PortFast **bypasses** Listening and Learning — it does **not** disable STP entirely
- The port is still a Designated port — it just enters Forwarding immediately
- PortFast is **inactive on trunk ports** unless `portfast trunk` is explicitly used
- **Never** enable PortFast on a port connected to another switch — use BPDU Guard alongside PortFast as a safety net (covered in the next video)
- `spanning-tree portfast default` enables PortFast **globally on access ports only**, despite the warning saying "all interfaces"

---

# Day 21 (BPDU Guard & BPDU Filter (Part 2))

# 🛡️ CCNA — STP Toolkit: BPDU Guard & BPDU Filter

---

## ✅ Key Takeaways

- A PortFast-enabled port **still sends BPDUs** and will revert to normal STP behavior if it receives one — this is the gap BPDU Guard fills
- **BPDU Guard** err-disables a port the moment it receives a BPDU — protects against unauthorized switches
- **BPDU Filter** stops a port from **sending** BPDUs — behavior on *receiving* a BPDU differs by configuration method
- ⚠️ **BPDU Filter per-port = STP disabled on that port** — can cause **permanent loops**; use with extreme caution
- **BPDU Filter by default (global)** = safe — if a BPDU is received, PortFast + BPDU Filter are disabled and the port acts as a normal STP port
- **ErrDisable Recovery** can auto-re-enable err-disabled ports — disabled by default, default timer = **300 seconds (5 min)**
- Always **fix the underlying problem** before re-enabling an err-disabled port

---

## 1. PortFast + BPDUs — The Gap

- A PortFast-enabled port **still sends BPDUs** every 2 seconds
- If it **receives** a BPDU → it **reverts to normal STP behavior** (PortFast is deactivated)
- End hosts don't send BPDUs, so this is normally fine
- **Problem:** If a user (e.g., "Bob from Accounting") connects an unauthorized switch to a PortFast port:
    - The switch sends BPDUs
    - The PortFast port processes them and acts as a regular STP port
    - If that switch has a **lower Bridge ID**, it can become the **root bridge**
    - This reshapes the entire STP topology — potentially blocking efficient paths and degrading network performance

---

## 2. BPDU Guard

### Purpose

Protects ports intended for end hosts from unauthorized switches by **err-disabling the port** the moment it receives a BPDU.

### How It Works

- Port continues to **send** BPDUs normally
- If it **receives** a BPDU → port enters **err-disabled state** immediately
- Port can no longer send or receive data or BPDUs
- Bob's switch is isolated; Bob contacts IT — mission accomplished

### Configuration

```
! Per-port (interface config mode)
SW(config-if)# spanning-tree bpduguard enable

! Global default (activates on all PortFast-enabled ports)
SW(config)# spanning-tree portfast bpduguard default

! Disable on a specific port after global enable
SW(config-if)# spanning-tree bpduguard disable
```

> ⚠️ **Key difference from PortFast default:**
> 
> - `spanning-tree portfast default` → activates on all **access ports**
> - `spanning-tree portfast bpduguard default` → activates on all **PortFast-enabled ports** (not necessarily all access ports)

### Verification

```
SW# show spanning-tree interface g0/1 detail
```

Look for: `BPDU Guard is enabled`

---

## 3. ErrDisable State

### What It Is

- A Cisco switch feature that **disables a port** when a violation is detected
- Causes include: BPDU Guard, Port Security, Power Policing, DAI (Dynamic ARP Inspection)
- CLI indicators when BPDU Guard triggers:
    - `"Received BPDU on port G0/1 with BPDU Guard enabled. Disabling port."`
    - `"bpduguard error detected on G0/1, putting G0/1 in err-disable state"`
    - Interface status → **down/down**
- **Physical indicator:** Link light turns **solid amber** and stays amber even after disconnecting the cable

### Re-enabling an Err-Disabled Port

> ⚠️ Always fix the underlying problem **first** — otherwise the port will be immediately err-disabled again
> 

**Method 1 — Manual:**

```
SW(config-if)# shutdown
SW(config-if)# no shutdown
```

**Method 2 — Automatic (ErrDisable Recovery):**

```
! View ErrDisable Recovery status
SW# show errdisable recovery

! Enable ErrDisable Recovery for BPDU Guard
SW(config)# errdisable recovery cause bpduguard

! Modify the recovery timer (default = 300 seconds)
SW(config)# errdisable recovery interval <seconds>
```

### ErrDisable Recovery Key Facts

- **Disabled by default** — must be explicitly enabled
- Default recovery timer: **300 seconds (5 minutes)**
- After the timer expires, the port is automatically re-enabled
- `show errdisable recovery` shows: which causes are enabled, recovery timer, and which ports are pending recovery

---

## 4. BPDU Filter

### Purpose

Prevents a port from **sending BPDUs** — reduces unnecessary bandwidth/processing and avoids leaking STP topology info to end-user devices.

### Two Configuration Methods — Behavior Differs

|  | **Per-Port (interface config)** | **Global Default** |
| --- | --- | --- |
| **Command** | `spanning-tree bpdufilter enable` | `spanning-tree portfast bpdufilter default` |
| **Applies to** | That specific interface | All PortFast-enabled ports |
| **Sends BPDUs?** | ❌ No | ❌ No |
| **Receives BPDUs?** | ❌ **Ignores them** | ✅ **Disables PortFast + BPDU Filter; acts as normal STP port** |
| **STP disabled?** | ✅ **Yes — effectively** | ❌ No — STP re-engages if BPDU received |
| **Loop risk?** | 🔴 **High — permanent loops possible** | 🟢 **Low — safe fallback behavior** |

### Configuration

```
! Per-port — USE WITH CAUTION
SW(config-if)# spanning-tree bpdufilter enable

! Global default — RECOMMENDED
SW(config)# spanning-tree portfast bpdufilter default

! Disable on a specific port after global enable
SW(config-if)# spanning-tree bpdufilter disable
```

> 🔴 **Warning:** Per-port BPDU Filter **disables STP** on the interface. If accidentally enabled on a port connected to another switch, it can cause **permanent Layer 2 loops** and broadcast storms.
> 

> ✅ **Recommendation:** Only enable BPDU Filter in **global config mode** unless you have a specific reason to use per-port configuration.
> 

---

## 5. BPDU Guard + BPDU Filter Together

Both can be enabled on the same port simultaneously — but the outcome depends on how BPDU Filter was configured:

| BPDU Filter Config | BPDU Received | BPDU Guard Triggered? |
| --- | --- | --- |
| **Global default** | BPDU Filter disabled → BPDU processed | ✅ Yes — port err-disabled |
| **Per-port** | BPDU **ignored** by BPDU Filter | ❌ No — BPDU Guard never sees it |

> This is another reason to prefer **global default** for BPDU Filter — it allows BPDU Guard to still protect the port.
> 

---

## 6. Feature Comparison Summary

| Feature | Sends BPDUs | Receives BPDUs | Action on Receiving BPDU | Risk if Misconfigured |
| --- | --- | --- | --- | --- |
| **PortFast** | ✅ Yes | ✅ Yes — reverts to normal STP | Acts as normal STP port | Temporary loop |
| **BPDU Guard** | ✅ Yes | ✅ Yes | **Err-disables the port** | Port unnecessarily disabled |
| **BPDU Filter (per-port)** | ❌ No | ❌ Ignores | Nothing — BPDU ignored | **Permanent loop** 🔴 |
| **BPDU Filter (global)** | ❌ No | ✅ Yes — disables itself | Reverts to normal STP port | Low risk |

---

## 7. Complete Command Reference

| Command | Mode | Purpose |
| --- | --- | --- |
| `spanning-tree bpduguard enable` | Interface | Enable BPDU Guard on this port |
| `spanning-tree portfast bpduguard default` | Global | Enable BPDU Guard on all PortFast ports |
| `spanning-tree bpduguard disable` | Interface | Disable BPDU Guard on this port |
| `spanning-tree bpdufilter enable` | Interface | Enable BPDU Filter (per-port) — caution! |
| `spanning-tree portfast bpdufilter default` | Global | Enable BPDU Filter on all PortFast ports |
| `spanning-tree bpdufilter disable` | Interface | Disable BPDU Filter on this port |
| `shutdown` + `no shutdown` | Interface | Manually re-enable an err-disabled port |
| `errdisable recovery cause bpduguard` | Global | Auto-recover ports disabled by BPDU Guard |
| `errdisable recovery interval <sec>` | Global | Set auto-recovery timer (default: 300s) |
| `show errdisable recovery` | Exec | View ErrDisable Recovery status and timers |
| `show spanning-tree interface <int> detail` | Exec | Verify BPDU Guard/Filter status on a port |
| `show interfaces <int>` | Exec | Confirm err-disabled state |

---

# Day 21 (Root Guard (Part 3))

# 🛡️ CCNA — STP Toolkit: Root Guard

---

## ✅ Key Takeaways

- **Root Guard** prevents a port from accepting a superior BPDU and allowing another switch to become root bridge
- Configure Root Guard on ports connecting to **switches outside your direct control** (e.g., customer switches in a service provider network)
- Root Guard is **interface-only** — no global default command exists
- If a Root Guard port receives a superior BPDU → port enters **"broken / root inconsistent"** state (blocked)
- Recovery is **automatic** — once superior BPDUs stop, the port re-enables itself after ~**20 seconds (Max Age)**; no manual intervention needed
- Unlike BPDU Guard, Root Guard does **not** use ErrDisable — it uses its own "root inconsistent" blocking state
- Good root bridge selection considers **optimal traffic flow** and **stability/reliability**

---

## 1. Why Root Bridge Placement Matters

### Two Key Considerations

**Optimal Traffic Flow**

- End hosts typically communicate with servers **outside the LAN** (internet, WAN) via the gateway router
- The root bridge should be placed so that each switch has an **efficient, direct path** to reach the router
- If a suboptimal switch is root, traffic may take **longer paths** → added latency and potential **congestion** on shared links

**Stability and Reliability**

- The root bridge is central to the entire STP topology — it must stay up
- In mixed environments, assign root bridge role to the **most reliable, modern switch**
- An old or unstable switch as root can cause repeated reconvergence events

### Example: Bad Root Bridge Placement

- SW1 connects directly to the router (R1) and is the intended root
- If SW3 becomes root instead:
    - SW3's hosts → efficient path
    - SW2's hosts → must travel SW2 → SW3 → SW1 → R1 (suboptimal)
    - The SW3→SW1 link becomes a bottleneck — **congestion risk** under heavy traffic

---

## 2. The Problem Root Guard Solves

### Scenario: Inter-Organization Switch Connections

- Common example: **Service provider** switches connected to a **customer's** switches
- Each organization controls its own root bridge (e.g., SP sets SW1 priority = 0; customer sets SW6 priority = 0)
- When the networks are connected, all switches exchange BPDUs and elect **one global root bridge**
- If SW6 (customer) has a **lower MAC address** than SW1 (provider), SW6 wins — even with equal priorities
- Result: The **customer's switch becomes the root bridge** for the service provider's LAN

### Impact of Losing Root Bridge Control

- Traffic paths through the SP's LAN are **reshaped around the customer's switch**
- Links in the SP's LAN get blocked that shouldn't be
- Frames take **inefficient detours** through the customer's network
- The SP has **no control** over the customer's switch priority or MAC address

---

## 3. Root Guard: The Solution

### What It Does

- Configured on **specific ports** connecting to untrusted/external switches
- If that port receives a **superior BPDU** (one claiming a better root bridge), the port is **blocked**
- The switch continues using its **own root bridge** and ignores the superior BPDU

### Port State When Root Guard Triggers

- Port status: **BKN (Broken)**
- Port reason: **ROOT_Inc (Root Inconsistent)**
- The port **cannot forward or receive data frames** — all traffic is cut off on that link
- The port still participates in STP processing internally

> **Note:** In a normal STP topology there is one designated port per link. When Root Guard triggers, both sides may appear as designated ports — this is an abnormal state specific to Root Guard blocking.
> 

---

## 4. Configuration

### Enable Root Guard (Interface Only)

```
SW(config-if)# spanning-tree guard root
```

- **No global default command** — must be configured **per port**
- Apply to ports connecting to switches **outside your administrative control**

### Why No Global Default?

- PortFast/BPDU Guard/BPDU Filter can be enabled globally because they target predictable port types (access ports, PortFast ports)
- Root Guard needs **selective placement** — enabling it on the wrong port would block legitimate uplinks

### Where to Apply Root Guard

| Port | Apply Root Guard? | Reason |
| --- | --- | --- |
| SP switch → customer switch | ✅ Yes | Prevent customer from becoming root |
| Customer switch → SP switch | ❌ No | Would block links to the provider's network |
| Internal switch → internal switch | ❌ Usually no | You control both switches — set priority directly |

---

## 5. Verification

```
SW# show spanning-tree
```

Look for in output:

| Column | Value | Meaning |
| --- | --- | --- |
| Status | `BKN` | Port is **broken** (blocked by Root Guard) |
| Reason | `ROOT_Inc` | Port is **root inconsistent** — disabled by Root Guard |

Log messages when Root Guard triggers:

```
%SPANTREE-2-ROOTGUARD_BLOCK: Root guard blocking port GigabitEthernet0/2 ...
```

Log message when port recovers:

```
%SPANTREE-2-ROOTGUARD_UNBLOCK: Root guard unblocking port GigabitEthernet0/2 ...
```

---

## 6. Recovery

### How to Re-enable a Root Guard-Blocked Port

Root Guard recovery is **automatic** — no CLI commands needed on the Root Guard switch.

**Steps:**

1. **Identify** why superior BPDUs are being received (e.g., customer switch has lower Bridge ID)
2. **Fix the source** — have the other organization increase their switch's STP priority
3. Once superior BPDUs stop arriving, the port **automatically recovers** after ~**20 seconds** (Max Age timer)
4. The previously blocked port transitions back to normal STP operation

> **Key difference from BPDU Guard:**
> 
> - **BPDU Guard** → err-disables port; requires manual `shutdown/no shutdown` or ErrDisable Recovery
> - **Root Guard** → root inconsistent state; **auto-recovers** when superior BPDUs stop

---

## 7. Root Guard vs. BPDU Guard — Comparison

| Feature | BPDU Guard | Root Guard |
| --- | --- | --- |
| **Triggers on** | Any BPDU received | **Superior** BPDU received |
| **Typical use case** | Ports to end hosts | Ports to external/untrusted switches |
| **Port state when triggered** | `err-disabled` | `broken / root inconsistent` |
| **Recovery method** | Manual or ErrDisable Recovery | **Automatic** when BPDUs stop |
| **Global config option** | ✅ Yes | ❌ No — interface only |
| **STP still runs?** | Port fully disabled | Port blocked but STP-aware |

---

## 8. Command Summary

| Command | Mode | Purpose |
| --- | --- | --- |
| `spanning-tree guard root` | Interface | Enable Root Guard on this port |
| `show spanning-tree` | Exec | View port status — look for `BKN` and `ROOT_Inc` |

---

## 9. Root Guard Process Summary

```
External switch sends superior BPDU
         ↓
Root Guard-enabled port receives it
         ↓
Port enters "broken / root inconsistent" state
         ↓
Port cannot forward or receive data frames
         ↓
Fix: External switch priority is increased
         ↓
Superior BPDUs stop arriving
         ↓
After ~20 seconds (Max Age), port auto-recovers
         ↓
Normal STP operation resumes with correct root bridge
```

---

# Day 21 (Loop Guard (Part 4))

# 🛡️ CCNA — STP Toolkit: Loop Guard

---

## ✅ Key Takeaways

- **Loop Guard** protects against Layer 2 loops caused by ports that **unexpectedly stop receiving BPDUs**
- Most common cause: **unidirectional links** on fiber-optic connections (one fiber damaged but interface stays up/up)
- Without Loop Guard: a non-designated blocking port that stops receiving BPDUs will **transition to Forwarding** — creating a loop
- With Loop Guard: that port instead enters **"broken / loop inconsistent"** state — no loop
- Recovery is **automatic** — when BPDUs resume, the port re-enables itself
- Loop Guard and Root Guard are **mutually exclusive** — cannot be active on the same port simultaneously
- Apply Loop Guard to **root ports and non-designated ports**; apply Root Guard to **designated ports** facing untrusted switches

---

## 1. What Is a Unidirectional Link?

### Definition

A network link where data flows in **only one direction** — the other direction is broken but the interface remains **up/up** (the failure goes undetected).

### Why It Happens

- Typically a **Layer 1 physical issue** — most common on **fiber-optic cables**
- Fiber-optic connections use **two separate fibers** (one Tx, one Rx per side)
- If **one fiber is damaged**, data flows in one direction only
- If the devices **detect** the failure → interface goes down entirely (no loop risk)
- If the devices **fail to detect** the failure → interface stays up/up → **unidirectional link**

### Why Fiber Is More Vulnerable Than Copper

- Fiber-optic cables are **physically fragile** — bending too sharply can snap a fiber permanently
- Two separate fibers mean **one can break while the other still works**
- Copper UTP cables are less likely to fail in a single-direction pattern

### Fully Functional vs. Unidirectional Fiber Link

| State | Both Fibers OK? | Interface Status | Traffic Flow |
| --- | --- | --- | --- |
| Normal | ✅ Yes | up/up | Bidirectional |
| Detected failure | ❌ No | **down/down** | None |
| Unidirectional link | ⚠️ One fiber broken (undetected) | **up/up** | One direction only |

---

## 2. The Problem: How a Unidirectional Link Causes a Loop

### Normal STP Behavior (No Failure)

- Root bridge sends BPDUs → forwarded by other switches out of **designated ports**
- SW3 G0/1 is a **non-designated blocking port** because it receives superior BPDUs from SW2
- SW3 correctly keeps G0/1 in blocking state → no loop

### With a Unidirectional Link (SW2 → SW3 direction broken)

1. SW2's BPDUs can no longer reach SW3
2. SW3's **Max Age timer** counts down to 0
3. SW3 G0/1 assumes the loop no longer exists → transitions to **Designated/Forwarding**
4. SW2 receives SW3's inferior BPDUs and **ignores them** — SW2 G0/1 stays Forwarding
5. **Both SW2 G0/1 and SW3 G0/1 are now Forwarding** → **loop created**
6. Broadcast frames loop continuously: SW1 → SW3 → SW2 → SW1...

> The unidirectional nature means SW2's traffic can still loop through SW3, even though SW3 can't receive SW2's BPDUs directly.
> 

---

## 3. Loop Guard: The Solution

### What It Does

- When a **Loop Guard-enabled port** stops receiving BPDUs and the Max Age timer reaches 0:
    - **Without Loop Guard** → port becomes Designated → transitions to Forwarding → **loop**
    - **With Loop Guard** → port enters **"broken / loop inconsistent"** state → **blocked, no loop**
- The port remains **up/up** physically — only STP blocks it
- Recovery is **automatic** when BPDUs resume

### Port State When Loop Guard Triggers

| Display | Meaning |
| --- | --- |
| `BKN` | **Broken** — port is blocked |
| `LOOP_Inc` | **Loop Inconsistent** — disabled by Loop Guard |

---

## 4. Configuration

### Method 1 — Per-Port

```
SW(config-if)# spanning-tree guard loop
```

Enables Loop Guard on that specific interface only.

### Method 2 — Global Default (All Ports)

```
SW(config)# spanning-tree loopguard default
```

Enables Loop Guard on **all ports** by default.

### Disable on a Specific Port (After Global Enable)

```
SW(config-if)# spanning-tree guard none
```

---

## 5. Verification

```
SW# show spanning-tree interface g0/1 detail
```

Look for: `"Loop guard is enabled on the port"` or `"Loop guard is enabled on the port by default"`

```
SW# show spanning-tree
```

Look for: `BKN` in the status column and `LOOP_Inc` in the reason column when the port is blocked.

**Log messages:**

```
! When Loop Guard blocks a port:
%SPANTREE-2-LOOPGUARD_BLOCK: Loop guard blocking port GigabitEthernet0/1

! When the port automatically recovers:
%SPANTREE-2-LOOPGUARD_UNBLOCK: Loop guard unblocking port GigabitEthernet0/1
```

---

## 6. Recovery

- **Automatic** — no CLI commands needed
- When the physical issue is resolved and BPDUs resume arriving on the port, Loop Guard **automatically re-enables** the port
- Port transitions from `BKN / LOOP_Inc` back to **Blocking** (non-designated) or **Forwarding** (root/designated) as appropriate

---

## 7. Loop Guard vs. Root Guard — Mutually Exclusive

### Why They Can't Coexist on the Same Port

- **Root Guard** → prevents a **Designated port** from accepting superior BPDUs and becoming a Root port
- **Loop Guard** → prevents a **Root or Non-Designated port** from becoming a Designated port when BPDUs stop
- They serve **opposite protective purposes** → configuring both on the same port makes no sense

### Conflict Resolution — More Specific Config Wins

| Scenario | Result |
| --- | --- |
| Loop Guard per-port → then Root Guard per-port | **Root Guard wins**; Loop Guard disabled |
| Root Guard per-port → then Loop Guard per-port | **Loop Guard wins**; Root Guard disabled |
| Loop Guard global default → Root Guard per-port | **Root Guard wins** on that port; Loop Guard still active elsewhere |

> **IOS rule:** The **more specific** (interface-level) command takes precedence over the **less specific** (global) command.
> 

---

## 8. Where to Apply Each Feature

| Port Role | Correct Feature | Reason |
| --- | --- | --- |
| **Non-Designated (Blocking)** | Loop Guard | Should receive BPDUs; protect against stopping |
| **Root Port** | Loop Guard | Should receive BPDUs; protect against stopping |
| **Designated Port (to untrusted switch)** | Root Guard | Shouldn't receive superior BPDUs |
| **Designated Port (to end host)** | PortFast + BPDU Guard | Shouldn't receive BPDUs at all |

---

## 9. Full STP Toolkit Summary

| Feature | Protects Against | Trigger | Port State | Recovery |
| --- | --- | --- | --- | --- |
| **PortFast** | 30s delay for end hosts | Port enabled | Immediate Forwarding | N/A |
| **BPDU Guard** | Unauthorized switches on host ports | BPDU received | `err-disabled` | Manual or ErrDisable Recovery |
| **BPDU Filter** | Unnecessary BPDUs to hosts | N/A | No BPDUs sent | N/A |
| **Root Guard** | Unauthorized root bridge takeover | Superior BPDU received | `BKN / ROOT_Inc` | **Automatic** |
| **Loop Guard** | Loops from unidirectional links | BPDUs stop arriving | `BKN / LOOP_Inc` | **Automatic** |

---

## 10. Command Summary

| Command | Mode | Purpose |
| --- | --- | --- |
| `spanning-tree guard loop` | Interface | Enable Loop Guard on this port |
| `spanning-tree loopguard default` | Global | Enable Loop Guard on all ports |
| `spanning-tree guard none` | Interface | Disable Loop Guard on a specific port |
| `spanning-tree guard root` | Interface | Enable Root Guard (replaces Loop Guard if present) |
| `show spanning-tree interface <int> detail` | Exec | Verify Loop Guard status |
| `show spanning-tree` | Exec | View BKN / LOOP_Inc port states |

---

# Day 22 (Rapid Spanning Tree Protocol)

## 🎯 Key Takeaways

- **RSTP (802.1w)** is the industry-standard fast version of STP; **Rapid PVST+** is Cisco's version (RSTP + one instance per VLAN, like PVST+ was to classic STP).
- RSTP converges in **seconds**, not the **up to 50 seconds** classic STP (802.1D) needs.
- Root bridge, root port, and designated port election rules are **unchanged** from classic STP.
- Port states are simplified to **3** (Discarding, Learning, Forwarding) — listening is removed, and blocking/disabled merge into "discarding."
- The old **non-designated** role splits into two new roles: **Alternate** (backup for a root port) and **Backup** (backup for a designated port, hub scenarios only).
- Classic STP's optional features **UplinkFast** and **BackboneFast** are now **built into RSTP by default** — no configuration needed.
- Cisco's CLI still displays "blocking" for discarding ports, and labels the protocol "rstp" even when running Rapid PVST+.

---

## 1. Comparing STP Versions

| Standard (IEEE) | Cisco Version | VLAN Instances | Load Balancing? | Speed |
| --- | --- | --- | --- | --- |
| **802.1D** – classic STP | **PVST+** | PVST+: 1 per VLAN | STP: ❌ / PVST+: ✅ | Slow (up to 50s) |
| **802.1w** – RSTP | **Rapid PVST+** | Rapid PVST+: 1 per VLAN | RSTP: ❌ / Rapid PVST+: ✅ | Fast (seconds) |
| **802.1s** – MSTP | *(none — Cisco runs the standard)* | Groups of VLANs per instance | ✅ | Fast |

**Details:**

- **802.1D (classic STP):** original standard (created 1985, published 1990). One shared instance for all VLANs → **no load balancing possible**.
- **PVST+:** Cisco's upgrade to 802.1D. Runs a **separate STP instance per VLAN**, enabling load balancing by blocking different ports per VLAN. (Older plain "PVST" only supported ISL trunking — obsolete now that dot1q is standard.)
- **802.1w (RSTP):** much faster convergence than 802.1D, but still **one shared instance** for all VLANs → no load balancing.
- **Rapid PVST+:** Cisco's upgrade to 802.1w — combines RSTP's speed with **per-VLAN instances** for load balancing.
- **802.1s (MSTP):** uses RSTP mechanics but **groups multiple VLANs into instances** (e.g., VLANs 1–100 → instance 1, VLANs 101–200 → instance 2). Easier to manage at scale than configuring root bridges per VLAN. Cisco does **not** have a proprietary version — it just runs standard MSTP.

**Practical guidance:**

- Large networks with many VLANs → **MSTP** is best (less configuration overhead).
- Small/medium networks → **Rapid PVST+** is most common and is what's covered on the CCNA exam topics.

---

## 2. RSTP Introduction

- RSTP is **not timer-based** like 802.1D (which relied on long timers to safely avoid loops).
- Core mechanism: a **bridge-to-bridge handshake** that lets ports move directly to forwarding when safe, instead of waiting through long timer-based states.
- **Note:** "RSTP" and "Rapid PVST+" are used interchangeably in this context — Rapid PVST+ = RSTP + per-VLAN instances.

---

## 3. What Stays the Same (STP vs. RSTP Similarities)

- Same purpose: blocking ports to prevent Layer 2 loops.
- **Root bridge election:** lowest Bridge ID wins — unchanged.
- **Root port election:** lowest root cost wins (tie-breakers: neighbor bridge ID, then neighbor port ID) — unchanged.
- **Designated port election:** lowest root cost on the segment wins; ties broken by lowest bridge ID — unchanged.
- Cisco describes RSTP as an **"evolution," not a "revolution"** of STP.

---

## 4. What's Different in RSTP

### Port Costs (Expanded for Faster Speeds)

| Speed | Classic STP Cost | RSTP Cost |
| --- | --- | --- |
| 10 Mbps | 100 | 2,000,000 |
| 100 Mbps | 19 | 200,000 |
| 1 Gbps | 4 | 20,000 |
| 10 Gbps | 2 | 2,000 |
| 100 Gbps | 1 | 200 |
| 1 Tbps | 1 | 20 |
| 10 Tbps | 1 | 2 |

### Port States (Simplified from 5 → 3)

- **Discarding** — merges classic STP's *Blocking* and *Disabled* states (listening state is eliminated entirely).
    - Administratively shut down → discarding (was "disabled").
    - Blocking traffic to prevent loops → discarding (was "blocking").
- **Learning**
- **Forwarding**

### Port Roles (Root & Designated Unchanged; Non-Designated Splits in Two)

- **Root port** — same as classic STP: lowest root cost path to the root bridge. Root bridge itself has no root port.
- **Designated port** — same as classic STP: the port on a segment sending the best BPDU; only one per segment.
- **Alternate port** *(new)* — a discarding port that receives a **superior BPDU from a different switch**. Functions as backup for the **root port**; if the root port fails, the alternate port moves straight to forwarding with no transitional delay.
- **Backup port** *(new)* — a discarding port that receives a superior BPDU from **another interface on the same switch** (only occurs via a hub connecting two interfaces to the same collision domain). Functions as backup for the **designated port**. Between two same-switch interfaces, the one with the lower port ID becomes designated; the other becomes backup.

---

## 5. Built-In Legacy Features

RSTP automatically incorporates two optional classic-STP features — **no configuration required**, and they run by default:

- **UplinkFast:** allows an alternate port to instantly become the root port if the current root port fails (previously a manually configured STP feature).
- **BackboneFast:** allows a switch to expire the max age timer early and rapidly accept a superior BPDU from another switch, instead of waiting through normal timer delays.

> Not deeply tested on the CCNA — just know their **names** and that their **functionality is built into RSTP**.
> 

---

## 6. CLI Notes (Rapid PVST+ on Cisco Switches)

- Three STP modes available: `MST`, `PVST`, `Rapid-PVST` — **rapid-PVST is the default** on modern Cisco switches.
- To set explicitly: `spanning-tree mode rapid-pvst`
- Verify with: `show spanning-tree`
    - Output shows **"protocol rstp"** (not "ieee") even though it's actually running Rapid PVST+.
    - Alternate/Backup ports still display status as **"BLK" (blocking)** in the CLI, even though the RSTP term for this state is **"discarding."**
- **Interoperability:** Rapid STP is backward-compatible with classic STP. A port connecting a Rapid STP switch to a classic STP switch will fall back to operating in **classic STP mode** (same legacy timers and blocking behavior) on that link.

---

## 7. Practice Scenario Recap (Root Bridge & Port Roles)

Example topology: SW1–SW4, with a hub between SW2 and SW3.

- **Root bridge** = switch with the lowest MAC address (when priorities are tied).
- Each switch's port toward the root = **root port** (lowest root cost; ties broken by neighbor bridge ID, then neighbor port ID).
- On any segment, the lower-cost switch's port = **designated**; among two ports on the same switch in a collision domain, lowest port ID = designated, the other = **backup**.
- A port receiving a superior BPDU from a *different* switch = **alternate**; from the *same* switch = **backup**.
- Hubs do **not** participate in STP — they simply flood frames, which is what creates backup-port scenarios.

---

# Day 23 (EtherChannel)

## 🎯 Key Takeaways

- **EtherChannel** groups multiple physical interfaces into one logical interface, solving the problem of **STP blocking redundant links** between switches (only one link would otherwise forward).
- It gives both **increased bandwidth** (links combine) and **redundancy** (backup links activate instantly on failure) without causing Layer 2 loops, because STP treats the whole group as a single interface.
- Load balancing happens per **flow** (a communication between two nodes) — all frames in the same flow use the same physical interface, preventing out-of-order delivery.
- The interface-selection calculation can use **source/destination MAC**, **source/destination IP**, or (on some switches) **Layer 4 port numbers**.
- Three configuration methods: **PAgP** (Cisco proprietary), **LACP** (industry standard, 802.3ad, exam-preferred), and **static** (no negotiation, generally avoided).
- Up to **8 active interfaces** per EtherChannel (LACP allows 16 total — 8 active + 8 standby).
- Confusingly, Cisco uses **three different keywords** for the same concept: `etherchannel` (verify), `port-channel` (configure load balancing / virtual interface name), and `channel-group` (assign interfaces).

---

## 1. Why EtherChannel Is Needed

- Scenario: an access switch (**ASW1**) connects to a distribution switch (**DSW1**). Adding more physical links between them does **not** increase usable bandwidth.
- Reason: **Spanning Tree Protocol** blocks all but one link between the same two switches to prevent Layer 2 loops and broadcast storms — extra links stay in **blocking** (orange port lights), only activating if the active link fails.
- Result: bandwidth is wasted, and congestion persists despite adding more cables.
- **Access switch (ASW):** connects to end hosts (PCs, servers).
- **Distribution switch (DSW):** connects to access switches.
- **Oversubscription:** when total bandwidth of end-host-facing interfaces exceeds the bandwidth of the uplink to the distribution switch. Some oversubscription is normal/acceptable; too much causes congestion.

---

## 2. EtherChannel Introduction

- **EtherChannel** bundles multiple physical interfaces so they **behave as a single logical interface** — shown in diagrams as a circle around the grouped interfaces.
- STP treats the whole bundle as **one interface**, so all member links can forward simultaneously without forming a loop.
- **Broadcast handling example:** even though there are 4 physical links between ASW1 and DSW1, a broadcast frame is sent only **once** across the bundle (not duplicated per physical link), and the receiving switch won't forward it back out of the same logical interface — no loop is created.
- Analogous to how **VLANs** create a virtual separation despite shared physical hardware — EtherChannel creates a virtual combination of physical interfaces, effectively forming one faster logical link (e.g., four 1 Gbps links → one virtual 4 Gbps link).
- **Other names:** Port Channel, **LAG** (Link Aggregation Group).

---

## 3. EtherChannel Load Balancing

### Concept of a "Flow"

- A **flow** = a communication session between two specific nodes (e.g., PC1 ↔ SRV1).
- An algorithm assigns each flow to **one specific physical interface** within the bundle; all frames in that flow use that same interface to avoid out-of-order delivery (which can break some applications).
- Different flows (e.g., PC1 ↔ PR1, PC2 ↔ PR1) may be assigned to different physical interfaces, achieving load distribution across the bundle.

### Inputs Used for the Load-Balancing Calculation

- **Source MAC address**
- **Destination MAC address**
- **Source AND destination MAC address**
- **Source IP address**
- **Destination IP address**
- **Source AND destination IP address**
- Some switches also support **Layer 4 TCP/UDP port numbers** (covered in a later lesson).
- Available options depend on the **switch model** — some support only MAC-based, some MAC or IP, some all methods.
- **Non-IP traffic** automatically falls back to MAC-address-based balancing, since there's no IP address to use.

### Verification & Configuration Commands

- **`show etherchannel load-balance`** — displays the current load-balancing method (e.g., default may be source/destination IP).
- **`port-channel load-balance [method]`** (entered in global config mode) — changes the load-balancing method.
- ⚠️ Note the inconsistent terminology: you **view** with `etherchannel`, but **configure** with `port-channel`.

---

## 4. EtherChannel Configuration Protocols

| Protocol | Type | Standard | Modes | Notes |
| --- | --- | --- | --- | --- |
| **PAgP** | Cisco proprietary | — | Desirable / Auto | Only works between Cisco switches |
| **LACP** | Industry standard | IEEE 802.3ad | Active / Passive | Works across vendors; preferred and the only one listed in the exam topics |
| **Static** | Manual, no protocol | — | (none) | No dynamic negotiation; generally avoided |

### PAgP (Port Aggregation Protocol)

- Cisco proprietary; dynamically negotiates EtherChannel formation/maintenance (similar role to DTP for trunks).
- **Modes:**
    - **Desirable** — actively attempts to form an EtherChannel.
    - **Auto** — only forms an EtherChannel if the other side is set to Desirable.
- **Formation outcomes:**
    - Auto + Auto → ❌ no EtherChannel
    - Auto + Desirable → ✅ forms
    - Desirable + Desirable → ✅ forms

### LACP (Link Aggregation Control Protocol)

- Industry standard (IEEE 802.3ad); functionally similar to PAgP but vendor-independent — works with non-Cisco switches.
- **Modes:**
    - **Active** — actively attempts to form an EtherChannel (equivalent to PAgP's Desirable).
    - **Passive** — only forms an EtherChannel if the other side is Active (equivalent to PAgP's Auto).
- **Formation outcomes:**
    - Passive + Passive → ❌ no EtherChannel (interface/virtual port-channel is still created, but doesn't function as one)
    - Passive + Active → ✅ forms
    - Active + Active → ✅ forms

### Static EtherChannel

- No negotiation protocol — interfaces are manually configured to be in the EtherChannel.
- Generally **avoided** because the switch can't dynamically remove a problematic interface from the group.

### Interface Limits

- Up to **8 active interfaces** per EtherChannel.
- **LACP** allows up to **16** interfaces total — 8 active + 8 in **standby**, ready to activate if an active interface fails.

---

## 5. Configuration Commands

### General Approach

- Use **`interface range`** to configure all member interfaces at once — ensures consistent configuration across all members (configurations on member interfaces must match).

### Core Command

```
channel-group [number] mode [desirable | auto | active | passive | on]
```

- **`channel-group`** — the keyword used to actually build the EtherChannel and assign interfaces to it.
- The resulting virtual interface is named **`port-channel [number]`** (visible in `show ip interface brief`).
- **PAgP modes:** `desirable`, `auto`
- **LACP modes:** `active`, `passive`
- **Static:** `on` (one additional mode not covered in detail here)

### Important Notes on Channel-Group Numbers

- The **channel-group number must match between member interfaces on the same local switch**.
- It does **NOT** need to match the channel-group number used on the neighboring switch (e.g., `channel-group 1` on ASW1 can form an EtherChannel with `channel-group 2` on DSW1) — the number is only a local identifier, useful when a switch has multiple EtherChannels.

---

## 6. Terminology Cheat Sheet (Easy to Mix Up)

- **`channel-group`** → command to assign physical interfaces into the EtherChannel.
- **`port-channel`** → name of the resulting virtual interface; also the keyword for configuring load-balancing (`port-channel load-balance`).
- **`etherchannel`** → keyword used to **verify** settings (`show etherchannel load-balance`).
- **LAG** → alternate generic name for an EtherChannel (Link Aggregation Group).

---

# Day 24 (Dynamic Routing)

## 🎯 Key Takeaways

- **Dynamic routing** lets routers automatically discover, share, and update routes via a routing protocol — unlike **static routing**, which requires manually configuring every route with `ip route`.
- Routers exchange routing info by forming **adjacencies** (neighbor relationships) and choosing the best route based on the lowest **metric**.
- Dynamic routing protocols split into **IGP** (within one autonomous system) and **EGP** (between autonomous systems); IGPs further split into **distance vector** (RIP, EIGRP) and **link state** (OSPF, IS-IS) algorithm types; **BGP** is the only EGP in modern use (path vector algorithm).
- **Distance vector** = "routing by rumor" (routers only know what neighbors tell them); **link state** = routers build a full network map and calculate routes independently (more resource-intensive, but reacts to changes faster).
- When multiple equal-metric routes to the same destination exist, all are added to the routing table and traffic load-balances across them — this is **ECMP (Equal Cost MultiPath)**, and it works with both dynamic protocols and static routes.
- **Administrative Distance (AD)** is a separate value from metric, used to choose between routes learned via *different* sources/protocols (covered in more depth later — OSPF AD = 110, static routes AD = 1).

---

## 1. Static vs. Dynamic Routing

- **Static routing:** routes manually configured with `ip route`; the router has no awareness if a path fails — it keeps forwarding traffic toward a dead end.
- **Dynamic routing:** a routing protocol is enabled, and routers automatically:
    - Learn about new networks added anywhere in the topology.
    - Remove failed/invalid routes (or replace them with the next-best backup route if one exists).
    - Switch to the next-best path automatically when the primary path goes down.
- **Demonstrated behavior:** when a router learns two valid paths to the same destination (e.g., via R2 and via R3), it keeps only the **lower-metric** route in the table — if that path fails, the table automatically updates to the surviving (next-best) path.

---

## 2. Network Routes vs. Host Routes

- **Network route:** a route to an entire network/subnet — any route with a mask **less than /32** (e.g., `192.168.4.0/24`).
- **Host route:** a route to one specific address — uses a **/32** mask (e.g., the auto-generated routes to a router's own interface IPs).
- To configure a static host route: `ip route [host-address] 255.255.255.255 [next-hop]`.

---

## 3. Dynamic Routing Protocol Fundamentals

- Routers using a dynamic routing protocol **advertise** their connected and learned routes to neighbors.
- This forms **adjacencies** (a.k.a. **neighbor relationships** / **neighborships**) with directly connected routers running the same protocol.
- When multiple routes to the same destination are learned, the router compares their **metric** — the **lower metric wins** and is installed in the routing table (conceptually similar to **STP root cost**, where the lowest cost wins).

---

## 4. Categories of Dynamic Routing Protocols

### By Scope

- **IGP (Interior Gateway Protocol):** shares routes **within** a single autonomous system (AS) — e.g., within one company's network.
- **EGP (Exterior Gateway Protocol):** shares routes **between** different autonomous systems (e.g., between two companies, or between an ISP and a customer).

### By Algorithm Type

| Category | Algorithm Type | Protocols |
| --- | --- | --- |
| EGP | **Path Vector** | **BGP** (Border Gateway Protocol) — the only EGP in modern use |
| IGP | **Distance Vector** | **RIP**, **EIGRP** |
| IGP | **Link State** | **OSPF**, **IS-IS** |
- **BGP** and **IS-IS** are mentioned only briefly for CCNA purposes — deep knowledge isn't required.
- **OSPF** is the only dynamic routing protocol explicitly listed in the official CCNA exam topics, but candidates still need a working understanding of RIP, EIGRP, and basic IGP/EGP concepts for comparison purposes.

---

## 5. Distance Vector Routing Protocols (RIP, EIGRP)

- Predate link state protocols (invented in the early 1980s). Early examples: **RIP** and Cisco's proprietary **IGRP** (later evolved into **EIGRP**).
- **Mechanism:** each router sends its directly connected neighbors a list of **known destination networks** plus its **metric** to reach each one.
- Nicknamed **"routing by rumor"** — a router only knows what its immediate neighbor tells it; it has no visibility into the broader network topology.
- **Name origin:**
    - **Distance** = the metric value.
    - **Vector** = the direction (next-hop router) to send traffic.
- **Example flow:** R4 advertises "reach 192.168.4.0/24 via me, metric 1" to R2. R2 doesn't independently know the network — it just adds the route and re-advertises to R1 with an updated (incremented) metric. R1 repeats this to R3, and so on down the chain.

---

## 6. Link State Routing Protocols (OSPF, IS-IS)

- **Mechanism:** every router advertises information about its own interfaces and connected networks; these advertisements propagate until **every router builds an identical "connectivity map"** of the entire network.
- Each router then **independently calculates** the best route to every destination using that shared map.
- **Trade-offs vs. distance vector:**
    - Uses **more CPU and memory** (more information is shared and stored).
    - **Reacts faster** to topology changes.
- The two link state protocols in use today: **OSPF** (Open Shortest Path First) and **IS-IS** (Intermediate System to Intermediate System). OSPF receives in-depth coverage in this course; IS-IS does not (relevant to CCNP Service Provider track instead).

---

## 7. Metric

- The **metric** is a protocol-specific value representing how "far" a destination is — used to pick the best route when multiple options exist.
- **Lower metric = better route** (same logic as STP's root cost).
- Each routing protocol calculates its metric differently (to be covered protocol-by-protocol in later videos). Example shown: **RIP** uses the simplest metric, **hop count**.
- **Demonstration:** R1 has two paths to `192.168.4.0/24` — via R2 (Gigabit Ethernet) and via R3 (FastEthernet, higher cost). Only the lower-metric route (via R2) was installed in the routing table.

### Equal Cost MultiPath (ECMP)

- If a router learns **two or more routes to the exact same destination network (same address + same prefix length) via the same routing protocol with identical metrics**, **all of them** are installed in the routing table.
- Traffic is then **load-balanced** across all the equal-cost routes — this behavior is called **ECMP (Equal Cost MultiPath)**.
- **Example:** when both R2 and R3 paths became Gigabit Ethernet (equal cost), R1's routing table showed both routes to `192.168.4.0/24`, both via OSPF (code `O`), both with metric `3`.
- **ECMP also works with static routes:** two static routes to the same destination via different next hops will both be installed and load-balanced. Static routes always show a metric of **0** (the metric concept doesn't really apply to them).

---

## 8. Administrative Distance (AD) — Preview

- A separate value (shown to the **left** of the square brackets in routing table output, vs. metric on the right) used to choose between routes learned from **different sources/protocols** for the same destination.
- Mentioned values so far (full explanation comes later):
    - **OSPF** → AD of **110**
    - **Static routes** → AD of **1**
- Lower AD is preferred when comparing routes from different protocols/sources.

---

# Day 25 (RIP & EIGRP)

## 🎯 Key Takeaways

- **RIP** (Routing Information Protocol) and **EIGRP** (Enhanced Interior Gateway Routing Protocol) aren't explicitly on the exam topics list, but related questions can still appear — Cisco's exam topics are only "general guidelines."
- **RIP** is a simple, industry-standard **distance vector** IGP using **hop count** (max 15) as its metric — bandwidth is irrelevant, so it's rarely used in real/large networks.
- **RIPv2** (always prefer this over v1) supports **VLSM/CIDR** and multicasts updates to **224.0.0.9**; **RIPv1** is classful-only and broadcasts to 255.255.255.255.
- **EIGRP** is an advanced/hybrid distance vector protocol, mostly **Cisco-only** in practice, with no hop-count limit, faster convergence than RIP, multicast address **224.0.0.10**, and metric based on **bandwidth + delay**.
- EIGRP is the **only IGP capable of unequal-cost load balancing** (proportional to path bandwidth), unlike RIP's equal-cost-only ECMP.
- The `network` command for RIP/EIGRP **does not directly state what to advertise** — it only tells the router which interfaces to activate the protocol on (based on a classful range for RIP, or a wildcard mask range for EIGRP); the router then advertises the actual prefix configured on the matching interface(s).
- **Wildcard masks** (used by EIGRP and later OSPF) are inverted subnet masks: a **0 bit means "must match,"** a **1 bit means "don't care."**
- **Router ID priority** (EIGRP, and later OSPF): **manual configuration** → **highest loopback interface IP** → **highest physical interface IP**.
- **Administrative Distance (AD)** is only used to choose between routes to the **same destination** learned from **different routing protocols** (not used when comparing same-protocol routes, and not relevant when destinations differ).

---

## 1. RIP (Routing Information Protocol)

### Basic Characteristics

- **Industry standard** (not Cisco proprietary).
- **Distance vector** interior gateway protocol ("routing by rumor" logic).
- **Metric = hop count** — each router in the path counts as 1 hop, regardless of bandwidth (a 10 Gbps link and a 10 Mbps link both count as 1 hop).
- **Maximum hop count = 15** — anything beyond is considered unreachable and excluded from the routing table. This limits RIP to small networks/labs.
- Rarely used in real production networks today; mainly useful for **small networks or lab/learning environments** due to its simplicity.
- **Versions:** RIPv1 and RIPv2 (for IPv4), plus **RIPng** (for IPv6, not covered in this course).
- **Message types:**
    - **Request** — asks neighbor routers to send their routing table.
    - **Response** — sends the local router's routing table to neighbors.
- Routers share their full routing table **every 30 seconds** by default — can create excessive traffic in larger networks.

### RIPv1 vs. RIPv2

| Feature | RIPv1 | RIPv2 |
| --- | --- | --- |
| Addressing | **Classful only** (no subnet mask sent) | Supports **VLSM/CIDR** (subnet mask included in advertisements) |
| Delivery | **Broadcast** to 255.255.255.255 | **Multicast** to 224.0.0.9 |
| Recommended? | ❌ Avoid | ✅ Always prefer |
- **RIPv1 classful conversion examples:**
    - `10.1.1.0/24` → advertised as `10.0.0.0` (Class A, /8)
    - `172.16.192.0/18` → advertised as `172.16.0.0` (Class B, /16)
    - `192.168.1.4/30` → advertised as `192.168.1.0` (Class C, /24)
- **Broadcast vs. multicast (brief):** broadcast reaches **all** devices on the local network; multicast reaches only devices that have **joined that specific multicast group**. (Deeper multicast detail is beyond CCNA scope.)

### RIP Configuration

```
router rip
version 2
no auto-summary
network 10.0.0.0
passive-interface g2/0
default-information originate
```

- **`router rip`** — enters RIP configuration mode (prompt changes to `config-router`).
- **`version 2`** — forces RIPv2 (always recommended).
- **`no auto-summary`** — disables automatic conversion of advertised networks into classful networks (auto-summary is **on by default**).
- **`network [address]`** — classful command; it does **not** specify what to advertise. It tells the router to activate RIP on any interface whose IP falls within the resulting classful range, then the router advertises that interface's **actual configured prefix**.
    - Example: `network 10.0.0.0` is treated as `10.0.0.0/8` → matches any interface starting with `10.` → RIP activates on those interfaces, which then advertise their real prefixes (e.g., `10.0.12.0/30`).
- **`passive-interface [interface]`** (entered in RIP config mode) — stops RIP advertisements from going out that interface, while the router still advertises that interface's prefix to its other RIP neighbors. Recommended on any interface with no RIP neighbors (e.g., a LAN-facing interface). EIGRP and OSPF support the same command.
- **`default-information originate`** (RIP config mode) — advertises the local router's configured default route (`0.0.0.0/0`) to RIP neighbors, who then propagate it onward. OSPF has the same command.

### `show ip protocols` (RIP) — Key Fields

- **Protocol in use** (RIP) and **version** (2, if configured).
- **Automatic network summarization:** should show "not in effect" if `no auto-summary` was used.
- **Maximum paths:** default is **4** for ECMP load balancing (routes with equal metric); configurable via `maximum-paths [1–32]` in RIP config mode — same command works for EIGRP and OSPF.
- **Routing for networks:** lists the ranges entered via the `network` command (not the actual advertised prefixes).
- **Passive interfaces:** lists any configured passive interfaces.
- **Routing information sources:** lists RIP neighbors (by IP).
- **Distance (Administrative Distance):** default is **120** for RIP; changeable via `distance [1–255]` in RIP config mode (same command for EIGRP/OSPF). Example: lowering RIP's AD to 85 would make RIP routes preferred over EIGRP routes (AD 90) for the same destination.

### Default Route / ECMP Example

- After configuring a default route on R1 and using `default-information originate`, the route propagates to other RIP routers.
- If a downstream router learns the default route via two equal-hop-count paths (even if one link is slower, since RIP ignores bandwidth), it will **load-balance** traffic across both — a real-world illustration of RIP's metric blind spot.

---

## 2. EIGRP (Enhanced Interior Gateway Routing Protocol)

### Basic Characteristics

- Improved successor to Cisco's older **IGRP**.
- Technically published openly by Cisco, but key parts remain proprietary — **practically still a Cisco-only protocol**.
- Classified as an **"advanced" / "hybrid" distance vector** protocol — much faster convergence than RIP.
- **No hop-count limit** → suitable for large networks (unlike RIP).
- Uses multicast address **224.0.0.10** for its messages.
- **Unique feature:** EIGRP is the **only IGP capable of unequal-cost load balancing** — it can distribute traffic across multiple unequal-cost paths in proportion to their bandwidth (more traffic on lower-metric/faster paths). By default it still performs **ECMP over up to 4 equal-cost paths**, like RIP.
- Less widely used than OSPF because of its largely Cisco-only nature — this is why OSPF is the CCNA's primary dynamic routing focus.

### Multicast Address Comparison

| Protocol | Delivery Method | Address |
| --- | --- | --- |
| RIPv1 | Broadcast | 255.255.255.255 |
| RIPv2 | Multicast | 224.0.0.9 |
| EIGRP | Multicast | 224.0.0.10 |

### EIGRP Configuration

```
router eigrp 1
no auto-summary
passive-interface g2/0
network 10.0.0.0
network 172.16.1.0 0.0.0.15
eigrp router-id 1.1.1.1
```

- **`router eigrp [AS-number]`** — enters EIGRP config mode; the **AS (autonomous system) number must match** between routers, or they won't form an adjacency.
- **`no auto-summary`** — same purpose as in RIP (disable classful summarization); may already be disabled by default depending on platform/IOS version — verify either way.
- **`passive-interface`** — same function and command as RIP.
- **`network`** command — same underlying logic as RIP's (activates EIGRP on matching interfaces; doesn't dictate advertised prefix), but EIGRP supports an optional **wildcard mask** for more precise interface matching instead of relying purely on classful boundaries.
- **`eigrp router-id [id]`** — manually sets the router ID (highest priority method).
- You can run RIP and EIGRP simultaneously on a router, but it's wasteful — normally only **one IGP** runs per router.

### Wildcard Masks

- A **wildcard mask** is an **inverted subnet mask**: every `1` bit in the subnet mask becomes `0`, and every `0` bit becomes `1`.
- **Conversion examples:**
    - `/24` (255.255.255.0) → **0.0.0.255**
    - `/16` (255.255.0.0) → **0.0.255.255**
    - `/8` (255.0.0.0) → **0.255.255.255**
    - `/28` (255.255.255.240) → **0.0.0.15**
    - `/25` → **0.0.0.127**
    - `/14` → **0.3.255.255**
    - `/19` → **0.0.31.255**
    - `/21` → **0.0.7.255**
    - `/32` (255.255.255.255) → **0.0.0.0**
- **Shortcut:** subtract each octet of the subnet mask from 255 to get the wildcard mask octet.
- **Matching logic:** a **`0`** in the wildcard mask means that bit position **must match** between the interface IP and the `network` command address; a **`1`** means it **doesn't need to match**.
    - Example: `network 172.16.1.0 0.0.0.15` requires the first 28 bits to match — interface IP `172.16.1.14` matches, so EIGRP activates on that interface.
    - Counter-example: `network 172.16.1.0 0.0.0.7` requires the first 29 bits to match — `172.16.1.14` does **not** match this stricter range, so EIGRP would **not** activate.
- In practice, admins usually just use a wildcard mask matching the interface's actual prefix length, or a `/32` wildcard (`0.0.0.0`) to target one exact IP. OSPF also uses wildcard masks (covered later).

### `show ip protocols` (EIGRP) — Key Fields

- **Routing protocol: EIGRP [AS number]**.
- **K-values:** by default only **K1** (bandwidth) and **K3** (delay) are active (set to 1); K2, K4, K5 are 0 and unused unless reconfigured.
- **Metric calculation:** uses the **bandwidth of the slowest link** in the path **plus the sum of delay values** of all links in the path.
- **Router ID** — shown next; see priority rules below.
- **Automatic summarization** and **maximum paths (default 4)** fields work the same as in RIP.
- **Routing for networks** — lists the configured `network` command ranges.
- **Passive interfaces** and **neighbors** listed same as RIP.
- **Administrative Distance:** EIGRP has **two AD values** — **90** for internal EIGRP routes, **170** for external (redistributed) routes. (External routes / redistribution is a CCNP-level topic.)

### EIGRP Router ID — Priority Order

1. **Manually configured** router ID (`eigrp router-id`) — highest priority.
2. **Highest IP address on a loopback interface**, if no manual ID is set.
3. **Highest IP address on a physical interface**, if no loopback interfaces exist.
- Note: the router ID is just a 32-bit number formatted like an IP address — it isn't necessarily a real, reachable IP.

### `show ip route` (EIGRP) — Notes

- EIGRP routes are marked with the code **`D`** (not `E`).
- EIGRP metric values (e.g., 3072, 3328, 28416) are **much larger and harder to interpret** than RIP or OSPF metrics — considered a minor downside of EIGRP, especially as networks scale up.

---

## 3. Quiz Review

**Q1:** R1 and R2 use RIP. To advertise R1's default route to R2, which command is correct?

- **Answer: `default-information originate`, entered in RIP config mode on R1.** (`network` activates RIP on an interface, not advertise a default route; configuring a static route on R2 doesn't make R1 advertise anything; the command must run on R1, the originator, not R2.)

**Q2:** R1's G1/0 = 172.20.20.17, G2/0 = 172.26.20.12. Which EIGRP `network` command activates EIGRP on both interfaces?

- **Answer: `network 128.0.0.0 127.255.255.255`.** Only the first bit of the wildcard mask is 0 (must match); the first bit of both IP addresses and the network command is `1` in binary, so both interfaces match.

**Q3:** What is the correct priority order for determining the EIGRP router ID?

- **Answer: Manual configuration → highest loopback interface address → highest physical interface address.**

**Bonus (Boson ExSim):** In which situation does a router use **Administrative Distance (AD)** to select a route?

- **Answer: When multiple routes to the *same* destination network are received from *different* routing protocols.**
    - Different destinations → no comparison needed, all routes are installed regardless of protocol.
    - Same destination + same protocol → compare **metric**, not AD (since AD would be identical).
    - Same destination + different protocols → metrics aren't comparable across protocols, so **AD** is used to break the tie.

---

# Day 26 (OSPF pt.1)

Key Takeaways

- OSPF is a Link State IGP using Dijkstra's algorithm to calculate best routes
- All routers in the same area share an identical LSDB
- Area 0 is the backbone — all other areas must connect to it
- OSPF process ID is locally significant and does not need to match between routers

<aside>
🧠

**OSPF** = Open Shortest Path First

- Link State IGP
- **OSPFv2** = IPv4 (exam focus)

**Dijkstra's Algorithm** (SPF) = used to calculate best routes — know this name!

**LSA** (Link State Advertisement) = packet containing route/link info

**LSDB** (Link State Database) = collection of all LSAs — **identical on all routers in same area**

</aside>

- LSAs flooded to all OSPF neighbors
- LSA aging timer = **30 min** (re-flooded after)

3-Step OSPF Process

1. Become **neighbors** with routers on same segment
2. **Exchange LSAs** with neighbors
3. Each router runs **SPF algorithm** → inserts best routes into routing table

OSPF Areas

**OSPF Areas** = logical groupings of routers/links sharing the same LSDB

- **Area 0** = Backbone area — ALL other areas must connect to it
- Small networks: single area is fine
- Large networks: divide into multiple areas

**Why multi-area?** Single large area = slower SPF, more CPU/memory, more LSA flooding

OSPF Router Types

| Type | Definition |
| --- | --- |
| **Internal Router** | All interfaces in same area |
| **ABR** (Area Border Router) | Interfaces in 2+ areas; maintains separate LSDB per area |
| **Backbone Router** | Has at least one interface in Area 0 (includes ABRs) |
| **ASBR** (AS Boundary Router) | Connects OSPF to external network (e.g. Internet) |
- ABR best practice: connect to max 2 areas only
- **Intra-area route** = destination in same area
- **Interarea route** = destination in different area

OSPF Area Rules

1. Areas must be **contiguous** (not split up)
2. All areas must have an **ABR connected to Area 0**
3. Interfaces in the **same subnet must be in the same area** or they won't become neighbors

Basic OSPF Configuration

```
router ospf 1                        ← process ID (locally significant — doesn't need to match)
  network 10.0.12.0 0.0.3.255 area 0 ← wildcard mask; activates OSPF on matching interfaces
  passive-interface G2/0             ← stops hello msgs; still advertises subnet to neighbors
  default-information originate      ← advertises default route → makes router an ASBR
  router-id 1.1.1.1                  ← manual config (highest priority)
  maximum-paths 8                    ← default is 4 (ECMP only — no unequal cost like EIGRP)
  distance 85                        ← change AD (default = 110)
```

- **Process ID does not equal AS number** — different routers can have different process IDs and still neighbor
- After changing router ID: must use `clear ip ospf process` or reload to take effect

Router ID Priority

1. Manual config → 2. Highest loopback IP → 3. Highest physical interface IP

*Example: R1 has no loopback, highest physical IP is 172.16.1.14 → becomes router ID by default*

Key show commands

- `show ip protocols` → protocol, router ID, neighbors, passive ints, AD, max paths
- `show ip route` → OSPF routes marked with **O**
    - format: **[AD/metric]**
- **OSPF AD = 110**
- No unequal-cost load balancing (ECMP only, up to 4 paths default)
- *DEFAULT-INFORMATION ORIGINATE on R1 → R2/R3/R4 all learn default route via OSPF → R1 becomes ASBR*

---

# Day 27 (OSPF pt.2)

**OSPF Cost** = reference bandwidth ÷ interface bandwidth

- Default reference bandwidth = **100 Mbps**
- Any result less than 1 is converted to **1** (FastEthernet, GigE, 10GigE all = cost 1 by default)

| Interface | Bandwidth | Default Cost |
| --- | --- | --- |
| Ethernet | 10 Mbps | 10 |
| FastEthernet | 100 Mbps | 1 |
| GigabitEthernet | 1000 Mbps | 1 |
| Loopback | — | 1 (always) |

Fix this by changing reference bandwidth so faster links have different costs:

```
(ospf config) auto-cost reference-bandwidth [Mbps]  ← must match on ALL routers
(interface)   ip ospf cost [value]                  ← overrides auto-calc; recommended
(interface)   bandwidth [kilobits]                  ← changes cost calc only; NOT recommended
```

- *Ref BW = 100,000 → FastE cost = 1000 | GigE cost = 100*
- Route metric = **sum of outgoing interface costs** along the path
- *R1→R2→R4: cost 100+100+100 = 300 | R1→R2 loopback: 100+1 = 101*

**Verify:** `show ip ospf interface brief` — quick overview of all OSPF interface costs

**OSPF Neighbor States (in order):**

1. **Down** — OSPF activated; Hello sent with neighbor RID = 0.0.0.0; no neighbors known
2. **Init** — Hello received; but receiver's own RID not yet in the packet
3. **2-Way** — Both RIDs in each other's Hellos | DR/BDR election may occur here
4. **Exstart** — Exchange **DBD** packets to elect **Master** (higher RID) and **Slave**
5. **Exchange** — Exchange DBDs listing LSAs in their LSDB (summary only, not full LSAs)
6. **Loading** — Send **LSR** to request missing LSAs → neighbor replies with **LSU** → confirm with **LSAck**
7. **Full** — Identical LSDBs; Full OSPF adjacency formed

**OSPF 5 Message Types:**

| # | Type | Purpose |
| --- | --- | --- |
| 1 | **Hello** | Discover/maintain neighbors |
| 2 | **DBD** | Summary of LSDB contents |
| 3 | **LSR** | Request specific LSAs |
| 4 | **LSU** | Send requested LSAs |
| 5 | **LSAck** | Acknowledge receipt of LSAs |
- **Hello multicast address = 224.0.0.5** | OSPF IP protocol number = **89**
- **Default Hello timer = 10 sec | Dead timer = 40 sec** (Ethernet)
- Dead timer resets every time a Hello is received; hits 0 → neighbor removed

**Additional OSPF Configurations:**

Enable OSPF directly on interface:

```
(interface config) ip ospf [process-id] area [area-id]
```

Configure all interfaces passive by default, then selectively enable:

```
(ospf config) passive-interface default
(ospf config) no passive-interface [interface]
```

**Verify neighbors:** `show ip ospf neighbor`**Verify interface:** `show ip ospf interface [int]`

**Key distinctions to memorize:**

- **Exstart** = Master/Slave decided (higher RID = Master)
- **2-Way** = DR/BDR elected (in some network types)
- **Loading** = actual LSAs exchanged (LSR → LSU → LSAck)
- **Full** = complete adjacency; same LSDB on both routers

---

# Day 28 (OSPF pt.3)

**Loopback Interface** = virtual interface; always up/up unless manually shut down

- Provides a **stable, consistent IP** to identify/reach the router even if physical interfaces fail
- Best practice: configure on every router; IP becomes OSPF router ID if highest

**OSPF Network Types:**

| Type | Default On | DR/BDR? | Neighbor Discovery |
| --- | --- | --- | --- |
| **Broadcast** | Ethernet, FDDI | Yes | Dynamic (224.0.0.5) |
| **Point-to-Point** | PPP, HDLC (Serial) | No | Dynamic (224.0.0.5) |
| **Non-Broadcast** | Frame Relay, X.25 | Yes | Manual config |
- Hello/Dead timers: **Broadcast & P2P = 10s/40s** | Non-broadcast = 30s/120s
- Change network type: `(interface) ip ospf network [type]`

**DR/BDR Election (Broadcast only):**

- **DR** = Designated Router | **BDR** = Backup DR | **DROther** = everyone else
- Election order: **1. Highest OSPF interface priority** → **2. Highest Router ID**
- Default priority = **1** on all interfaces
- Priority **0** = router CANNOT be DR or BDR (ever)
- Election is **non-preemptive** — winner keeps role until OSPF resets or interface fails
- Change priority: `(interface) ip ospf priority [0-255]`
- When DR fails → BDR becomes DR → new election for BDR only
- DROthers form **Full adjacency only with DR and BDR** — stay at **2-way** with other DROthers
- DR/BDR multicast address = **224.0.0.6** | All routers = **224.0.0.5**

**Serial Interfaces (brief):**

- Default encapsulation: **HDLC** (Cisco proprietary cHDLC) | Can change to PPP
- One side = **DCE** (sets clock rate/speed) | Other side = **DTE**
- `clock rate [bps]` on DCE side | `encapsulation ppp` to change | must match both ends
- Identify DCE/DTE: `show controllers [interface]`
- Serial uses `clock rate` for speed — NOT the `speed` command like Ethernet

**OSPF Neighbor Requirements:**

| Requirement | Notes |
| --- | --- |
| **Same area** | Must match |
| **Same subnet** | Must be on same network |
| **OSPF process not shutdown** | `shutdown` in OSPF config disables it |
| **Unique Router IDs** | Duplicate RIDs = neighbor stays down |
| **Hello/Dead timers match** | Mismatch = neighbor drops |
| **Authentication match** | `ip ospf authentication-key` + `ip ospf authentication` |
| **IP MTU must match** | Mismatch = stuck in EXSTART state |
| **OSPF network type match** | Mismatch = Full state shown but routes missing |
- **Process ID does NOT need to match** between routers
- *MTU mismatch example: R2 set to 1400, R1 at default 1500 → stuck at EXSTART — fix with `no ip mtu`*

**OSPF LSA Types (know these 3):**

| Type | Name | Generated By |
| --- | --- | --- |
| **Type 1** | Router LSA | Every OSPF router — identifies itself + connected networks |
| **Type 2** | Network LSA | DR of each multi-access (broadcast) segment |
| **Type 5** | AS-External LSA | ASBR — advertises routes outside OSPF domain |
- View LSDB: `show ip ospf database` (same output on all routers in area)

---

# Day 29 (FHRP — First Hop Redundancy Protocol)

**FHRP** = protocol that provides a redundant default gateway for a subnet

- Problem: if the default gateway router fails, hosts still point to its IP → no Internet
- Solution: two routers share a **Virtual IP (VIP)** and **Virtual MAC** → hosts use VIP as gateway

**How FHRPs Work:**

- **Active router** = currently acting as default gateway | **Standby** = backup (terms vary by FHRP)
- Routers send multicast **Hello messages** to each other to maintain roles
- Active router replies to ARP requests using the **virtual MAC address**
- If active fails → standby takes over → sends **Gratuitous ARP** to update switch MAC tables
- **Gratuitous ARP** = ARP reply sent without a request; broadcast; updates switches
- Hosts do NOT need to update ARP tables — virtual IP/MAC stays the same
- **Non-preemptive by default** — recovered router becomes standby, not active again
- Enable **preemption** to allow recovered router to reclaim active role automatically

**FHRP Comparison Chart:**

| Feature | HSRP | VRRP | GLBP |
| --- | --- | --- | --- |
| Standard | Cisco proprietary | Open standard | Cisco proprietary |
| Active role name | Active | Master | AVG / AVF |
| Backup role name | Standby | Backup | — |
| Multicast (v1) | 224.0.0.2 | 224.0.0.18 | 224.0.0.102 |
| Multicast (v2) | 224.0.0.102 | — | 224.0.0.102 |
| Load balance single subnet? | No | No | Yes |

**Virtual MAC Addresses — Memorize these:**

| Protocol | Virtual MAC Format |
| --- | --- |
| **HSRP v1** | `0000.0c07.acXX` (XX = group # in hex) |
| **HSRP v2** | `0000.0c9f.fXXX` (XXX = group # in hex) |
| **VRRP** | `0000.5e00.01XX` (XX = group # in hex) |
| **GLBP** | `0007.b400.XXYY` (XX = group, YY = AVF #) |
- *HSRP v1, group 171 (0xAB) → MAC = 0000.0c07.acab*
- *VRRP group 10 (0x0a) → MAC = 0000.5e00.010a*
- *GLBP group 1, AVF 1 → MAC = 0007.b400.0101*

**GLBP Unique Roles:**

- **AVG** (Active Virtual Gateway) = one elected per subnet; assigns virtual MACs to AVFs
- **AVF** (Active Virtual Forwarder) = up to 4; each handles a portion of hosts
- Enables true **load balancing within a single subnet** — unique to GLBP

**HSRP Configuration:**

```
(interface) standby version 2
(interface) standby [group#] ip [virtual-IP]
(interface) standby [group#] priority [0-255]   ← default = 100; higher = preferred active
(interface) standby [group#] preempt            ← allows router to reclaim active role
```

- **Active router election:** Highest priority → if tie, highest IP address wins
- **HSRP v1 and v2 are NOT compatible** — must match on both routers
- **Group number must match** between routers; match group # to VLAN # (best practice)

**Verify:** `show standby` → shows state, virtual IP/MAC, timers, priority, preemption, active/standby IPs

---

# Day 30 (TCP & UDP)

**Layer 4 (Transport Layer)** = transparent data transfer between end hosts

- Provides **port numbers** (Layer 4 addressing) for both TCP and UDP
- **Port numbers** identify Application Layer protocols and enable **session multiplexing**
- **Session** = exchange of data between two communicating devices

**Port Ranges (IANA):**

| Range | Ports | Use |
| --- | --- | --- |
| **Well-known** | 0 – 1023 | Major protocols (HTTP, FTP, etc.) |
| **Registered** | 1024 – 49151 | Requires registration |
| **Ephemeral** | 49152 – 65535 | Randomly selected source ports |

**TCP vs UDP — Core Comparison:**

| Feature | TCP | UDP |
| --- | --- | --- |
| Connection | Connection-oriented | Connectionless |
| Reliable delivery | Yes (ACK + retransmit) | Best-effort only |
| Sequencing | Yes | No |
| Flow control | Yes (window size) | No |
| Header size | Larger (overhead) | Smaller (faster) |
| Use case | File downloads, email | VoIP, video streaming |
- **Both TCP and UDP provide port numbers and session multiplexing**

**TCP Three-Way Handshake (connection setup):**

- **SYN → SYN-ACK → ACK**

**TCP Four-Way Handshake (termination):**

- PC1 sends **FIN** | SRV1 sends **ACK** | SRV1 sends **FIN** | PC1 sends **ACK**

**TCP Key Header Fields:**

- **Source/Destination Port** (16 bits each = 65,536 possible ports)
- **Sequence Number** = tracks order of segments
- **Acknowledgment Number** = forward acknowledgment (next expected sequence #)
- **Window Size** = flow control — slides dynamically to adjust send rate
- **Flags**: SYN, ACK, FIN (used for connection setup/teardown)

**Forward Acknowledgment** = ACK field contains the next expected sequence number, not the one received

- *PC1 sends seq 20 → SRV1 acks with 21 = "send me 21 next"*
- **TCP Retransmission** = if no ACK received after timeout → segment is resent automatically

**Flow Control — Sliding Window:**

- Multiple segments sent before ACK required (e.g. seq 20, 21, 22 → one ACK 23)
- Window size grows until a segment is dropped → resets smaller → slowly grows again

**Important Port Numbers — Memorize:**

| Protocol | Transport | Port(s) |
| --- | --- | --- |
| FTP | TCP | 20, 21 |
| SSH | TCP | 22 |
| Telnet | TCP | 23 |
| SMTP | TCP | 25 |
| HTTP | TCP | 80 |
| POP3 | TCP | 110 |
| HTTPS | TCP | 443 |
| DHCP | UDP | 67, 68 |
| TFTP | UDP | 69 |
| SNMP | UDP | 161, 162 |
| Syslog | UDP | 514 |
| DNS | TCP + UDP | 53 |
- **DNS = only common protocol using BOTH TCP and UDP** (usually UDP, TCP for larger transfers)

---

# Day 31 (IPv6 PT.1)

**IPv6** = 128-bit addresses written in **hexadecimal**

- IPv4 (32-bit) = ~4.3 billion addresses — exhausted
- IPv6 = 340 undecillion+ addresses
- No IPv5 — Internet Stream Protocol used value 5 in IP header, so next version skipped to 6

**Hexadecimal Review:**

- **Hex (base 16)** = digits 0–9 and A(10) B(11) C(12) D(13) E(14) F(15)
- Each hex digit = **4 bits** | 2 hex digits = 1 byte
- Convert binary to hex: split into 4-bit groups → decimal → hex
- *1101 1011 → 13=D, 11=B → DB*

**IPv6 Address Structure:**

- 128 bits ÷ 4 = **32 hex characters** split into **8 groups of 4** separated by colons
- Uses **slash notation** for prefix length (no dotted decimal subnet mask)

**Shortening IPv6 Addresses (2 Rules):**

1. **Remove leading 0s** from any quartet
2. **Replace consecutive all-zero quartets** with `::` — only ONCE per address
- *2001:0db8:0000:0000:0000:0000:0000:0001 → 2001:db8::1*
- Cannot use `::` twice — ambiguous

**Expanding a Shortened Address:**

1. Add leading 0s to make all quartets 4 digits
2. Replace `::` with enough all-zero quartets to make 8 total quartets

**Finding the IPv6 Prefix:**

- Change all **host bits to 0**
- If prefix length is a **multiple of 4** → straightforward (each hex digit = 4 bits)
- If prefix **NOT a multiple of 4** → must convert to binary

**Enterprise IPv6 Structure:**

- ISP assigns a **/48 block** (global routing prefix)
- Subnet identifier = next **16 bits**
- Host portion = last **64 bits** | Convention: use **/64 prefix length** for subnets

**IPv6 Configuration:**

```
(global config) ipv6 unicast-routing        ← REQUIRED to enable IPv6 routing
(interface)     ipv6 address [addr/prefix]
(interface)     no shutdown
```

**Verify:** `show ipv6 interface brief`

- Each interface shows **2 IPv6 addresses** — one configured + one **link-local** (auto-generated)

**IPv6 Static Route:**

```
ipv6 route [destination/prefix] [next-hop]
```

**Invalid IPv6 Address indicators:**

- Contains letters beyond A–F
- More than 8 quartets
- Uses `::` more than once
- Removing non-leading 0s

---

# Day 32 (IPv6 PT.2)

**EUI-64** = method to auto-generate a 64-bit interface ID from a 48-bit MAC address

**EUI-64 Conversion (3 steps):**

1. **Split** MAC address in half
2. **Insert FFFE** in the middle
3. **Invert the 7th bit** (0→1 or 1→0)

Configure EUI-64:

```
(interface) ipv6 address [prefix/64] eui-64
```

**IPv6 Address Types:**

| Type | Range | Routable? | Notes |
| --- | --- | --- | --- |
| **Global Unicast** | 2000::/3 | Yes (Internet) | Public; must register |
| **Unique Local** | FC00::/7 (always FD) | Internal only | Private; no registration |
| **Link-Local** | FE80::/10 | No | Auto-configured; stays in subnet |
| **Multicast** | FF00::/8 | Depends on scope | No broadcast in IPv6 |
| **Anycast** | Any unicast range | Yes | One-to-one-of-many |
| **Unspecified** | ::/0 | No | Device doesn't know its IP yet |
| **Loopback** | ::1 | No | Local device only |
- **IPv6 has NO broadcast — uses multicast (FF02::1 = all nodes) instead**

**Global Unicast Address Parts:**

- /48 Global Routing Prefix | 16-bit Subnet ID | 64-bit Interface ID

**Unique Local Address Parts:**

- **FD** + **40-bit Global ID** (randomly generated) + **16-bit Subnet ID** + **64-bit Interface ID**
- Global ID should be randomly generated to avoid subnet overlap when companies merge

**Link-Local Addresses:**

- Always begins with **FE80**
- NOT routed between subnets — stays on local link only
- Used for: OSPF neighbor adjacencies, NDP, next-hop addresses in static routes

**IPv6 Multicast Scopes — scope = 4th hex character:**

| Scope | Prefix | Boundary |
| --- | --- | --- |
| Interface-local | FF01:: | Local device only |
| Link-local | FF02:: | Local subnet |
| Site-local | FF05:: | One physical site |
| Org-local | FF08:: | Entire organization |
| Global | FF0E:: | No boundaries |

**Important Multicast Addresses:**

| Purpose | IPv6 | IPv4 |
| --- | --- | --- |
| All nodes/hosts | FF02::1 | — |
| All routers | FF02::2 | — |
| All OSPF routers | FF02::5 | 224.0.0.5 |
| All OSPF DR/BDR | FF02::6 | 224.0.0.6 |
| All EIGRP routers | FF02::A | 224.0.0.10 |
| All RIP routers | FF02::9 | 224.0.0.9 |

**Special Addresses:**

- **::** (all zeros) = unspecified; IPv4 equivalent = 0.0.0.0
- **::1** = loopback address; IPv4 equivalent = 127.0.0.1

---

# Day 33 (IPv6 PT.3)

**NDP** (Neighbor Discovery Protocol) = replaces ARP in IPv6; uses ICMPv6 + solicited-node multicast
**SLAAC** = Stateless Address Auto-configuration; host learns prefix via NDP → auto-generates IPv6 address
**DAD** = Duplicate Address Detection; checks if another device uses same IPv6 address

**IPv6 Address Representation (RFC 5952):**

- Leading 0s MUST be removed
- Double colon used for longest string of all-0 quartets; if tied → use left side
- Single all-0 quartet → don't use double colon
- Hex letters MUST be lower-case (a–f)

**IPv6 Header (fixed = 40 bytes):**

| Field | Size | Purpose |
| --- | --- | --- |
| Version | 4 bits | Always = 6 |
| Traffic Class | 8 bits | QoS / priority |
| Flow Label | 20 bits | Identifies traffic flows |
| Payload Length | 16 bits | Length of encapsulated segment |
| Next Header | 8 bits | Type of next header (TCP/UDP) |
| Hop Limit | 8 bits | Decremented by each router; 0 = discard |
| Source Address | 128 bits | IPv6 source |
| Destination Address | 128 bits | IPv6 destination |
- No header length field (always 40 bytes) | No checksum | Simpler than IPv4

**Solicited-Node Multicast Address:**

- Format: **ff02::1:ff** + last 6 hex digits of unicast address

**NDP Message Types — Memorize all 4:**

| Message | ICMPv6 Type | Purpose | Sent To |
| --- | --- | --- | --- |
| **NS** (Neighbor Solicitation) | 135 | Learn MAC address (like ARP request) | Solicited-node multicast |
| **NA** (Neighbor Advertisement) | 136 | Reply with MAC (like ARP reply) | Unicast back to requester |
| **RS** (Router Solicitation) | 133 | Ask routers to identify themselves | FF02::2 (all routers) |
| **RA** (Router Advertisement) | 134 | Router announces presence + prefix info | FF02::1 (all nodes) |
- No ARP in IPv6 — replaced by NS/NA | View neighbor table: `show ipv6 neighbor`
- **DAD process:** Send NS to own solicited-node multicast address → no reply = unique; reply = duplicate
- **SLAAC:** `ipv6 address autoconfig` → uses RS/RA to learn prefix → generates interface ID via EUI-64

**IPv6 Static Route Types:**

| Type | Specifies | Notes |
| --- | --- | --- |
| **Directly Attached** | Exit interface only | Does NOT work on Ethernet in IPv6 |
| **Recursive** | Next-hop only | Requires recursive routing table lookup |
| **Fully Specified** | Exit interface + next-hop | Required when using link-local next-hop |
- Link-local next-hop MUST use fully specified route

**Route Destination Types:**

- **Network route** → /64 prefix
- **Host route** → /128 prefix
- **Default route** → ::/0
- **Floating static** → raise AD above primary route's AD

---

# Day 34 (Standard ACLs)

**ACL** (Access Control List) = packet filter; permits or denies traffic based on rules
**ACE** (Access Control Entry) = individual rule within an ACL

- ACLs filter based on **source IP** (standard) or src/dst IP + ports (extended)
- Configured globally → must be **applied to an interface** (inbound or outbound) to take effect
- Max 1 ACL per interface per direction — applying a second replaces the first

**ACL Processing Rules:**

- Processed **top to bottom** — first match wins; remaining entries ignored
- **Implicit deny** at the end of EVERY ACL — unmatched traffic is dropped
- Always add `permit any` at the end if you don't want to block everything else

**ACL Types:**

| Type | Matches On | Number Range |
| --- | --- | --- |
| **Standard Numbered** | Source IP only | 1–99, 1300–1999 |
| **Standard Named** | Source IP only | Name (text) |
| **Extended Numbered** | Src/Dst IP + ports | 100–199, 2000–2699 |
| **Extended Named** | Src/Dst IP + ports | Name (text) |
- **Standard ACL rule of thumb: apply as close to the DESTINATION as possible**

**Standard Numbered ACL Configuration:**

```
access-list [1-99 or 1300-1999] {permit|deny} [IP] [wildcard]
access-list 1 deny host 1.1.1.1
access-list 1 permit any
access-list 1 remark [description]
```

Apply to interface: `ip access-group [#] {in|out}`

**Standard Named ACL Configuration:**

```
ip access-list standard [NAME]
  [seq#] deny 192.168.1.0 0.0.0.255
  [seq#] permit any
  remark [description]
(interface) ip access-group [NAME] {in|out}
```

- Use **wildcard masks** in ACLs — NOT subnet masks
- `ip access-list` (named) differs from `access-list` (numbered)

**Verification Commands:**

- `show access-lists` → all ACL types
- `show ip access-lists` → IP ACLs only
- `show running-config | section access-list` → full ACL with entries
- `show running-config | include access-list` → ACL name only
- Router auto-converts `/32` entries: `deny 1.1.1.1 0.0.0.0` → `deny 1.1.1.1`
- Router auto-converts `0.0.0.0 255.255.255.255` → `any`
- Remarks only appear in running-config, not in `show access-lists`

---

# Day 35 (Extended ACLs)

**Extended ACLs** = match on protocol, src/dst IP, src/dst port — more precise than standard

- Number ranges: **100–199** and **2000–2699**
- **Extended ACLs: apply as close to the SOURCE as possible** (opposite of standard)

**Numbered ACL in Named Config Mode (key advantages):**

- Traditional global config mode → `no access-list [#]` deletes the ENTIRE ACL
- Named config mode → `no [seq#]` deletes only that entry
- Named config mode → specify seq# to insert entries in the middle

**Resequencing:** `ip access-list resequence [ACL] [start] [increment]`

**Extended ACL Configuration:**

```
ip access-list extended [NAME or 100-199/2000-2699]
  [seq#] {permit|deny} [protocol] [src IP][wildcard] [dst IP][wildcard]
  [seq#] permit ip any any
(interface) ip access-group [NAME/#] {in|out}
```

**Protocols to Know:**

| Protocol | Number | Name |
| --- | --- | --- |
| ICMP (ping) | 1 | icmp |
| TCP | 6 | tcp |
| UDP | 17 | udp |
| EIGRP | 88 | eigrp |
| OSPF | 89 | ospf |
| All IP | — | ip |

**Port Matching Options:**

| Keyword | Meaning |
| --- | --- |
| `eq [port]` | Equal to port |
| `gt [port]` | Greater than port |
| `lt [port]` | Less than port |
| `neq [port]` | Not equal to port |
| `range [x] [y]` | Port range x to y |
- Must match ALL specified parameters — if even one doesn't match, no match
- `permit ip any any` = extended ACL version of `permit any`

**Key Port Numbers:**

| Protocol | Port | Transport |
| --- | --- | --- |
| FTP | 20, 21 | TCP |
| SSH | 22 | TCP |
| Telnet | 23 | TCP |
| HTTP | 80 | TCP |
| HTTPS | 443 | TCP |
| TFTP | 69 | UDP |
| DHCP | 67, 68 | UDP |
| DNS | 53 | TCP+UDP |

**Verify ACL on interface:** `show ip interface [int]`

- Standard → close to **destination**
- Extended → close to **source**

---

# Day 36 (CDP & LLDP)

**Layer 2 Discovery Protocols** = share info (hostname, IP, device type, SW version) with directly connected neighbors only

- Security risk — share network info; many admins disable them
- **CDP** = Cisco proprietary | **LLDP** = IEEE 802.1AB (industry standard, multi-vendor)

**CDP Key Facts:**

- Enabled by default on all Cisco devices
- Multicast MAC: **0100.0CCC.CCCC**
- Messages sent every **60 seconds** | Holdtime = **180 seconds**
- Version 2 used by default
- Messages not forwarded — only directly connected neighbors become CDP neighbors

**CDP Configuration:**

```
(global)    cdp run / no cdp run
(interface) cdp enable / no cdp enable
(global)    cdp timer [seconds]
(global)    cdp holdtime [seconds]
(global)    cdp advertise-v2
```

**CDP Show Commands:**

| Command | Shows |
| --- | --- |
| `show cdp` | Timers, version |
| `show cdp traffic` | Packets sent/received |
| `show cdp interface [int]` | Per-interface timer info |
| `show cdp neighbors` | Neighbor table (basic) |
| `show cdp neighbors detail` | + IP, OS version, VTP, duplex, native VLAN |
| `show cdp entry [name]` | Detail for one neighbor |
- `show cdp neighbors` does NOT show IP address or SW version → need `detail`

**CDP Neighbor Table Fields:**

| Column | Meaning |
| --- | --- |
| Device ID | Neighbor hostname |
| Local Interface | Interface on this device |
| Holdtime | Countdown until neighbor removed |
| Capability | R=Router, S=Switch, I=IGMP |
| Platform | Neighbor model number |
| Port ID | Interface on neighbor device |
- Local Interface does not equal Port ID

**LLDP Key Facts:**

- Disabled by default on Cisco devices — must manually enable
- Multicast MAC: **0180.C200.000E**
- Messages sent every **30 seconds** | Holdtime = **120 seconds** | Reinit delay = **2 seconds**
- Two separate interface commands required:

```
(global)    lldp run
(interface) lldp transmit
(interface) lldp receive
(global)    lldp timer [seconds]
(global)    lldp holdtime [seconds]
(global)    lldp reinit [seconds]
```

- **R** = Router | **B** = Bridge (switch) | No "S" in LLDP
- LLDP cannot share VTP info — VTP is Cisco proprietary; only CDP can share it

**CDP vs LLDP Quick Comparison:**

| Feature | CDP | LLDP |
| --- | --- | --- |
| Standard | Cisco only | IEEE 802.1AB |
| Default state | Enabled | Disabled |
| Timer | 60s | 30s |
| Holdtime | 180s | 120s |
| MAC address | 0100.0CCC.CCCC | 0180.C200.000E |
| VTP info | Yes | No |
| Multi-vendor | No | Yes |

---

# Day 37 (NTP)

**NTP** = auto-syncs time across network devices | UDP **port 123**

- Why it matters: accurate logs for troubleshooting (syslog timestamps)
- Default time zone = **UTC** | Hardware clock (calendar) differs from software clock

**Manual Time Commands (privileged-exec mode):**

```
clock set HH:MM:SS [day] [month] [year]
calendar set HH:MM:SS [day] [month] [year]
clock update-calendar
clock read-calendar
show clock / show clock detail
show calendar
```

**Time Zone & DST (global config mode):**

```
clock timezone JST 9
clock summer-time EDT recurring 2 Sunday March 2:00 1 Sunday November 2:00
```

- Time zone is in running-config | `clock set` is NOT in running-config

**NTP Hierarchy (Stratum):**

- **Stratum** = distance from reference clock | lower = more accurate | max = 15
- Stratum 0 = reference clock (atomic/GPS)
- Stratum 1 = primary servers (directly connected to stratum 0)
- Stratum 2+ = secondary servers
- When a device syncs to a stratum N server → it becomes stratum N+1

**NTP Modes:**

| Mode | Command | Description |
| --- | --- | --- |
| **Client** | `ntp server [IP]` | Syncs to NTP server |
| **Server** (manual) | `ntp master [stratum]` | Acts as master clock; default stratum = 8 |
| **Symmetric Active** | `ntp peer [IP]` | Peers with equal-stratum device |

**NTP Configuration:**

```
ntp server [IP]
ntp server [IP] prefer
ntp master [stratum]
ntp peer [IP]
ntp source loopback0
ntp update-calendar
```

- **NTP MASTER** = device uses loopback address `127.127.1.1` as its own reference clock
- **NTP SERVER** = puts device in static client mode (auto-becomes server for others too)

**NTP Verification:**

```
show ntp associations
show ntp status
show clock detail
```

`show ntp associations` flags:

- = sys.peer (currently syncing)
- `+` = candidate
- `~` = configured
- No mark / "x" = outlier/falseticker

**NTP Authentication:**

```
ntp authenticate
ntp authentication-key 1 md5 [password]
ntp trusted-key 1
ntp server [IP] key 1
ntp peer [IP] key 1
```

- All 4 commands needed on clients | Server needs first 3 only
- Same key must match on server and all clients

---

# Day 38 (DNS)

**DNS** = resolves human-readable names (e.g. [youtube.com](http://youtube.com/)) to IP addresses

- DNS server learned via DHCP (automatic) or manually configured
- Routers don't need DNS config to forward DNS traffic — they just route the packets

**DNS Record Types:**

- **A record** = maps hostname → IPv4 address
- **AAAA record** (quad-A) = maps hostname → IPv6 address
- **CNAME** (Canonical Name) = maps a name → another name

**Transport:**

- DNS uses **UDP port 53** for standard queries/responses
- DNS uses **TCP port 53** for messages greater than 512 bytes

**DNS Process:**

1. Host checks local DNS cache first
2. Host checks hosts file (if configured)
3. Host sends DNS query to configured DNS server
4. DNS server replies with IP address
5. Host caches the response (temp = expires; manual = permanent)

**Windows Commands:**

| Command | Purpose |
| --- | --- |
| `ipconfig /all` | Show IP config including DNS server |
| `nslookup [name]` | Query DNS server for IP of name |
| `ipconfig /displaydns` | View local DNS cache |
| `ipconfig /flushdns` | Clear local DNS cache |
- `ipconfig` alone does NOT show DNS server — need `/all`

**DNS Configuration on Cisco Router:**

Router as DNS Server:

```
ip dns server
ip host PC1 192.168.0.1
ip name-server 8.8.8.8
ip domain lookup
```

Router as DNS Client only:

```
ip name-server 8.8.8.8
ip domain lookup
ip domain name jeremysitlab.com
```

- `ip domain lookup` = enabled by default
- Old syntax uses hyphen: `ip domain-lookup` / `ip domain-name` — both still work

**Verify DNS on Router:**

```
show hosts
```

Shows manually configured hosts (perm) + DNS-cached hosts (temp)

**Hosts File:**

- Windows location: `C:\\Windows\\System32\\drivers\\etc\\hosts`
- Format: `[IP address] [hostname]`
- Checked before DNS query

**Key Distinctions for Exam:**

- HTTP uses TCP → web browser accessing a website = TCP connection to web server
- DNS queries use UDP → no TCP connection established with DNS server

---

# Day 39 (DHCP)

**DHCP** = Dynamic Host Configuration Protocol; auto-assigns IP, subnet mask, default gateway, DNS server, etc.

- Used for client devices (PCs, phones); servers/routers use static IPs
- DHCP uses UDP: server = port 67 | client = port 68

**DORA Process — memorize order:**

| # | Message | Direction | Broadcast? |
| --- | --- | --- | --- |
| 1 | **Discover** | Client → Server | Always broadcast |
| 2 | **Offer** | Server → Client | Unicast OR broadcast |
| 3 | **Request** | Client → Server | Always broadcast |
| 4 | **Ack** | Server → Client | Unicast OR broadcast |
| — | **Release** | Client → Server | Always unicast |
- **Lease** = temporary assignment of IP address
- Discover source IP = 0.0.0.0 | destination = 255.255.255.255

**DHCP Relay Agent:**

- Problem: DHCP Discover is broadcast → routers don't forward broadcasts → centralized server unreachable
- Solution: Configure router as DHCP relay agent → forwards client broadcasts as unicast

**Windows Commands:**

- `ipconfig /all` → shows DHCP enabled/disabled, IP, lease time, DNS, gateway
- `ipconfig /release` → sends DHCP Release; gives up IP address
- `ipconfig /renew` → triggers DORA process to get new IP

**DHCP Server Configuration:**

```
ip dhcp excluded-address 192.168.1.1 192.168.1.10
ip dhcp pool [NAME]
  network 192.168.1.0 /24
  dns-server 8.8.8.8
  domain-name jeremysitlab.com
  default-router 192.168.1.1
  lease 0 5 30
```

**Verify DHCP Server:**

```
show ip dhcp binding
```

**DHCP Relay Agent Configuration:**

```
(interface facing clients)
ip helper-address [DHCP server IP]
```

Verify: `show ip interface [int]` → shows "helper address" line

- Router must have a route to the DHCP server or relay won't work

**DHCP Client Configuration:**

```
(interface)
ip address dhcp
```

**Key Facts for Exam:**

- DHCP clients = UDP 68 | DHCP servers = UDP 67
- Discover and Request = always broadcast
- Offer, Ack, Release = can be unicast (Release always unicast)
- `ip helper-address` = DHCP relay agent command
- Centralized DHCP server needs relay agent on each router connecting client subnets

---

# Day 40 (SNMP)

**SNMP** = framework/protocol to monitor and manage network devices

- SNMP Agent listens on UDP port 161 | SNMP Manager listens on UDP port 162

**Two Main SNMP Components:**

| Component | Role |
| --- | --- |
| **Managed Device** | Device being managed — runs SNMP Agent |
| **NMS** (Network Management Station) | SNMP server — runs SNMP Manager + SNMP Application |
- **SNMP Agent** = software on managed device
- **MIB** (Management Information Base) = database of variables on managed device
- **OID** (Object ID) = unique identifier for each variable in the MIB

**Three Main SNMP Operations:**

1. Managed device notifies NMS of events
2. NMS reads info from managed device
3. NMS writes config changes to managed device

**SNMP Versions:**

| Version | Key Feature |
| --- | --- |
| **SNMPv1** | Original; uses community strings |
| **SNMPv2c** | Adds GetBulk; community strings back |
| **SNMPv3** | Preferred — adds encryption + authentication |
- SNMPv1 and v2c send community strings in plain text — not secure

**SNMP Message Types:**

| Class | Message | Direction | Reliable? |
| --- | --- | --- | --- |
| Read | Get | NMS → Agent | N/A |
| Read | GetNext | NMS → Agent | N/A |
| Read | GetBulk (v2+) | NMS → Agent | N/A |
| Write | Set | NMS → Agent | N/A |
| Notification | Trap | Agent → NMS | No — no response |
| Notification | Inform | Agent → NMS | Yes — acknowledged |
| Response | Response | Agent → NMS | N/A |
- Trap = unreliable (no ACK, no retransmission — uses UDP)
- Inform = reliable (NMS sends Response to acknowledge receipt)

**SNMP Port Summary:**

- Managed devices (agents) listen on UDP 161 — receive Get, Set from NMS
- NMS (manager) listens on UDP 162 — receives Trap, Inform from agents

**Basic SNMPv2c Configuration:**

```
snmp-server contact [email]
snmp-server location [location]
snmp-server community Jeremy1 ro
snmp-server community Jeremy2 rw
snmp-server host [NMS IP] version 2c Jeremy1
snmp-server enable traps snmp linkdown linkup
snmp-server enable traps config
```

- Default community strings: "public" (RO) and "private" (RW) — change these for security
- SNMPv2c community strings shown in plain text — not secure → use v3

---

# Day 41 (Syslog)

**Syslog** = industry standard protocol for logging events on network devices

- Syslog server listens on UDP port 514
- Syslog messages sent FROM devices TO server — server cannot pull info or make changes

**Syslog Message Format:**`[seq#] [timestamp] : %[facility]-[severity]-[mnemonic]: [description]`

| Field | Description | Always shown? |
| --- | --- | --- |
| Sequence # | Order of messages | Optional |
| Timestamp | Date/time of event | Optional (but recommended) |
| **Facility** | Which process generated it | Yes |
| **Severity** | How serious the event is (0–7) | Yes |
| Mnemonic | Short code describing what happened | Yes |
| Description | Full detail of the event | Yes |

**Syslog Severity Levels — MEMORIZE:**

| Level | Keyword | Meaning |
| --- | --- | --- |
| 0 | Emergency | System unusable |
| 1 | Alert | Immediate action needed |
| 2 | Critical | Critical conditions |
| 3 | Error | Error conditions |
| 4 | Warning | Warning conditions |
| 5 | Notice / Notification | Normal but significant |
| 6 | Informational | Informational messages |
| 7 | Debugging | Debug-level messages |

Mnemonic: Every Awesome Cisco Engineer Will Need Ice cream Daily

**Logging Locations & Defaults:**

| Location | Default | Command |
| --- | --- | --- |
| Console line | Enabled (levels 0–7) | `logging console [level]` |
| VTY lines (SSH/Telnet) | Disabled | `logging monitor [level]` |
| Buffer (RAM) | Enabled (levels 0–7) | `logging buffered [size] [level]` |
| External server | Disabled | `logging [IP]` or `logging host [IP]` |
- Specifying a level = enables that level AND all more severe levels (numerically lower)

**Syslog Configuration Commands:**

```
logging console 6
logging monitor informational
logging buffered 8192 6
logging 192.168.1.100
logging trap debugging
terminal monitor               ← REQUIRED each SSH/Telnet session to see logs
line console 0
  logging synchronous
service timestamps log datetime
service timestamps log uptime
service sequence-numbers
```

- `terminal monitor` only lasts for that session — must re-enter each time

**Syslog vs SNMP:**

|  | Syslog | SNMP |
| --- | --- | --- |
| Direction | Device → Server (push) | NMS can pull/push |
| Server can pull info? | No | Yes (Get) |
| Server can change config? | No | Yes (Set) |
| Port | UDP 514 | UDP 161/162 |
| Purpose | Event logging | Device management + monitoring |

---

# Day 42 (**SSH**)

**Key Takeaways**

- SSH replaces Telnet for secure remote CLI access — Telnet sends data in plain text, SSH encrypts it
- SSH uses TCP port 22 | Telnet uses TCP port 23
- To generate RSA keys, you must first configure a non-default hostname AND a domain name
- RSA key modulus must be at least 768 bits to use SSHv2
- K9 IOS images support SSH; NPE images do not

**Console Port Security**

By default, no password is required to access the CLI via the console port.

**Method 1 — Password only:**

```
line console 0
  password ccna
  login              ← required to enforce the password
  exec-timeout 3 30  ← logs out after 3 min 30 sec of inactivity
```

**Method 2 — Username and password (recommended):**

```
username jeremy secret ccnp   ← create local user account first

line console 0
  login local        ← requires username + password from local user database
  exec-timeout 3 30
```

- `login` = requires the line password
- `login local` = requires a local username and password
- Even if a line password exists, `login local` overrides it — the line password can no longer be used
- **exec-timeout** is a good security practice in case a user leaves a console session open

**Layer 2 Switch Management IP**

Layer 2 switches don't route packets, but they can have a management IP assigned to an SVI:

```
interface vlan 1
  ip address 192.168.1.2 255.255.255.0
  no shutdown

ip default-gateway 192.168.1.1   ← required for switch to communicate outside its local LAN
```

- Without a default gateway, the switch cannot reach devices on other networks
- This is separate from a routing table — Layer 2 switches use `ip default-gateway` instead of `ip route`

**Telnet**

- Developed in 1969; largely replaced by SSH
- Sends all data including usernames and passwords in **plain text — not secure**
- Telnet server listens on **TCP port 23**

**Telnet Configuration:**

```
enable secret [password]
username [name] secret [password]

line vty 0 15                    ← configure all 16 VTY lines (up to 16 simultaneous connections)
  login local
  exec-timeout 5 0
  transport input telnet         ← allow Telnet only
  access-class [ACL#] in        ← restrict which IPs can connect (optional but recommended)
```

- `access-class` applies an ACL to VTY lines (different from `ip access-group` used on interfaces)
- `transport input telnet` — Telnet only
- `transport input ssh` — SSH only
- `transport input telnet ssh` — both
- `transport input all` — all protocols
- `transport input none` — no remote connections allowed

**SSH**

- Developed in 1995 to replace Telnet and other insecure protocols
- Encrypts all data — username, password, and all traffic
- **SSH uses TCP port 22**
- SSHv2 released in 2006 — more secure; should always be used
- If a device supports both v1 and v2, it shows version **1.99** (not a real version number — means both supported)

**Verify SSH support before configuring:**

```
show version     ← look for K9 in the IOS image name = SSH supported
show ip ssh      ← shows SSH status and version; "SSH disabled" if RSA keys not generated
```

- K9 in image name = cryptography supported = SSH supported
- NPE (No Payload Encryption) images = no SSH support

**SSH Configuration — Step by Step:**

**Step 1 — Configure a non-default hostname:**

```
hostname SW1
```

Cannot be "Router" or "Switch" — must be changed first.

**Step 2 — Configure a domain name:**

```
ip domain name jeremysitlab.com
```

The FQDN (hostname + domain name) is used to name the RSA key pair.

**Step 3 — Generate RSA keys:**

```
crypto key generate rsa modulus 2048
```

- Minimum modulus = **768 bits** to use SSHv2
- Larger key = more secure but slower to generate
- After generation, Syslog message confirms SSH is enabled

**Step 4 — Configure enable secret and local user:**

```
enable secret [password]
username [name] secret [password]
```

**Step 5 — Enable SSHv2 only (recommended):**

```
ip ssh version 2
```

**Step 6 — Configure VTY lines:**

```
line vty 0 15
  login local                   ← SSH requires login local (not just login)
  exec-timeout 5 0
  transport input ssh           ← best practice: SSH only, no Telnet
  access-class [ACL#] in        ← optional but recommended
```

**Connect to SSH from a client:**

```
ssh -l [username] [IP address]
ssh [username]@[IP address]
```

---

**Key Command Distinctions — Don't Confuse These:**

| Command | Purpose |
| --- | --- |
| `access-list` / `ip access-list` | Create an ACL |
| `ip access-group` | Apply ACL to an interface |
| `access-class` | Apply ACL to VTY lines |

**SSH vs Telnet Comparison:**

| Feature | SSH | Telnet |
| --- | --- | --- |
| Port | TCP 22 | TCP 23 |
| Encryption | Yes | No (plain text) |
| Year developed | 1995 | 1969 |
| Security | Secure | Not secure |
| Recommended? | Yes | No |

---

# Day 43 (**FTP & TFTP**)

**Key Takeaways**

- Both FTP and TFTP transfer files over a network using a client-server model
- TFTP is simple, uses UDP port 69, has no authentication or encryption
- FTP is feature-rich, uses TCP ports 20 and 21, has username/password authentication but no encryption
- FTP active mode: server initiates data connection | FTP passive mode: client initiates (used behind firewalls)
- Most common use case for network engineers: downloading new IOS images to upgrade devices

Purpose of FTP and TFTP

Both protocols transfer files between devices over a network using a client-server model. Clients can copy files to or from a server. For network engineers, the most common use is upgrading the operating system of a network device by downloading a newer IOS image.

**Typical IOS upgrade workflow:**

1. Download new IOS image from Cisco (software.cisco.com)
2. Transfer the image to a local FTP or TFTP server reachable by the device
3. Use CLI commands on the device to copy the file to flash memory
4. Configure the device to boot using the new image
5. Reload the device

**TFTP — Trivial File Transfer Protocol**

- First standardized in 1981
- Named "trivial" because it is intentionally simple with only basic features
- Only allows a client to **copy a file to or from a server** — nothing else
- No authentication (no usernames or passwords) — servers respond to all TFTP requests
- No encryption — all data sent in plain text
- Best used in a **controlled environment** to transfer small files quickly
- Listens on **UDP port 69**

**TFTP Reliability (built into the protocol itself):**

- Every data message is acknowledged with an Ack message
- Timers are used — if an expected message is not received in time, the previous message is retransmitted
- Uses **lock-step communication** — devices alternately send a message and wait for a reply before sending the next

**TFTP Connection Phases:**

1. **Connection** — client sends a request; server responds, initializing the connection
2. **Data Transfer** — data and Ack messages are exchanged
3. **Termination** — client sends a final Ack for the last data message; connection ends

**TFTP TID (Transfer Identifier) — not required for CCNA but useful to know:**

- Client sends first message to server using destination port 69 and a random source port (TID)
- Server replies using its own random port as the source (not port 69)
- All subsequent messages use these two random ports — port 69 is only used in the very first message

**FTP — File Transfer Protocol**

- First standardized in 1971 (predates TCP/IP)
- Uses **TCP ports 20 and 21**
- Requires **username and password authentication**
- No encryption — all data including credentials sent in plain text
- For greater security: use **FTPS** (FTP over SSL/TLS, also called FTP Secure) or **SFTP** (SSH File Transfer Protocol — a different protocol entirely)
- Much more feature-rich than TFTP — clients can navigate directories, add/remove directories, list files, and more in addition to transferring files

**FTP uses two types of connections:**

**Control connection (TCP port 21):**

- Established first; used to send FTP commands and receive replies
- Maintained throughout the entire session

**Data connection (TCP port 20):**

- Established separately when files or data need to be transferred
- Created and terminated as needed

**FTP Data Connection Modes:**

**Active mode (default):**

- The server initiates the TCP data connection to the client
- Can be blocked by firewalls on the client side

**Passive mode:**

- The client initiates the TCP data connection to the server
- Used when the client is behind a firewall, since firewalls typically block unsolicited incoming connections from external devices

**FTP vs TFTP Comparison**

| Feature | FTP | TFTP |
| --- | --- | --- |
| Transport | TCP | UDP |
| Port(s) | 20 (data), 21 (control) | 69 |
| Authentication | Yes (username + password) | No |
| Encryption | No (use FTPS or SFTP) | No |
| File transfer | Yes | Yes |
| Directory navigation | Yes | No |
| List files on server | Yes | No |
| Delete files on server | Yes | No |
| Complexity | High | Low (trivial) |

**IOS File Systems**

View file systems with: `show file systems`

| Type | Description |
| --- | --- |
| **disk** | Flash memory — where the IOS image is stored; copied into RAM at boot |
| **opaque** | Logical internal systems used for specific internal functions |
| **nvram** | Non-volatile RAM — stores the startup-config; preserves data after power loss |
| **network** | External file systems such as FTP or TFTP servers |

**Upgrading Cisco IOS — CLI Commands**

**View current IOS version:**

```
show version
```

**View contents of flash:**

```
show flash
```

**Copy IOS image from TFTP server to flash:**

```
copy tftp: flash:
```

Then enter: TFTP server IP address, source filename, destination filename (press Enter to keep same name)

**Copy IOS image from FTP server to flash:**

```
ip ftp username cisco
ip ftp password cisco

copy ftp: flash:
```

Then enter: FTP server IP address, source filename, destination filename

**Configure device to boot with the new IOS image:**

```
boot system flash:[filename]
```

- If this command is not used, the router boots from the first IOS file it finds in flash

**Save configuration and reload:**

```
copy running-config startup-config
reload
```

**Delete old IOS image:**

```
delete flash:[filename]
```

**Verify new version after reload:**

```
show version
```

---

# Day 44 (NAT (Part 1))

**Key Takeaways**

- Private IPv4 addresses (RFC 1918) cannot be used over the Internet — NAT allows internal hosts to communicate externally
- Static NAT creates permanent one-to-one mappings between inside local and inside global addresses
- Inside local = actual IP on the internal host (usually private) | Inside global = IP after NAT (usually public)
- Outside local and outside global are typically the same unless destination NAT is used
- Static NAT does not conserve public IP addresses — each internal host needs its own public IP

**Private IPv4 Addresses (RFC 1918)**

Three solutions to IPv4 address exhaustion: CIDR, private IPv4 addresses, and NAT.

RFC 1918 specifies three private IPv4 address ranges:

| Range | Addresses |
| --- | --- |
| 10.0.0.0/8 | 10.0.0.0 – 10.255.255.255 |
| 172.16.0.0/12 | 172.16.0.0 – 172.31.255.255 |
| 192.168.0.0/16 | 192.168.0.0 – 192.168.255.255 |
- These addresses do not have to be globally unique — multiple organizations can use the same private ranges
- Private IP addresses **cannot be used over the Internet** — ISPs will drop traffic to or from private addresses
- NAT solves this by translating private addresses to public ones before traffic reaches the Internet

**NAT Overview**

NAT (Network Address Translation) modifies the source and/or destination IP addresses of packets.

Most common reason: allow hosts with private IP addresses to communicate over the Internet.

For the CCNA, the focus is **source NAT** — translating the source IP address of outgoing packets.

Basic source NAT flow:

- PC1 (192.168.0.167) sends a packet to 8.8.8.8
- R1 translates the source from 192.168.0.167 to a public IP (e.g. 203.0.113.1)
- The packet travels over the Internet with the public source IP
- The server replies to 203.0.113.1
- R1 reverses the translation back to 192.168.0.167 and forwards it to PC1

**Static NAT**

Static NAT involves statically configuring **one-to-one mappings** of private IP addresses to public IP addresses.

**Four NAT terms to know:**

| Term | Definition |
| --- | --- |
| **Inside local** | IP address of the inside host from the perspective of the local network — the actual IP configured on the host (usually private) |
| **Inside global** | IP address of the inside host from the perspective of the outside network — the IP after NAT is applied (usually public) |
| **Outside local** | IP address of the outside host from the perspective of the local network |
| **Outside global** | IP address of the outside host from the perspective of the outside network |
- Outside local and outside global are the same unless destination NAT is used (destination NAT is outside CCNA scope)
- "Inside" and "outside" indicate the location of the host
- "Local" and "global" indicate the perspective — local = inside network's view; global = outside network's view

Example:

- PC1's inside local address = 192.168.0.167 (actual configured IP)
- PC1's inside global address = 100.0.0.1 (IP after NAT, seen by outside hosts)
- Server's outside local = 8.8.8.8 | Server's outside global = 8.8.8.8 (same, no destination NAT)

**Limitation of static NAT:** It is a one-to-one mapping, so each internal device needs its own public IP address. This does not help conserve public IP addresses.

**Static NAT Configuration**

Step 1 — Define inside and outside interfaces:

```
interface g0/1
  ip nat inside         ← internal network interface

interface g0/0
  ip nat outside        ← external/Internet-facing interface
```

Step 2 — Configure one-to-one address mappings:

```
ip nat inside source static 192.168.0.167 100.0.0.1
ip nat inside source static 192.168.0.168 100.0.0.2
```

- Format: `ip nat inside source static [inside local] [inside global]`
- A public IP address cannot be mapped to two different private IPs — the second command will be rejected

**Verification Commands**

View NAT translation table:

```
show ip nat translations
```

- Static entries appear permanently in the table
- Dynamic entries appear when translations are actively occurring and time out when traffic stops
- Columns: Protocol | Inside Global | Inside Local | Outside Local | Outside Global

Clear dynamic NAT translations (static entries remain):

```
clear ip nat translation *
```

View NAT statistics:

```
show ip nat statistics
```

- Shows total active translations, number of static vs dynamic, peak translations, inside and outside interfaces

**Command Summary**

| Command | Purpose |
| --- | --- |
| `ip nat inside` | Mark interface as inside (internal) |
| `ip nat outside` | Mark interface as outside (external) |
| `ip nat inside source static [local IP] [global IP]` | Configure static NAT mapping |
| `show ip nat translations` | View NAT translation table |
| `show ip nat statistics` | View NAT statistics |
| `clear ip nat translation *` | Clear all dynamic NAT translations |

---

# Day 45 (NAT (Part 2))

**Key Takeaways**

- Static NAT works two-ways — external hosts can reach internal hosts using the inside global address
- Dynamic NAT automatically assigns inside global addresses from a pool on a one-to-one basis, but mappings are temporary
- PAT (NAT Overload) translates both IP address and port number, allowing many internal hosts to share a single public IP
- PAT is the most widely used form of NAT — it is how home routers work
- When all pool addresses are exhausted in dynamic NAT, the router drops new packets requiring translation

**Static NAT — Additional Point**

Static NAT works in both directions:

- Internal host can reach external hosts (inside-to-outside)
- External hosts can also initiate connections to the internal host using the inside global IP address
- R1 will translate the destination IP of incoming packets to the inside local address and forward them to the internal host

**Dynamic NAT**

In dynamic NAT, the router automatically creates inside local to inside global mappings as needed, rather than being manually configured.

How it works:

- An ACL identifies which traffic should be translated — traffic permitted by the ACL will have its source IP translated; traffic denied will NOT be translated (but will not be dropped)
- A NAT pool defines the range of available inside global IP addresses
- When a packet arrives and its source IP is permitted by the ACL, the router assigns the next available address from the pool

Key characteristics:

- Mappings are still **one-to-one** — one inside local per inside global at any given time
- Mappings are **temporary** — they time out after about 24 hours of inactivity (timer resets with each translation)
- Default timeout for dynamic mappings is 24 hours; protocol-specific entries (UDP, ICMP) clear after about 1 minute
- Use `clear ip nat translation *` to manually clear dynamic entries (static entries are not cleared)

**NAT pool exhaustion:**

- If all inside global addresses in the pool are in use and another host needs translation, the router drops the packet
- The host will be unable to access outside networks until a pool address becomes available
- Addresses free up when dynamic mappings time out or are manually cleared

**Dynamic NAT Configuration**

Step 1 — Define inside and outside interfaces (same as static NAT):

```
interface g0/1
  ip nat inside

interface g0/0
  ip nat outside
```

Step 2 — Define which traffic to translate using an ACL:

```
access-list 1 permit 192.168.0.0 0.0.0.255
```

Step 3 — Define the NAT pool:

```
ip nat pool POOL1 100.0.0.0 100.0.0.255 prefix-length 24
```

- Format: `ip nat pool [name] [first IP] [last IP] prefix-length [length]`
- Both the first and last addresses must be in the same subnet — command is rejected otherwise
- Can also use `netmask` instead of `prefix-length`

Step 4 — Map the ACL to the pool:

```
ip nat inside source list 1 pool POOL1
```

**PAT (Port Address Translation / NAT Overload)**

PAT translates both the **source IP address** and the **source port number**. This allows many internal hosts to share a single public IP address simultaneously.

How it works:

- Each communication flow is tracked using a unique source port number
- If two inside hosts happen to select the same random source port, the router assigns a different port to one of them
- When replies arrive, the router uses the port number to determine which inside host the reply belongs to
- TCP/UDP port numbers are 16 bits = over 65,000 possible ports

Difference from dynamic NAT:

- Dynamic NAT: one-to-one mappings (one inside local per inside global at a time)
- PAT: many-to-one mappings (many inside local addresses share one inside global address)
- In `show ip nat translations`, PAT will not show the one-to-one mapping entries — only the active flows with port numbers

**PAT Configuration — Method 1: Using a Pool**

Same as dynamic NAT but with the `overload` keyword added:

```
access-list 1 permit 192.168.0.0 0.0.0.255

ip nat pool POOL1 100.0.0.0 100.0.0.3 prefix-length 24

ip nat inside source list 1 pool POOL1 overload
```

- Adding `overload` enables PAT — multiple hosts can share pool addresses using unique port numbers

**PAT Configuration — Method 2: Using the Router's Interface IP (most common)**

Instead of defining a pool, the router uses its own outside interface IP address:

```
access-list 1 permit 192.168.0.0 0.0.0.255

ip nat inside source list 1 interface g0/0 overload
```

- Format: `ip nat inside source list [ACL] interface [outside interface] overload`
- All inside hosts share the single public IP address of the router's outside interface
- This is how most home routers work

**Verification Commands**

View NAT translation table:

```
show ip nat translations
```

- Static NAT: permanent entries always visible
- Dynamic NAT: dynamic mapping entries plus temporary protocol-specific entries
- PAT: only protocol-specific entries with port numbers; no one-to-one mapping entries

View NAT statistics:

```
show ip nat statistics
```

- Shows total active translations, dynamic vs static count, pool usage, inside and outside interfaces

Clear dynamic translations:

```
clear ip nat translation *
```

- Clears dynamic entries only; static entries remain

**NAT Type Comparison**

| Feature | Static NAT | Dynamic NAT | PAT (NAT Overload) |
| --- | --- | --- | --- |
| Mapping type | One-to-one (permanent) | One-to-one (temporary) | Many-to-one |
| Preserves public IPs? | No | No | Yes |
| Port translation | No | No | Yes |
| Initiated from outside? | Yes | No | No |
| Most common? | Rarely | Rarely | Yes |

**Command Summary**

| Command | Purpose |
| --- | --- |
| `ip nat inside` | Mark interface as inside |
| `ip nat outside` | Mark interface as outside |
| `ip nat pool [name] [start] [end] prefix-length [n]` | Define NAT pool |
| `ip nat inside source list [ACL] pool [name]` | Dynamic NAT mapping |
| `ip nat inside source list [ACL] pool [name] overload` | PAT using a pool |
| `ip nat inside source list [ACL] interface [int] overload` | PAT using interface IP |
| `show ip nat translations` | View translation table |
| `show ip nat statistics` | View NAT statistics |
| `clear ip nat translation *` | Clear dynamic translations |

---

# Day 46 (QoS (Part 1))

**Key Takeaways**

- IP phones use VoIP to send audio over IP networks; they have a built-in 3-port switch allowing a PC to share a single switchport
- Voice VLANs separate voice and data traffic on the same port — voice is tagged, data is untagged
- PoE allows switches to deliver DC power to devices like IP phones over the same Ethernet cable used for data
- QoS manages bandwidth, delay, jitter, and loss to prioritize sensitive traffic like voice and video
- Tail drop causes TCP global synchronization — RED and WRED help prevent this

**IP Phones and Voice VLANs**

IP phones use VoIP (Voice over IP) to send audio in IP packets over a network instead of the traditional PSTN. Each IP phone has a built-in 3-port switch: one port connects to the external switch, one connects to a PC, and one connects internally to the phone. This allows a PC and phone to share a single switchport, reducing the number of switches needed.

It is recommended to separate voice and data traffic into different VLANs. Traffic from the PC is sent untagged (access/data VLAN), while the IP phone tags its traffic with a voice VLAN ID. The switch uses CDP to inform the phone which VLAN to tag its traffic with.

Voice VLAN configuration:

`interface g0/0
  switchport mode access
  switchport access vlan 10       ← data VLAN for the PC
  switchport voice vlan 11        ← voice VLAN for the IP phone`

Even though the port carries traffic from two VLANs, it is still considered an access port, not a trunk port. Verify with `show interfaces g0/0 switchport`.

**Power over Ethernet (PoE)**

PoE allows a **Power Sourcing Equipment (PSE)** — typically a switch — to supply DC power to **Powered Devices (PDs)** such as IP phones, cameras, and wireless access points over the same Ethernet cable used for data. The switch receives AC power from an outlet, converts it to DC, and delivers it to connected devices.

When a device connects, the PSE sends low-power signals to detect whether the device needs power and how much, preventing damage from excess current. **Power policing** prevents PDs from drawing too much power.

Power policing commands:

`power inline police                  ← default: err-disables port + Syslog message
power inline police action err-disable  ← same as above
power inline police action log       ← restarts interface + Syslog (does not err-disable)`

PoE Standards:

| Standard | Name | Max Power |
| --- | --- | --- |
| Cisco ILP | Cisco proprietary | 7W |
| 802.3af | PoE (Type 1) | 15.4W |
| 802.3at | PoE+ (Type 2) | 30W |
| 802.3bt | Type 3 / Type 4 | 60W / 100W |

**QoS Introduction**

Modern networks are **converged networks** — voice, video, and data all share the same IP network. Different traffic types now compete for bandwidth, and delay-sensitive traffic like voice can suffer if not prioritized. QoS (Quality of Service) is a set of tools that gives different treatment to different types of traffic.

QoS manages four traffic characteristics:

| Characteristic | Description |
| --- | --- |
| **Bandwidth** | Overall capacity of a link (bps); QoS can reserve portions for specific traffic |
| **Delay** | Time for traffic to travel from source to destination (one-way or two-way) |
| **Jitter** | Variation in one-way delay between packets from the same application |
| **Loss** | Percentage of packets that do not reach their destination |

Recommended standards for acceptable interactive audio (voice/video calls):

- **One-way delay:** 150 ms or less
- **Jitter:** 30 ms or less
- **Loss:** 1% or less

**Queuing and Tail Drop**

When a device receives packets faster than it can forward them, packets are placed in a **queue**. By default, queued packets are forwarded in **FIFO (First In First Out)** order — no special priority given to any traffic type.

When the queue is full, new incoming packets are **dropped — this is called tail drop**. Tail drop leads to **TCP global synchronization**:

1. All TCP hosts detect packet loss and reduce their transmission rate simultaneously
2. Network becomes underutilized
3. All hosts then increase transmission rate simultaneously
4. Network becomes congested again → cycle repeats in waves

**RED and WRED**

**RED (Random Early Detection)** prevents tail drop by randomly dropping packets from select TCP flows before the queue is completely full. This causes only some flows to slow down, avoiding the synchronized waves caused by tail drop.

**WRED (Weighted Random Early Detection)** improves on RED by allowing different drop thresholds for different traffic classes. Lower-priority traffic is dropped sooner, while higher-priority traffic is protected longer. Traffic classification and more QoS mechanics are covered in Part 2.

---

# Day 47 (QoS (Part 2))

# QoS Part 2

**Key Takeaways**

- Classification identifies traffic types; marking sets field values so devices know how to prioritize
- PCP/CoS (Layer 2) can only mark traffic on trunk links or voice VLAN access ports — it cannot mark traffic beyond the local network
- DSCP (Layer 3) is preferred for marking because it travels end-to-end in the IP header
- LLQ creates a strict priority queue, giving voice/video traffic immediate forwarding with minimal delay and jitter
- Shaping buffers excess traffic; policing drops it — both are used to control traffic rates

**Classification and Marking**

Classification organizes packets into traffic classes so devices know how to treat them. Methods include ACLs (identify traffic by source/destination), NBAR (deep packet inspection up to Layer 7), and field markings in Layer 2 and Layer 3 headers.

**Marking** = setting values in PCP or DSCP fields so downstream devices can classify traffic without deep inspection.

**PCP / CoS (Layer 2)**

The **PCP (Priority Code Point)** field, also called **CoS (Class of Service)**, is a 3-bit field in the 802.1Q dot1q tag — giving 8 values (0–7).

| PCP Value | Traffic Type |
| --- | --- |
| 0 | Best effort (default) |
| 3 | Call signaling |
| 4 | Video |
| 5 | Voice |

Limitation: PCP only works where a dot1q tag exists — trunk links and voice VLAN access ports. It cannot mark traffic beyond the local switched network.

**IP ToS Byte (Layer 3)**

The **ToS (Type of Service)** byte in the IPv4 header contains two modern fields: **DSCP (6 bits)** and **ECN (2 bits)**. Previously, the first 3 bits were used for **IP Precedence (IPP)**, which worked like PCP but at Layer 3. DSCP expanded IPP from 3 bits to 6 bits, allowing 64 possible values and much greater flexibility.

**DSCP Standard Markings:**

**DF (Default Forwarding)** — best effort traffic, DSCP = 0 (000000)

**EF (Expedited Forwarding)** — low loss, low latency, low jitter; used for voice traffic, DSCP = 46 (101110)

**AF (Assured Forwarding)** — 12 values across 4 classes and 3 drop precedence levels:

- Written as **AFxy** where x = class (1–4) and y = drop precedence (1–3)
- Formula: **DSCP = 8x + 2y**
- Higher class = higher priority; higher drop precedence = more likely to be dropped during congestion
- Best: AF41 | Worst: AF13

**CS (Class Selector)** — 8 values for backward compatibility with IPP; right 3 bits = 0, DSCP = 8 × CS number (e.g., CS5 = DSCP 40)

**RFC 4594 recommendations:**

| Traffic Type | Recommended Marking |
| --- | --- |
| Voice | EF |
| Interactive video | AF4x |
| Streaming video | AF3x |
| High priority data | AF2x |
| Best effort | DF (DSCP 0) |

**Trust Boundaries**

The **trust boundary** defines where devices trust or change QoS markings. If markings are trusted, they are forwarded unchanged. If not, the device re-marks the packet according to policy.

- Best practice: trust markings from **IP phones** (move trust boundary to the phone)
- Do not trust markings from **PCs** — a user could manually mark traffic as high priority to gain unfair bandwidth
- The trust boundary is configured on the switch port, not the phone

---

**Queuing and Congestion Management**

When traffic arrives faster than it can be forwarded, packets are queued. Multiple queues allow different traffic types to be treated differently. A **scheduler** decides which queue to forward from next.

**Weighted Round-Robin** — takes packets from each queue in a cycle; higher-priority queues get more data per turn.

**CBWFQ (Class-Based Weighted Fair Queuing)** — uses weighted round-robin and guarantees each queue a minimum percentage of bandwidth during congestion.

**LLQ (Low Latency Queuing)** — designates one or more **strict priority queues**. If traffic is present in the priority queue, the scheduler always sends from it first. Ideal for voice and video. Downside: can starve lower-priority queues if always full — use policing to limit the amount of traffic allowed in the priority queue.

Within each queue, **RED or WRED** can be used to prevent tail drop by dropping lower-priority packets early before the queue fills completely.

**Shaping and Policing**

Both tools control the **rate** of traffic on a link:

| Feature | Shaping | Policing |
| --- | --- | --- |
| Action when over rate | Buffers traffic in a queue | Drops traffic |
| Burst allowed? | No | Yes (briefly) |
| Typical location | Customer router (outbound) | ISP router (inbound) |

Common use case: A customer pays for 300 Mbps on a 1 Gbps physical link. The ISP **polices** inbound traffic to 300 Mbps; the customer **shapes** outbound traffic to 300 Mbps to avoid ISP drops.

---

# Day 48 (Security Fundamentals)

**Key Takeaways**

- The CIA Triad (Confidentiality, Integrity, Availability) is the foundation of network security
- Common attacks include DoS/DDoS, spoofing, man-in-the-middle, malware, and social engineering
- Multi-factor authentication requires two or more categories: something you know, have, or are
- AAA = Authentication (verify identity), Authorization (control access), Accounting (record activity)
- RADIUS uses UDP ports 1812/1813; TACACS+ uses TCP port 49

**The CIA Triad**

| Principle | Meaning |
| --- | --- |
| **Confidentiality** | Only authorized users can access data |
| **Integrity** | Data cannot be tampered with by unauthorized users |
| **Availability** | Systems remain operational and accessible to authorized users |

**Key Security Concepts**

- **Vulnerability** — a potential weakness that could compromise the CIA of a system
- **Exploit** — something that can be used to take advantage of a vulnerability
- **Threat** — the real possibility that a vulnerability will be exploited
- **Mitigation technique** — a method used to protect against threats; applied across all devices (clients, servers, routers, firewalls, physical access points)

No system is perfectly secure — security is about reducing risk, not eliminating it.

**Common Attacks**

**Denial-of-Service (DoS) / DDoS**

- Threatens **availability**
- TCP SYN Flood: attacker sends endless SYN messages, fills target's TCP connection table, preventing legitimate connections
- Attacker spoofs source IP so SYN-ACK replies never return
- **DDoS** uses a **botnet** (many infected devices) to amplify the attack

**Spoofing Attacks**

- Using fake source IP or MAC addresses
- Example: **DHCP exhaustion** — attacker floods DHCP Discover messages with spoofed MACs, exhausting the IP pool and denying service to legitimate hosts

**Reflection/Amplification Attacks**

- Attacker spoofs the target's IP and sends small requests to a reflector (e.g. DNS/NTP server)
- Reflector sends large responses to the target, causing a DoS

**Man-in-the-Middle (ARP Spoofing/Poisoning)**

- Attacker sends a fake ARP reply to overwrite the victim's ARP table
- Victim's traffic is redirected to the attacker, who reads and/or modifies it before forwarding
- Threatens **confidentiality and integrity**

**Reconnaissance Attacks**

- Not an attack itself — used to gather information for future attacks
- Techniques: NSLOOKUP, port scanning, WHOIS queries

**Malware**

- **Virus** — infects a host program; spreads when software is shared
- **Worm** — standalone malware; spreads automatically without user interaction; can carry a harmful payload
- **Trojan Horse** — disguised as legitimate software; spreads through user interaction (e.g. email attachments)

**Social Engineering**

- Targets people, not systems — the most unpredictable vulnerability
- **Phishing** — fraudulent emails linking to fake websites to steal credentials
- **Spear phishing** — targeted phishing at a specific organization
- **Whaling** — phishing targeting high-profile individuals (executives)
- **Vishing** — voice/phone phishing
- **Smishing** — SMS-based phishing
- **Watering hole** — compromising websites the target frequently visits
- **Tailgating** — physically following an authorized person into a restricted area

**Password Attacks**

- **Guessing** — simple trial and error
- **Dictionary attack** — automated attempts using common words/passwords
- **Brute force** — tries every possible character combination

Strong password requirements: 8+ characters, uppercase and lowercase, numbers, special characters, changed regularly.

**Multi-Factor Authentication (MFA)**

Requires at least two of the following categories:

| Category | Examples |
| --- | --- |
| Something you **know** | Password, PIN |
| Something you **have** | Phone notification, badge, authenticator app |
| Something you **are** | Fingerprint, face scan, retina scan |

Two factors from the same category (e.g. two biometrics) does not qualify as MFA.

Digital certificates verify the identity of websites. A **CA (Certificate Authority)** signs certificates after receiving a **CSR (Certificate Signing Request)** from the entity.

**AAA Framework**

| Component | Definition |
| --- | --- |
| **Authentication** | Verifying the identity of a user |
| **Authorization** | Granting appropriate access and permissions |
| **Accounting** | Recording user activity on the system |

AAA servers (e.g. Cisco ISE — Identity Services Engine) support two protocols:

- **RADIUS** — open standard | UDP ports **1812** and **1813**
- **TACACS+** — Cisco proprietary | TCP port **49**

**Security Program Elements**

- **User awareness programs** — educate employees about threats; may include simulated phishing exercises
- **User training programs** — formal training on security policies, password best practices, and threat avoidance; conducted at onboarding and regularly throughout the year
- **Physical access control** — restricts entry to sensitive areas (network closets, data centers) using badge systems, biometrics, or multi-factor door locks; access can be centrally managed and revoked

---

# Day 49 (Port Security)

**Key Takeaways**

- Port security controls which source MAC addresses and how many are allowed on a switchport
- Port security can only be enabled on statically configured access or trunk ports — not dynamic auto/desirable
- Default violation mode is shutdown (err-disable); restrict discards traffic and logs it; protect silently discards
- Sticky MAC addresses are dynamically learned but saved to running-config like static entries
- Always disconnect the unauthorized device before re-enabling an err-disabled interface

**What is Port Security**

Port security is a Cisco switch feature that controls which source MAC addresses can enter a switchport. If an unauthorized MAC address sends a frame into a port-security-enabled interface, the switch takes a configured action (default: err-disable the port).

Key behaviors:

- Default maximum allowed MAC addresses = **1**
- If no MAC is manually configured, the switch dynamically learns the first MAC it sees
- Maximum can be increased (e.g. for IP phone + PC setups requiring 2 MACs)
- Dynamically learned and manually configured MACs can be combined on the same interface

**Why use it:** Prevents unauthorized device access to the network and defends against MAC flooding / DHCP exhaustion attacks that use thousands of spoofed MAC addresses.

**Enabling Port Security**

Port security requires the interface to be statically configured first:

```
switchport mode access            ← or trunk — dynamic auto/desirable not allowed
switchport port-security          ← enables port security with default settings
```

Verify with:

```
show port-security interface [int]
show port-security
```

**Violation Modes**

| Mode | Action on Unauthorized Frame | Syslog/SNMP | Violation Counter | Interface Status |
| --- | --- | --- | --- | --- |
| **Shutdown** (default) | Err-disables the port | Yes (once) | Set to 1, reset on re-enable | secure-shutdown |
| **Restrict** | Discards frame | Yes (each frame) | Incremented each frame | secure-up |
| **Protect** | Discards frame silently | No | Not incremented | secure-up |

Configure violation mode:

```
switchport port-security violation [shutdown | restrict | protect]
```

**Re-enabling an Err-Disabled Interface**

Always disconnect the unauthorized device first, then use one of these methods:

**Method 1 — Manual:**

```
interface [int]
  shutdown
  no shutdown
```

**Method 2 — ErrDisable Recovery (automatic):**

```
errdisable recovery cause psecure-violation
errdisable recovery interval [seconds]    ← default = 300 seconds
```

View recovery status: `show errdisable recovery`

**Configuring Secure MAC Addresses**

Manually specify an allowed MAC:

```
switchport port-security mac-address [MAC]
```

Set maximum allowed MAC addresses:

```
switchport port-security maximum [number]
```

**Secure MAC Address Aging**

By default, secure MAC addresses never age out (aging time = 0). To configure aging:

```
switchport port-security aging time [minutes]
switchport port-security aging type [absolute | inactivity]
switchport port-security aging static     ← also age out statically configured MACs
```

- **Absolute** — timer starts when MAC is learned; MAC removed after timer expires regardless of activity
- **Inactivity** — timer resets each time a frame is received from that MAC; ages out only if inactive

**Sticky Secure MAC Addresses**

Sticky learning allows dynamically learned MACs to be saved to the running-config automatically:

```
switchport port-security mac-address sticky
```

- Sticky MACs appear in running-config as: `switchport port-security mac-address sticky [MAC]`
- They never age out (even with static aging enabled)
- They are lost on reload unless you save: `write memory` or `copy running-config startup-config`
- Disabling sticky converts sticky MACs back to regular dynamic secure MACs
- In the MAC address table, sticky MACs show type **Static** even though they were dynamically learned

**MAC Address Table**

| MAC Type | Table Entry Type |
| --- | --- |
| Static secure (manually configured) | Static |
| Sticky secure (dynamically learned, saved) | Static |
| Dynamic secure (learned, not sticky) | Dynamic |

View secure MACs: `show mac address-table secure`

---

# Day 50 (DHCP Snooping)

# 🛡️ DHCP Snooping — CCNA Study Notes

> **Exam Topic:** 5.7 — Configure Layer 2 security features: DHCP snooping, Dynamic ARP Inspection, Port Security
**Scope:** Configuration and conceptual understanding required.
> 

---

## ⚡ Key Takeaways

- **DHCP snooping** filters DHCP messages on **untrusted ports** only — all other traffic is unaffected.
- All ports are **untrusted by default**. Uplink ports (toward network infrastructure) must be manually set as **trusted**.
- **DHCP server messages** (OFFER, ACK, NAK) received on an untrusted port are **always dropped** — no exceptions.
- **DHCP client messages** (DISCOVER, REQUEST, RELEASE, DECLINE) are **inspected** before being forwarded or dropped.
- For DISCOVER/REQUEST: switch checks that the **Ethernet source MAC = DHCP CHADDR field**. Mismatch = drop.
- For RELEASE/DECLINE: switch checks the **source IP + interface against the DHCP snooping binding table**. Mismatch = drop.
- Must enable DHCP snooping **globally** (`ip dhcp snooping`) AND **per VLAN** (`ip dhcp snooping vlan <id>`).
- **Rate limiting** on untrusted ports protects against DHCP starvation attacks — exceeding the limit err-disables the interface.
- **Option 82** is added by default when DHCP snooping is enabled. In most non-relay deployments, disable it with `no ip dhcp snooping information option`.
- The **DHCP snooping binding table** stores: MAC address, IP address, lease time, VLAN, and interface — but NOT the default gateway.

---

## 🔒 What is DHCP Snooping?

**DHCP snooping** is a Cisco switch security feature that filters DHCP messages to protect against DHCP-based attacks.

- Only affects **DHCP messages** — all other traffic is forwarded normally.
- Works by classifying switch ports as **trusted** or **untrusted**.
- All ports are **untrusted by default**.

### Trusted vs. Untrusted Ports

| Port Type | Direction | Examples | Behavior |
| --- | --- | --- | --- |
| **Trusted** | Toward network infrastructure | Uplinks to switches, routers, DHCP servers | DHCP messages forwarded without inspection |
| **Untrusted** | Toward end hosts | Ports connected to PCs, client devices | DHCP messages inspected; may be dropped |

> Network admins control infrastructure devices but not end-user devices — end-host-facing ports should remain untrusted.
> 

---

## ⚠️ DHCP Attacks DHCP Snooping Prevents

### 1. DHCP Starvation (Exhaustion) Attack

- Attacker floods the network with **spoofed DHCP DISCOVER messages** using fake source MAC addresses and CHADDR fields.
- The DHCP server's IP pool becomes exhausted — **denial of service** for legitimate clients.
- Rate limiting mitigates this even when MAC/CHADDR spoofing bypasses the basic DHCP snooping check.

### 2. DHCP Poisoning (Man-in-the-Middle) Attack

- Attacker runs a **spurious (rogue) DHCP server** that responds to client DISCOVER messages.
- Attacker's OFFER arrives first (especially when the legitimate server is remote), so clients accept it.
- Attacker sets itself as the **default gateway** in the OFFER.
- Result: all client traffic destined for external networks flows through the attacker first → traffic can be inspected or modified.
- Clients are unaware because traffic still reaches its destination.

---

## 📨 DHCP Message Types (Know Which Side Sends Each)

### Server Messages (always dropped on untrusted ports)

| Message | Purpose |
| --- | --- |
| **OFFER** | Server offers an IP address to the client |
| **ACK** | Server confirms the client's IP address assignment |
| **NAK** | Server declines a client's REQUEST (opposite of ACK) |

### Client Messages (inspected on untrusted ports)

| Message | Purpose |
| --- | --- |
| **DISCOVER** | Client broadcasts to find a DHCP server |
| **REQUEST** | Client requests a specific IP address |
| **RELEASE** | Client tells the server it no longer needs its IP address |
| **DECLINE** | Client declines the IP address offered by the server |

---

## 🔍 DHCP Snooping Inspection Logic

`DHCP message received on port
         │
         ▼
   Trusted port? ──YES──→ Forward as normal (no inspection)
         │
        NO
         ▼
   Server message? ──YES──→ DISCARD (OFFER, ACK, NAK on untrusted = always drop)
         │
        NO (client message)
         ▼
   DISCOVER or REQUEST?
   ├─ Check: Ethernet source MAC == DHCP CHADDR field?
   │    YES → Forward    NO → DISCARD
   │
   RELEASE or DECLINE?
   └─ Check: Source IP + interface match DHCP snooping binding table?
        YES → Forward    NO → DISCARD`

---

## 🗃️ DHCP Snooping Binding Table

Automatically populated when a client successfully leases an IP address. Used to validate RELEASE and DECLINE messages.

**Stored fields:**

- MAC address
- IP address
- Lease time remaining
- VLAN
- Interface

> ⚠️ The **default gateway is NOT stored** in the binding table.
> 

**Verification command:**

`show ip dhcp snooping binding`

---

## 🔧 DHCP Snooping Configuration

### Step 1 — Enable globally

`ip dhcp snooping`

### Step 2 — Enable per VLAN

`ip dhcp snooping vlan <vlan-id>`

> Both commands are required. Global alone is not sufficient.
> 

### Step 3 — Configure trusted ports (on uplink interfaces)

`interface <interface-id>
 ip dhcp snooping trust`

### Step 4 — Disable Option 82 (usually required in non-relay setups)

`no ip dhcp snooping information option`

### Verify

`show ip dhcp snooping
show ip dhcp snooping binding`

---

## ⏱️ DHCP Snooping Rate Limiting

Limits the rate of DHCP messages allowed on an interface. Protects against DHCP starvation attacks.

`interface <interface-id>
 ip dhcp snooping limit rate <packets-per-second>`

- If the rate is exceeded → interface is **err-disabled**.
- Re-enable manually:

  `shutdown
  no shutdown`

- Or configure automatic recovery:

  `errdisable recovery cause dhcp-rate-limit`

> Verify recovery status: `show errdisable recovery`
> 

---

## 📎 DHCP Option 82 (Information Option)

**Option 82** = DHCP relay agent information option. It provides the DHCP server with details about which relay agent forwarded the message, on which interface, in which VLAN.

### The Problem

When DHCP snooping is enabled, Cisco switches **add Option 82 to client DHCP messages by default** — even when NOT acting as a relay agent. This causes two issues:

| Issue | What happens |
| --- | --- |
| Downstream switch drops the message | By default, Cisco switches drop DHCP messages with Option 82 received on an **untrusted port** |
| DHCP server/router drops the message | Router drops messages with Option 82 that were not sent by a legitimate relay agent ("inconsistent relay information") |

### The Fix

`no ip dhcp snooping information option`

Apply this on **all switches** in the path when not using DHCP relay agents. Without it, DHCP may fail silently.

---

## 📋 Key Commands Reference

| Command | Purpose |
| --- | --- |
| `ip dhcp snooping` | Enable DHCP snooping globally |
| `ip dhcp snooping vlan <id>` | Enable DHCP snooping on a specific VLAN |
| `ip dhcp snooping trust` | Mark an interface as trusted (interface config mode) |
| `no ip dhcp snooping information option` | Disable Option 82 insertion |
| `ip dhcp snooping limit rate <pps>` | Set rate limit for DHCP messages on an interface |
| `errdisable recovery cause dhcp-rate-limit` | Auto-recover interfaces err-disabled by rate limiting |
| `show ip dhcp snooping` | View DHCP snooping status and trusted ports |
| `show ip dhcp snooping binding` | View the DHCP snooping binding table |
| `show errdisable recovery` | View err-disable recovery settings and timers |

---

## 📋 Key Terms

| Term | Definition |
| --- | --- |
| **DHCP snooping** | Switch security feature that filters DHCP messages on untrusted ports |
| **Trusted port** | Port that forwards DHCP messages without inspection (uplinks to infrastructure) |
| **Untrusted port** | Port that inspects DHCP messages before forwarding (toward end hosts) |
| **CHADDR** | Client Hardware Address field in a DHCP message — identifies the client's MAC address |
| **DHCP starvation** | Attack that exhausts a DHCP server's IP pool using spoofed DISCOVER messages |
| **DHCP poisoning** | Attack using a rogue DHCP server to set itself as the default gateway (man-in-the-middle) |
| **Option 82** | DHCP relay agent information option — added by default by Cisco switches when DHCP snooping is enabled |
| **Binding table** | DHCP snooping table mapping MAC + IP + VLAN + interface for active leases |
| **Err-disabled** | Interface state caused by a violation (e.g., rate limit exceeded) — requires manual or automated recovery |

---

# Day 51 (**Dynamic ARP Inspection**)

# 🔍 Dynamic ARP Inspection (DAI) — CCNA Study Notes

> **Exam Topic:** 5.7 — Configure Layer 2 security features: DHCP snooping, DAI, Port Security
**Scope:** Configuration and conceptual understanding required.
> 

---

## ⚡ Key Takeaways

- **DAI** inspects ARP messages on **untrusted ports** and validates them against the **DHCP snooping binding table** or **ARP ACLs**.
- Like DHCP snooping, all ports are **untrusted by default** — uplinks to switches/routers must be manually set as **trusted**.
- DAI validates the **sender MAC** and **sender IP** in ARP messages. If no matching entry exists in the DHCP snooping binding table → **drop**.
- DAI only needs **one enable command per VLAN**: `ip arp inspection vlan <id>` — no separate global enable required (unlike DHCP snooping).
- **ARP ACLs** allow static IP hosts (not using DHCP) to be manually permitted — otherwise DAI will drop their ARP messages.
- **Optional validation checks** (dst-mac, ip, src-mac) must all be configured in **a single command** — each new command overwrites the previous one.
- **DAI rate limiting is ON by default** at **15 pkt/s on untrusted ports** (unlike DHCP snooping, which has rate limiting OFF by default).
- DAI supports a **burst interval** for rate limiting (e.g., 45 packets per 3 seconds) — DHCP snooping only supports packets per 1 second.
- Primary attack DAI prevents: **ARP poisoning** (man-in-the-middle via gratuitous ARP).

---

## 🔄 ARP Review

### Standard ARP Exchange

- **ARP request**: broadcast (dest MAC = FF:FF:FF:FF:FF:FF) — asks "who has IP X, tell MAC Y?"
- **ARP reply**: unicast — responds with the MAC address of the requested IP.
- ARP has no IP header — sender IP, sender MAC, target IP, and target MAC are fields inside the ARP message itself.
- Both devices add entries to their ARP tables after the exchange.

### Gratuitous ARP (GARP)

- An **ARP reply sent without a prior request** — broadcast to all devices.
- Used when a device's interface comes up, IP changes, or MAC changes.
- All receiving devices **update their ARP tables** automatically with the GARP sender's IP-to-MAC mapping.
- This behavior is exploited in **ARP poisoning attacks**.

---

## ⚠️ ARP Poisoning Attack

**Goal:** Redirect traffic through the attacker via a man-in-the-middle attack.

**How it works:**

1. Attacker sends a **gratuitous ARP** using the IP address of a legitimate device (e.g., the default gateway R1).
2. All devices on the LAN receive it and **update their ARP tables**, mapping the attacker's MAC to R1's IP.
3. When PC1 wants to send traffic externally, it sends it to the attacker (thinking it's R1).
4. Attacker copies/inspects the traffic, then forwards it to the real R1.
5. Traffic reaches its destination — **PC1 is unaware of the interception**.

> R1 does not update its own ARP table because the GARP uses R1's own IP — a device won't remap its own IP.
> 

---

## 🛡️ What is DAI?

**Dynamic ARP Inspection (DAI)** is a Cisco switch security feature that inspects ARP messages on untrusted ports and validates them to prevent ARP poisoning.

- Only filters **ARP messages** — all other traffic is unaffected.
- All ports are **untrusted by default**.
- **Trusted ports**: ARP messages forwarded without inspection (uplinks to switches, routers).
- **Untrusted ports**: ARP messages inspected and validated.

### How DAI validates ARP messages

`ARP message received on port
         │
         ▼
   Trusted port? ──YES──→ Forward without inspection
         │
        NO
         ▼
   Check sender MAC + sender IP against:
   1. DHCP snooping binding table
   2. ARP ACLs (for static IP hosts)
         │
   Match found? ──YES──→ Forward
         │
        NO → DISCARD`

---

## ⚙️ DAI Configuration

### Enable DAI per VLAN (no global command needed — unlike DHCP snooping)

`ip arp inspection vlan <vlan-id>`

### Configure trusted ports (interface config mode)

`interface <interface-id>
 ip arp inspection trust`

### Key difference from DHCP snooping

| Feature | DHCP Snooping | DAI |
| --- | --- | --- |
| Global enable command | ✅ Required (`ip dhcp snooping`) | ❌ Not needed |
| Per-VLAN enable command | ✅ Required | ✅ Required |
| Total commands to enable | **2** | **1** |

---

## ⏱️ DAI Rate Limiting

> DAI rate limiting is **enabled by default on untrusted ports** at **15 pkt/s**.
DHCP snooping rate limiting is **disabled by default** on all ports.
> 

### Configure rate limiting

`interface <interface-id>
 ip arp inspection limit rate <packets> [burst interval <seconds>]`

**Examples:**

`ip arp inspection limit rate 25 burst interval 2   ! 25 packets per 2 seconds
ip arp inspection limit rate 10                     ! 10 packets per 1 second (default interval)`

- **Burst interval** = optional. Default = 1 second. Allows flexible rate measurement (e.g., 45 pkt/3s = 15 avg/s).
- If rate exceeded → interface is **err-disabled**.
- Rate limiting applies to **received** ARP messages (not sent).

### Recovery from err-disabled

`! Manual
interface <id>
 shutdown
 no shutdown

! Automatic
errdisable recovery cause arp-inspection`

### Verify

`show ip arp inspection interfaces
show errdisable recovery`

---

## 🔎 DAI Optional Validation Checks

By default, DAI only checks **sender MAC** and **sender IP** against the binding table.
Additional checks can be enabled:

| Check | What it validates |
| --- | --- |
| **dst-mac** | ARP reply: destination MAC in Ethernet header must match target MAC in ARP message |
| **ip** | Checks for invalid IPs in ARP messages: 0.0.0.0, 255.255.255.255, multicast. Sender IP checked in requests + replies; target IP checked in replies only. |
| **src-mac** | Source MAC in Ethernet header must match sender MAC in ARP message |

### ⚠️ Critical configuration rule

Each `ip arp inspection validate` command **overwrites the previous one**. To enable multiple checks, specify all of them in **a single command**:

`! ❌ WRONG — only src-mac will be active; each line overwrites the previous
ip arp inspection validate dst-mac
ip arp inspection validate ip
ip arp inspection validate src-mac

! ✅ CORRECT — all three checks are active
ip arp inspection validate ip src-mac dst-mac`

> The order of the options in the command does not matter.
> 

---

## 📋 ARP ACLs (for static IP hosts)

Hosts with **static IP addresses** won't have an entry in the DHCP snooping binding table → DAI will drop their ARP messages.

**Solution:** Configure an ARP ACL to manually permit them.

`! Step 1: Create the ARP ACL
arp access-list <name>
 permit ip host <ip-address> mac host <mac-address>

! Step 2: Apply the ARP ACL to a VLAN
ip arp inspection filter <acl-name> vlan <vlan-id>`

> Without the `static` keyword, the implicit deny at the end of the ARP ACL is **ignored** — DAI will fall back to checking the DHCP snooping binding table for non-matched entries.
With `static`, the implicit deny applies and the DHCP snooping table is NOT checked.
> 

---

## 📊 DAI vs. DHCP Snooping Comparison

| Feature | DHCP Snooping | DAI |
| --- | --- | --- |
| What it filters | DHCP messages | ARP messages |
| Enable command | Global + per VLAN | Per VLAN only |
| Rate limiting (default) | Disabled on all ports | **Enabled at 15 pkt/s on untrusted ports** |
| Rate limit unit | Packets per second only | Packets per second **or per burst interval** |
| Validation source | Checks CHADDR vs source MAC | Checks sender MAC+IP vs binding table or ARP ACL |
| Trusted port behavior | No inspection | No inspection |

---

## 📋 Key Commands Reference

| Command | Purpose |
| --- | --- |
| `ip arp inspection vlan <id>` | Enable DAI on a VLAN |
| `ip arp inspection trust` | Mark an interface as trusted (interface config mode) |
| `ip arp inspection limit rate <n> [burst interval <s>]` | Configure rate limiting |
| `ip arp inspection validate ip src-mac dst-mac` | Enable optional validation checks (all in one command) |
| `arp access-list <name>` | Create an ARP ACL |
| `permit ip host <ip> mac host <mac>` | ARP ACL permit entry |
| `ip arp inspection filter <acl> vlan <id>` | Apply ARP ACL to a VLAN |
| `errdisable recovery cause arp-inspection` | Auto-recover err-disabled interfaces |
| `show ip arp inspection` | Summary of DAI config and statistics |
| `show ip arp inspection interfaces` | Trust state and rate limits per interface |
| `show errdisable recovery` | Recovery settings and timers |

---

## 📋 Key Terms

| Term | Definition |
| --- | --- |
| **DAI** | Dynamic ARP Inspection — switch security feature that validates ARP messages on untrusted ports |
| **Gratuitous ARP (GARP)** | Unsolicited ARP reply broadcast to all devices — used legitimately for announcements, exploited in ARP poisoning |
| **ARP poisoning** | Attack where a device sends false GARP messages to redirect traffic through itself (man-in-the-middle) |
| **Trusted port** | Port that forwards ARP messages without inspection (toward network infrastructure) |
| **Untrusted port** | Port that inspects ARP messages before forwarding (toward end hosts) — default for all ports |
| **Burst interval** | DAI feature allowing rate limits over a window longer than 1 second (e.g., 45 pkt/3s) |
| **ARP ACL** | Manual mapping of IP-to-MAC addresses for static IP hosts that don't appear in the DHCP snooping binding table |
| **Sender MAC / Sender IP** | Fields in the ARP message body — primary fields DAI validates by default |

---

# Day 52 (LAN Architectures)

# 🖧 LAN Architectures — CCNA Study Notes

**Source:** Jeremy's IT Lab – Free CCNA Course
**Exam Topics:** 1.2.a, 1.2.b, 1.2.c, 1.2.e

---

## ⚡ Key Takeaways

- **2-Tier (Collapsed Core):** Access + Distribution layers; suitable for smaller LANs
- **3-Tier:** Access + Distribution + Core layers; adds a Core Layer for large LANs with 3+ distribution layers
- **Distribution layer** is the boundary between Layer 2 and Layer 3 in both designs
- **Spine-Leaf** solves east-west traffic bottlenecks in data centers; every leaf connects to every spine
- **SOHO** networks use a single device (home router) for routing, switching, firewall, and wireless
- **Core Layer** focus = speed only — no STP, no QoS marking, no security features

---

## 1. 📐 Common Topology Terms

| Term | Definition |
| --- | --- |
| **Star** | Many devices connect to one central device |
| **Full Mesh** | Every device connects to every other device |
| **Partial Mesh** | Some devices interconnected, but not all |
| **Hybrid** | Combination of the above |

> 💡 These topologies appear within and across all LAN designs — learn to recognize them on diagrams.
> 

---

## 2. 🏢 2-Tier Campus LAN Design (Collapsed Core)

> Also called **"Collapsed Core"** because the Core Layer is merged with Distribution.
> 

### Layers

### 🔵 Access Layer

- End hosts connect here (PCs, printers, IP cameras, IP phones, WAPs)
- **QoS marking** done here (mark traffic as early as possible)
- Security features: **Port Security, DAI, DHCP Snooping**
- Switchports often **PoE-enabled** (for WAPs and IP phones)
- Topology pattern: **Star** (many hosts → one access switch)

### 🟡 Distribution Layer

- Aggregates connections from access layer switches
- **Boundary between Layer 2 and Layer 3**
    - Access → Distribution links: **Layer 2** (STP running)
    - Distribution → Distribution links: **Layer 3** (OSPF, no STP)
- Runs **multilayer switching** (L2 + L3)
- End hosts use **SVIs on distribution switches** as default gateways
- **FHRP** (HSRP/VRRP) provides redundant gateway IP
- Connects to external services: **Internet, WAN**
- Also called: **Aggregation Layer**
- Topology pattern: **Partial Mesh** (access ↔ distribution)

### Key Notes

- Redundant pair of distribution switches is standard
- STP disables redundant L2 links to prevent loops
- Between distribution switches: **Full Mesh** (L3 connections)
- Suitable for **smaller LANs**

---

## 3. 🏗️ 3-Tier Campus LAN Design

> Add a **Core Layer** when there are **more than 3 distribution layers** in one location (Cisco recommendation).
> 

### Additional Layer

### 🔴 Core Layer

- Connects multiple distribution layers together across a large campus
- Focus: **"Fast Transport"** — speed above all else
- **Avoid** at this layer:
    - QoS marking/classification
    - Security features (DAI, port security, etc.)
    - Spanning Tree Protocol (**STP**)
- All connections are **Layer 3**
- Redundancy is critical — Core is the backbone of the LAN
- In 3-tier design, Core switches connect to Internet (not Distribution)

### Layer Summary Table

| Layer | Hosts? | L2/L3 | Key Features |
| --- | --- | --- | --- |
| **Access** | ✅ Yes | L2 | QoS marking, Port Security, DAI, PoE |
| **Distribution** | ❌ No | L2 + L3 | Aggregation, SVI gateways, FHRP, OSPF |
| **Core** | ❌ No | L3 only | Fast forwarding, no security/QoS/STP |

---

## 4. 🗄️ Spine-Leaf Architecture (Data Center)

> Designed to solve **east-west traffic** bottlenecks in modern data centers.
> 

### Background

- Traditional 3-tier worked well for **north-south traffic** (host → Internet)
- **Virtualization** increased **east-west traffic** (server ↔ server within the DC)
- 3-tier caused **bandwidth bottlenecks** and **variable latency** for east-west flows
- Also called: **Clos Architecture**

### Rules of Spine-Leaf

- ✅ Every **leaf switch connects to every spine switch**
- ✅ Every **spine switch connects to every leaf switch**
- ❌ **Leaf switches do NOT connect to other leaf switches**
- ❌ **Spine switches do NOT connect to other spine switches**
- ✅ **End hosts (servers) connect only to leaf switches**

### Key Characteristics

- **Consistent latency:** Every server-to-server path crosses the same number of hops (except same-leaf)
- **Load balancing:** Traffic path is randomly chosen across spine switches
- **Easy to scale:** Add a new leaf switch → connect to all spines → done
- Leaf switches = "access layer" equivalent

### Traffic Path Example

`Server A → Leaf 1 → Spine X → Leaf 2 → Server B  (3 hops)
Server A → Leaf 1 → Spine Y → Leaf 3 → Server C  (3 hops)`

---

## 5. 🏠 SOHO Networks (Small Office/Home Office)

- Applies to: small company offices, home networks with Internet access
- **Single device** handles all network functions:
    - **Router** — connects to Internet
    - **Switch** — LAN ports for wired devices
    - **Firewall** — blocks inbound connections, allows outbound
    - **Wireless Access Point** — WiFi for phones, laptops
    - *(Sometimes)* **Modem** — for cable Internet
- Device commonly called: **home router** or **wireless router**
- No need for dedicated per-function devices at this scale

---

## 6. 📝 Quiz Answers (Self-Test)

| # | Question | Answer |
| --- | --- | --- |
| 1 | Layer that serves as the L2/L3 boundary? | **Distribution** |
| 2 | What would you NOT find in the Core Layer? | **STP (Spanning Tree Protocol)** |
| 3 | Where would you find PoE-enabled ports? | **Access Layer** |
| 4 | What should NOT connect to a leaf switch? | **Another Leaf Switch** |
| 5 | What functions does a wireless/home router include? | **Routing, Switching, Wireless, Security (Firewall), sometimes Modem** |

---

## 7. 🔑 Critical Reminders

> "The answer to most general questions about network design is **'it depends'**" — requirements differ per network.
> 
- **2-tier** = Access + Distribution (collapsed core)
- **3-tier** = Access + Distribution + Core (add core when > 3 distribution layers)
- **Spine-Leaf** = data center design for east-west traffic, consistent latency, easy scaling
- **SOHO** = one device does everything
- Distribution layer also called: **Aggregation Layer**
- Core Layer goal: **fast transport, nothing else**

---

# Day 53 (WAN Architectures)

# 🌐 WAN Architectures — CCNA Study Notes

**Source:** Jeremy's IT Lab – Free CCNA Course
**Exam Topics:** 1.2.d (WAN) · 5.5 (Remote Access & Site-to-Site VPNs)

> ℹ️ These topics use the term **"describe"** — no configurations required, only conceptual understanding.
> 

---

## ⚡ Key Takeaways

- A **WAN** connects geographically separate LANs (e.g., offices in different cities/countries)
- **Leased lines** = dedicated private physical connections using serial cables (HDLC/PPP); being replaced by Ethernet fiber
- **MPLS** uses **labels** (not IP addresses) to forward traffic; supports L2 and L3 VPN types
- **CE routers do NOT run MPLS** — only PE and P routers do
- Internet connections have four redundancy levels: **Single Homed → Dual Homed → Multihomed → Dual Multihomed**
- **Site-to-site VPNs** use **IPsec**; connect two entire sites
- **Remote-access VPNs** use **TLS**; connect one end device to a site
- **GRE over IPsec** = flexibility (multicast support) + security (encryption)
- **DMVPN** = dynamic full-mesh IPsec tunnels with hub-and-spoke simplicity

---

## 1. 🗺️ WAN Introduction

- **WAN (Wide Area Network):** A network spanning large geographic areas (cities, countries)
- WANs connect geographically separate **LANs** together
- The term "WAN" typically refers to an **enterprise's private connections** — not the public Internet
- The Internet *can* be used as a WAN, but requires **VPNs** to ensure privacy over the shared network

### Hub-and-Spoke Topology

- Common WAN topology where branch offices (**spokes**) all connect to a central site (**hub**, e.g., data center)
- **Advantage:** Centralized traffic control (e.g., a single firewall at the hub)
- Equivalent to a **star topology** in LAN terminology

---

## 2. 📡 Leased Lines

- A **dedicated physical link** connecting two sites — not shared, not Internet
- Use **serial connections** with **PPP** or **HDLC** encapsulation (not Ethernet)
- Speeds vary by standard and region:

| Region | Standards |
| --- | --- |
| **North America** | T1 (1.544 Mbps), T2, T3 |
| **Europe / Others** | E1, E2, E3 |

### Why Leased Lines Are Being Replaced

- **Higher cost** than modern alternatives
- **Longer installation lead time**
- **Slower speeds** compared to Ethernet fiber options

---

## 3. 🏷️ MPLS (Multi Protocol Label Switching)

- **Shared infrastructure** — multiple customer enterprises share the same provider network
- Uses **labels** (inserted between L2 and L3 headers) to forward traffic and separate customers
- Sometimes called a **"Layer 2.5 protocol"** due to label placement

### Router Types

| Term | Full Name | Role |
| --- | --- | --- |
| **CE** | Customer Edge | Customer's router; connects to PE; does **NOT** run MPLS |
| **PE** | Provider Edge | Provider's edge router; adds/removes MPLS labels |
| **P** | Provider Core | Internal provider routers; forward based on labels only |

### MPLS VPN Types

### 🔵 Layer 3 MPLS VPN

- CE and PE routers form **dynamic routing protocol peerings** (e.g., OSPF)
- CE routers learn remote routes via the PE routers
- CE routers use PE routers as next-hop for static routes

### 🟡 Layer 2 MPLS VPN

- CE and PE routers do **NOT** form peerings
- The entire provider network is **transparent** to CE routers
- Acts like a **big switch** connecting CE routers directly
- CE routers peer **directly with each other** (e.g., OSPF between CE routers)

### Connecting to MPLS

Multiple access technologies can connect a site to an MPLS provider:

- Fiber optic Ethernet
- Wireless 4G/5G
- CATV (cable)
- Serial leased line

---

## 4. 🌍 Internet Connections

### Common Access Technologies

### DSL (Digital Subscriber Line)

- Delivers Internet over existing **phone lines**
- Requires a **modem** (modulator-demodulator) to convert data for phone line transmission
- Modem can be separate or built into the home router

### Cable Internet

- Delivers Internet over existing **CATV (cable TV) lines**
- Requires a **cable modem**
- Modem can be separate or built into the home router

### Redundancy Levels

| Type | Connections | ISPs | Redundancy |
| --- | --- | --- | --- |
| **Single Homed** | 1 | 1 | ❌ None |
| **Dual Homed** | 2 | 1 | ⚠️ Limited |
| **Multihomed** | 1 each | 2 | ✅ Good |
| **Dual Multihomed** | 2 each | 2 | ✅✅ Best |

---

## 5. 🔒 Internet VPNs

> The Internet is a **shared, public network** — VPNs provide security by encrypting traffic.
> 

### Site-to-Site VPNs (IPsec)

- Connects **two entire sites** over the Internet
- A **VPN tunnel** is created between two routers/firewalls
- Process:
    1. Router receives packet destined for remote site
    2. **Encrypts** the original packet using IPsec + session key
    3. Adds **VPN header + new IP header**
    4. Sends encrypted packet over the Internet
    5. Receiving router **decrypts** and forwards to destination
- End hosts send unencrypted traffic to their local router — **the router handles encryption**

### ⚠️ Limitations of Standard IPsec

- **No broadcast or multicast support** → routing protocols (e.g., OSPF) cannot run over tunnels
- **Scaling is labor-intensive** → configuring full-mesh tunnels between many sites is complex

---

### GRE over IPsec

| Feature | GRE | IPsec | GRE over IPsec |
| --- | --- | --- | --- |
| Encryption | ❌ | ✅ | ✅ |
| Multicast/Broadcast | ✅ | ❌ | ✅ |
- **GRE** encapsulates the original packet → adds GRE header + new IP header
- **IPsec** then encrypts the GRE packet → adds IPsec header + new IP header
- Result: **flexibility of GRE + security of IPsec**

---

### DMVPN (Dynamic Multipoint VPN)

- Cisco-developed solution for scaling IPsec VPNs
- **Step 1:** Configure IPsec tunnels only to a **hub router** (hub-and-spoke)
- **Step 2:** Hub shares information so spokes can **dynamically form tunnels with each other**
- Result: **Full mesh of tunnels** built automatically

| Benefit | Description |
| --- | --- |
| **Configuration simplicity** | Only configure one tunnel per router (to hub) |
| **Spoke-to-spoke efficiency** | Traffic goes directly between spokes without hitting the hub |

---

### Remote-Access VPNs (TLS)

- Connects **one end device** (PC, phone, laptop) to a company's internal network
- Uses **TLS (Transport Layer Security)** — same protocol behind HTTPS
    - Formerly known as **SSL (Secure Sockets Layer)**
- Requires **VPN client software** installed on the end device (e.g., **Cisco AnyConnect**)
- End device forms a **TLS tunnel** to the company's router or firewall
- Used for **on-demand** secure remote access (e.g., working from home)

---

## 6. ⚖️ Site-to-Site vs. Remote-Access VPN Comparison

| Feature | Site-to-Site VPN | Remote-Access VPN |
| --- | --- | --- |
| **Protocol** | IPsec | TLS |
| **Scope** | Connects two entire sites | Connects one device to a site |
| **Who configures VPN?** | Routers/firewalls | End device (VPN client software) |
| **Use case** | Permanent inter-site connection | On-demand remote access |
| **Traffic coverage** | All hosts in both sites | Only the device with the VPN client |

---

## 7. 📝 Quiz Answers (Self-Test)

| # | Question | Answer |
| --- | --- | --- |
| 1 | Which leased line standard provides 1.544 Mbps? | **T1** |
| 2 | Which router type does NOT run MPLS? | **CE (Customer Edge)** |
| 3 | Which MPLS VPN type allows CE routers to peer directly with each other via OSPF? | **Layer 2 MPLS VPN** |
| 4 | Which Internet access technology uses phone lines? | **DSL** |
| 5 | Which protocol combined with IPsec allows multicast traffic in a tunnel? | **GRE** |

---

## 8. 🔑 Critical Reminders

- **MPLS forwards based on labels, NOT destination IP addresses**
- **CE routers do NOT run MPLS** — only PE and P routers do
- **Layer 2 MPLS VPN** = provider network acts like a transparent switch; CE routers peer directly
- **Layer 3 MPLS VPN** = CE routers peer with PE routers using a routing protocol
- **GRE** = flexible but not secure; **IPsec** = secure but no multicast → combine them with **GRE over IPsec**
- **DMVPN** = hub-and-spoke config simplicity + full-mesh spoke-to-spoke communication
- **Site-to-site = IPsec · Remote-access = TLS**

---

# Day 54 (Virtualization & Cloud (Part 1))

# ☁️ Virtualization & Cloud Computing — CCNA Study Notes

**Source:** Jeremy's IT Lab – Free CCNA Course
**Exam Topics:** 1.2.f (On-Premises & Cloud) · 1.12 (Virtualization Fundamentals / Virtual Machines)

---

## ⚡ Key Takeaways

- **Type 1 hypervisor** runs directly on hardware (bare-metal); used in data centers — e.g., VMware ESXi, Microsoft Hyper-V
- **Type 2 hypervisor** runs on a host OS; used on personal devices — e.g., VMware Workstation, Oracle VirtualBox
- Cloud has **5 essential characteristics**, **3 service models**, and **4 deployment models** — know all of them
- **SaaS** = provider controls everything, customer uses the app; **PaaS** = customer deploys apps on provider's platform; **IaaS** = customer controls OS, storage, apps on provider infrastructure
- **Private cloud ≠ on-premises only** — private clouds can exist off-premises (e.g., AWS GovCloud for the DoD)
- Cloud connects to enterprise networks via **private WAN (e.g., MPLS)** or **Internet + VPN**

---

## 1. 🖥️ Servers Before Virtualization

- Traditional model: **one physical server = one operating system**
- Each app (web server, email server, DB server) ran on its own dedicated physical machine
- **Problems with this model:**
    - Expensive hardware, consumes space/power/cooling
    - Hardware resources (CPU, RAM, storage, NIC) are typically **under-utilized**
    - Running all apps on one OS creates risk — a fault in one app can affect all others

---

## 2. ⚙️ Virtualization

> Virtualization breaks the one-to-one relationship of hardware to OS, allowing **multiple OSes to run on a single physical server**.
> 
- Each OS instance = a **VM (Virtual Machine)**
- A **hypervisor** (also called a **VMM — Virtual Machine Monitor**) manages and allocates hardware resources (CPU, RAM) to each VM

### Type 1 Hypervisor — "Bare-Metal" / "Native"

- Runs **directly on top of the hardware**
- No host OS required
- **Examples:** VMware ESXi, Microsoft Hyper-V
- Used in **data center environments**
- More efficient — fewer resources consumed by the hypervisor itself

### Type 2 Hypervisor — "Hosted"

- Runs **as a program on a host OS**
- **Examples:** VMware Workstation, Oracle VirtualBox
- Used on **personal devices** (Mac, Linux, Windows PC)
- Common use case: running a Windows VM on a Mac, or vice versa
- **Host OS** = OS running directly on the hardware
- **Guest OS** = OS running inside the VM

### Hypervisor Comparison

| Feature | Type 1 | Type 2 |
| --- | --- | --- |
| Runs on | Hardware directly | Host OS |
| Also called | Bare-metal / Native | Hosted |
| Use case | Data centers | Personal devices |
| Examples | VMware ESXi, Hyper-V | VMware Workstation, VirtualBox |
| Efficiency | ✅ Higher | ⚠️ Lower |

### Why Use Virtualization?

| Benefit | Explanation |
| --- | --- |
| **Partitioning** | Run multiple OSes on one physical machine |
| **Isolation** | Fault in one VM doesn't affect others |
| **Encapsulation** | Save, copy, and move VMs like files |
| **Hardware independence** | Run any VM on any compatible physical server |
| **Reduced CapEx** | Fewer physical servers needed |
| **Reduced OpEx** | Less space, power, cooling, and setup time |
| **Reduced downtime** | Deploy VMs to multiple servers for redundancy |
| **Speed & agility** | New VMs provisioned in minutes vs. days/weeks for physical servers |

### Connecting VMs to the Network

- VMs connect to each other and the external network via a **virtual switch (vSwitch)** running on the hypervisor
- vSwitch interfaces can operate as **access or trunk ports** and use **VLANs** to separate VMs at Layer 2
- The vSwitch connects to the physical **NIC(s)** of the server to reach the external network
- **vPC (Virtual Port Channel)** can bond two physical NICs to two separate switches for redundancy

---

## 3. ☁️ Cloud Computing

### Traditional IT Deployment Models (Pre-Cloud)

- **On-premises:** All infrastructure is located on company property, purchased and owned by the company
- **Colocation:** Company rents space in a third-party data center; company still owns the equipment, but the data center provides space, power, cooling, and physical security

### NIST Definition

> *"Cloud computing is a model for enabling ubiquitous, convenient, on-demand network access to a shared pool of configurable computing resources that can be rapidly provisioned and released with minimal management effort or service provider interaction."*
— NIST SP 800-145
> 

Cloud is defined by: **5 Essential Characteristics + 3 Service Models + 4 Deployment Models**

---

## 4. ✅ Five Essential Characteristics of Cloud

> A service must have **all five** to be considered true cloud computing.
> 

| Characteristic | Summary |
| --- | --- |
| **On-Demand Self-Service** | Customers provision resources themselves via a web portal — no need to contact the provider |
| **Broad Network Access** | Services accessible over standard network connections (Internet, WAN) from any device |
| **Resource Pooling** | Provider's resources are shared across multiple customers (multi-tenant); customers don't control exact physical location |
| **Rapid Elasticity** | Resources can be scaled up or down quickly — appears unlimited to the customer |
| **Measured Service** | Usage is monitored and metered; customers are charged based on consumption; full visibility into usage |

---

## 5. 🛠️ Three Service Models of Cloud

### SaaS — Software as a Service

- Customer uses the **provider's application** over the cloud
- Customer has **no control** over infrastructure, OS, or app (only limited user settings)
- **Examples:** Microsoft Office 365, Google Workspace (Gmail, Docs)
- Provider controls: data center → network → servers → OS → tools → **app**

### PaaS — Platform as a Service

- Provider offers a **platform for developers** to build and deploy their own applications
- Customer controls: deployed apps and some environment settings
- Provider controls: data center → network → servers → OS → tools (but NOT the customer's hosted apps)
- **Examples:** AWS Lambda, Google App Engine

### IaaS — Infrastructure as a Service

- Provider offers **fundamental computing resources** (compute, storage, networking)
- Customer controls: OS, storage, deployed applications, and some networking (e.g., host firewalls)
- Provider controls: physical data center, network/security infrastructure, servers and storage
- **Examples:** Amazon EC2, Google Compute Engine
- Offers the **most control to the customer**

### Service Model Comparison

| Model | Provider Controls | Customer Controls | Examples |
| --- | --- | --- | --- |
| **SaaS** | Everything (infra → app) | App usage only | Office 365, Gmail |
| **PaaS** | Infra + OS + tools | App deployment | AWS Lambda, Google App Engine |
| **IaaS** | Physical infra only | OS, storage, apps | Amazon EC2, Google Compute Engine |

---

## 6. 🌍 Four Deployment Models of Cloud

### Private Cloud

- Infrastructure reserved for **exclusive use by a single organization**
- May be owned/operated by the organization or a **third party**
- Can exist **on or off premises**
- Example: AWS GovCloud — Amazon's infrastructure reserved for the U.S. Department of Defense

### Community Cloud

- Infrastructure reserved for **a specific group of organizations** with shared concerns (e.g., compliance, security requirements)
- Least common deployment model
- May be on or off premises

### Public Cloud

- Infrastructure available for **open use by the general public**
- Most common deployment model
- Exists **on the cloud provider's premises**
- **Major providers:** AWS, Microsoft Azure, Google Cloud (GCP), Oracle Cloud (OCI), IBM Cloud, Alibaba Cloud

### Hybrid Cloud

- A **combination of two or more** cloud deployment types (private + public, etc.)
- Bound together by standardized or proprietary technology
- Example: a private cloud that **bursts to a public cloud** when it runs low on resources

### Deployment Model Summary

| Type | Who Uses It | Location | Common? |
| --- | --- | --- | --- |
| **Private** | Single organization | On or off premises | Large enterprises / gov |
| **Community** | Group of organizations | On or off premises | Rare |
| **Public** | General public | Cloud provider's premises | ✅ Most common |
| **Hybrid** | Mixed | Mixed | ✅ Very common |

---

## 7. 💡 Benefits of Cloud Computing

| Benefit | Detail |
| --- | --- |
| **Cost** | Shifts from CapEx (hardware purchases) to OpEx (ongoing usage fees); often reduces total cost |
| **Global scale** | Services can be deployed close to end users geographically |
| **Speed & agility** | Resources provisioned in minutes on demand |
| **Productivity** | Eliminates time-consuming tasks: ordering, racking, cabling, OS installs |
| **Reliability** | Easy backups; data mirrored across geographic locations for disaster recovery |

> ⚠️ Cloud is not always the best option. Most companies use a mix of **on-premises, colocation, and public cloud**. Evaluate benefits and drawbacks before deciding.
> 

---

## 8. 🔗 Connecting to Cloud Resources

Two primary options for connecting an enterprise to public cloud resources:

- **Private WAN** (e.g., MPLS VPN) — more secure, dedicated connection
- **Internet** — cheap and flexible, but less secure; use a **VPN** to secure traffic over the Internet

> Best practice: use **redundant connections** to cloud resources — a single point of failure is never acceptable.
> 

---

## 9. 📝 Quiz Answers (Self-Test)

| # | Question | Answer |
| --- | --- | --- |
| 1 | What is the main role of a hypervisor? | **Manages and allocates hardware resources to VMs** |
| 2 | Which hypervisor type is called a "native hypervisor"? | **Type 1** |
| 3 | Which is NOT an essential characteristic of cloud? | **Infinite resource pool** (the pool appears infinite but isn't) |
| 4 | Which service model lets customers use apps on the provider's cloud infrastructure? | **SaaS** |
| 5 | Which deployment models can exist off-premises? | **All of them** (public, private, community, hybrid) |

---

## 10. 🔑 Critical Reminders

- **Type 1 = bare-metal / native** — runs on hardware, used in data centers
- **Type 2 = hosted** — runs on a host OS, used on personal devices
- Cloud's **5 characteristics:** On-demand self-service, Broad network access, Resource pooling, Rapid elasticity, Measured service
- Cloud's **3 service models:** SaaS → PaaS → IaaS (increasing customer control)
- Cloud's **4 deployment models:** Private, Community, Public, Hybrid
- **Private cloud ≠ on-premises** — private clouds can be hosted off-premises by a third party
- **IaaS = most customer control; SaaS = least customer control**

---

# Day 54 (Containers (Part 2))

# 📦 Containers — CCNA Study Notes

**Source:** Jeremy's IT Lab – Free CCNA Course
**Exam Topic:** Virtualization Fundamentals (Containers)

---

## ⚡ Key Takeaways

- Containers are **software packages** containing an app + all its dependencies — no OS per container
- Containers run on a **Container Engine** (e.g., Docker Engine) on top of a **host OS**
- The critical difference from VMs: **VMs run their own OS per instance; containers share one host OS**
- Containers are **faster, smaller, and more portable** than VMs; VMs are **more isolated and secure**
- **Docker** = most popular container engine; **Kubernetes** = most popular container orchestrator
- **Microservice Architecture** = large apps broken into small services, each running in its own container
- Container orchestrators (Kubernetes, Docker Swarm) automate deployment and scaling of containers at scale

---

## 1. 🖥️ Virtual Machines — Quick Review

### Without Virtualization

- One OS runs on the physical server; all apps share that OS
- Apps are **not isolated** — a fault in one can affect all others
- Buying a separate physical server per app = extremely cost-inefficient

### With VMs

- Multiple OSes run on a single physical server via a **hypervisor**
- Each app runs in its own isolated VM with its own guest OS
- **Binaries and libraries** needed by an app are contained within the VM

### Hypervisor Types (recap)

| Type | Also Known As | Runs On | Common Use |
| --- | --- | --- | --- |
| **Type 1** | Native / Bare-metal | Directly on hardware | Data centers |
| **Type 2** | Hosted | On top of a Host OS | Personal devices / labs |

---

## 2. 📦 Containers

### What Is a Container?

- A **software package** that contains:
    - An **app**
    - All **dependencies** (binaries and libraries) the app needs to run
- Containers do **not** include their own OS — this is the key differentiator from VMs
- Generally: **one container = one app**

### Container Architecture (bottom to top)

`[ Hardware ]
    ↑
[ Host OS ] (usually Linux)
    ↑
[ Container Engine ] (e.g., Docker Engine)
    ↑
[ Container ] [ Container ] [ Container ]`

### Container Engine

- Software that runs and manages containers
- Most popular: **Docker Engine**

### Container Orchestrator

- Automates **deployment, management, and scaling** of containers
- Required when managing large numbers of containers (e.g., thousands)
- **Examples:**
    - **Kubernetes** — most popular
    - **Docker Swarm** — Docker's native orchestrator

### Microservice Architecture

- Approach where a large application is divided into many small, independent services called **microservices**
- Each microservice runs in its own container
- Containers are orchestrated by platforms like Kubernetes
- Enables large-scale, flexible, automated systems

---

## 3. ⚖️ VMs vs. Containers

> All differences stem from one core distinction: **VMs run their own OS; containers share a host OS.**
> 

| Feature | VMs | Containers |
| --- | --- | --- |
| **OS per instance** | ✅ Yes (guest OS per VM) | ❌ No (shared host OS) |
| **Boot time** | Minutes | Milliseconds |
| **Disk space** | Tens of **GB** | Tens of **MB** |
| **CPU/RAM usage** | Higher | Lower |
| **Isolation** | ✅ Stronger (separate OS per VM) | ⚠️ Weaker (shared OS) |
| **Security** | ✅ Better (OS-level isolation) | ⚠️ Host OS crash affects all containers |
| **Portability** | Good (within same hypervisor) | ✅ Better (Docker runs on nearly any container service) |
| **Use case** | Traditional enterprise workloads | Microservices, DevOps, cloud-native apps |

### When to Use VMs

- When strong **isolation** and **security** between workloads is required
- Running different OSes on the same hardware
- Traditional enterprise environments

### When to Use Containers

- **Microservices** and cloud-native application architectures
- **DevOps** workflows requiring rapid deployment and scaling
- When resource efficiency and fast boot times are priorities

---

## 4. 🔑 Key Terms

| Term | Definition |
| --- | --- |
| **Container** | Lightweight software package with an app and its dependencies; no OS included |
| **Container Engine** | Software platform containers run on (e.g., Docker Engine) |
| **Container Orchestrator** | Automates container deployment and scaling (e.g., Kubernetes, Docker Swarm) |
| **Microservice** | A small, independent component of a larger application, typically run in its own container |
| **DevOps** | Combination of software development and IT operations; containers are central to modern DevOps |
| **Docker** | Most popular container engine |
| **Kubernetes** | Most popular container orchestration platform |

---

## 5. 📝 Quiz Answers (Self-Test)

| # | Question | Answer |
| --- | --- | --- |
| 1 | What three components do containers run on top of? | **Hardware → Host OS → Container Engine** |
| 2 | Which are container orchestrators? (pick 2) | **Kubernetes** and **Docker Swarm** |
| 3 | Which statements about VMs vs containers are true? (pick 3) | VMs require **more resources** than containers · VMs are **more isolated** than containers · Containers run on a **host OS with a container engine** |

---

## 6. 🔑 Critical Reminders

- **Containers ≠ VMs** — the absence of a per-instance OS is the defining difference
- **Docker Engine** = container engine (not an orchestrator)
- **Docker Swarm & Kubernetes** = orchestrators
- **Hyper-V** = Type 1 hypervisor by Microsoft — not related to containers
- Containers are **not replacing VMs** — both are widely used; containers are growing due to microservices and DevOps trends
- If the **host OS crashes**, all containers on it are affected — this is the main isolation weakness of containers vs VMs

---

# Day 54 (VRF (Part 3))

# 🔀 Virtual Routing and Forwarding (VRF) — CCNA Study Notes

**Source:** Jeremy's IT Lab – Free CCNA Course
**Exam Topic:** Virtualization Fundamentals (VRF)

---

## ⚡ Key Takeaways

- **VRF** divides one physical router into multiple virtual routers, each with its own **separate routing table**
- Think of VRF as **"VLANs for routers"** — VLANs divide switches into virtual LANs; VRF divides routers into virtual routers
- Traffic in one VRF **cannot be forwarded** out of an interface in a different VRF (unless VRF Leaking is configured)
- The main use cases: **service provider customer isolation** and allowing **overlapping IP addresses** on the same router
- What's covered here is technically **VRF-lite** — VRF without MPLS
- VRF is a **Layer 3 concept** — applies to router interfaces, SVIs, and routed ports on multilayer switches only
- When assigning an interface to a VRF, **any existing IP address is removed** and must be re-configured
- To view a VRF's routing table: `show ip route vrf <NAME>`; to ping within a VRF: `ping vrf <NAME> <destination>`

---

## 1. 🧠 VRF Concept

### What Is VRF?

- **Virtual Routing and Forwarding (VRF)** creates multiple independent virtual routers inside a single physical router
- Each VRF has its own **separate routing table** (also called a VRF Instance)
- By default, all router interfaces share one routing table — any interface can forward traffic to any other
- With VRF, interfaces are assigned to specific VRF instances and **traffic cannot cross between VRFs**

### VRF vs. VLANs Analogy

| Concept | Applies To | What It Divides | Creates Separate... |
| --- | --- | --- | --- |
| **VLAN** | Switches | One switch → multiple virtual switches | Broadcast domains |
| **VRF** | Routers | One router → multiple virtual routers | Routing tables |

> ⚠️ VRF does **not** create separate broadcast domains — router interfaces are already in separate broadcast domains by default.
> 

### What VRF Applies To (Layer 3 only)

- ✅ Router interfaces
- ✅ SVIs (Switched Virtual Interfaces) on multilayer switches
- ✅ Routed ports on multilayer switches
- ❌ Layer 2 switch interfaces

### VRF Leaking

- By default, VRFs are fully isolated from each other
- **VRF Leaking** can be configured to allow traffic to pass between VRFs
- Advanced concept — not covered at CCNA level

---

## 2. 🏢 Why Use VRF?

### Use Case 1 — Service Provider Customer Isolation

- A single service provider router can serve **multiple customers**
- Each customer connects to its own VRF instance (its own virtual router)
- Customer traffic is fully **isolated** from other customers on the same physical device

### Use Case 2 — Overlapping IP Addresses

- Without VRF: two interfaces on the same router **cannot be in the same subnet** — IOS will reject the configuration
- With VRF: interfaces in **different VRFs can use the same IP addresses/subnets** without conflict
- Each VRF maintains its own routing table, so overlapping addresses are resolved within the correct context

---

## 3. ⚙️ VRF Configuration

### Step 1 — Create VRFs

`R1(config)# ip vrf CUSTOMER1
R1(config)# ip vrf CUSTOMER2`

### Step 2 — Assign Interfaces to VRFs

`R1(config-if)# ip vrf forwarding CUSTOMER1`

> ⚠️ **Important:** Assigning an interface to a VRF **removes any existing IP address**. Always re-configure the IP address after assigning the VRF.
> 

**Recommended order:**

1. Assign interface to VRF (`ip vrf forwarding <NAME>`)
2. Then configure the IP address (`ip address <IP> <MASK>`)

### Step 3 — Verify VRFs

`R1# show ip vrf`

- Displays all VRF instances and their assigned interfaces

---

## 4. 📋 VRF Routing Tables

### Viewing Routing Tables

| Command | What It Shows |
| --- | --- |
| `show ip route` | **Global routing table** (interfaces NOT in any VRF) |
| `show ip route vrf CUSTOMER1` | Routing table for the **CUSTOMER1** VRF |
| `show ip route vrf CUSTOMER2` | Routing table for the **CUSTOMER2** VRF |

> 💡 If **all** interfaces are assigned to VRFs, the global routing table will be **empty**.
> 

### Key Rules

- Interfaces in a VRF → routes appear in that **VRF's routing table**
- Interfaces NOT in any VRF → routes appear in the **global routing table**
- Interfaces in different VRFs are **isolated from each other**, just like interfaces in a VRF are isolated from the global table

---

## 5. 🔁 Pinging Across VRFs

### Standard Ping (uses global routing table)

`R1# ping 192.168.1.2`

- If all interfaces are in VRFs, the global table is empty → **ping will fail**

### VRF-Aware Ping

`R1# ping vrf CUSTOMER1 192.168.1.2`

- Sends the ping using the **CUSTOMER1** VRF's routing table
- Will reach the device in CUSTOMER1, not a device with the same IP in CUSTOMER2

---

## 6. 🔑 Key Terms

| Term | Definition |
| --- | --- |
| **VRF (VRF-lite)** | Divides a router into multiple virtual routers, each with its own routing table; without MPLS |
| **VRF Instance** | A single virtual router created by VRF configuration |
| **Global Routing Table** | The default routing table used by interfaces not assigned to any VRF |
| **VRF Leaking** | Advanced config that allows traffic to pass between VRFs |
| **Overlapping IPs** | Multiple interfaces using the same subnet — only possible when each is in a different VRF |

---

## 7. 📝 Quiz Answers (Self-Test)

| # | Question | Answer |
| --- | --- | --- |
| 1 | G0/0 had an IP address, then `ip vrf forwarding VRF1` was run. Now G0/0 has no IP — why? | **`ip vrf forwarding` removes the existing IP address; it must be re-configured after assigning the VRF** |
| 2 | All of R1's interfaces are in VRFs. You run `ping 192.168.1.10` — which device responds? | **No device** — the global routing table is empty; ping cannot be sent without specifying a VRF |
| 3 | Which statements about VLANs and VRFs are true? (3 correct) | VRFs create **separate routing tables** · VRFs divide routers into **separate virtual routers** · Router interfaces in different VRFs **can share the same IP address** |

---

## 8. 🔑 Critical Reminders

- **VRF = separate routing tables** per virtual router instance
- **VLANs = separate broadcast domains** per virtual switch — do NOT confuse these
- VLANs do **not** create separate MAC address tables — the switch still has one unified MAC table
- VRF can be configured on **multilayer switch SVIs and routed ports** — not just physical routers
- **Always configure IP address AFTER assigning to a VRF** — the assignment wipes any existing address
- A ping or route lookup without specifying a VRF uses the **global routing table**
- What this video covers = **VRF-lite** (VRF without MPLS) — full VRF with MPLS is a CCNP+ topic

---

# Day 55 (Wireless Fundamentals)

# 📡 Wireless Networks Fundamentals — CCNA Study Notes

**Source:** Jeremy's IT Lab – Free CCNA Course
**Exam Topics:** 1.1.d (Access Points) · 1.11.a (Non-overlapping Wi-Fi Channels) · 1.11.b (SSID) · 1.11.c (RF)

---

## ⚡ Key Takeaways

- Wireless LANs use **IEEE 802.11** standards; "Wi-Fi" is a trademark of the Wi-Fi Alliance (certifies 802.11 compliance)
- Wireless uses **CSMA/CA** (Collision Avoidance) — not CSMA/CD like wired Ethernet
- Two main Wi-Fi frequency bands: **2.4 GHz** (longer range, more interference) and **5 GHz** (shorter range, less interference)
- In the **2.4 GHz band**, use only channels **1, 6, and 11** — the only three non-overlapping channels
- **5 GHz band** channels are all non-overlapping — easier to plan
- **SSID** = human-readable network name; **BSSID** = unique identifier for an AP (its radio MAC address)
- Service set types: **IBSS** (ad hoc, no AP) · **BSS** (single AP) · **ESS** (multiple APs, same SSID, roaming) · **MBSS** (mesh)
- In an **ESS**, each BSS uses the **same SSID**, a **unique BSSID**, and a **different non-overlapping channel**
- The upstream wired network connected to an AP is called the **DS (Distribution System)**
- AP modes: **Repeater**, **Workgroup Bridge (WGB)**, **Outdoor Bridge**

---

## 1. 📶 Wireless LAN Fundamentals

### Key Differences from Wired LANs

| Feature | Wired (Ethernet) | Wireless (Wi-Fi) |
| --- | --- | --- |
| **Frame visibility** | Only intended recipient (via switch) | All devices in range receive all frames |
| **Duplex** | Full-duplex (switch) | Half-duplex |
| **Collision handling** | CSMA/**CD** (detect & recover) | CSMA/**CA** (avoid before they occur) |
| **Data encryption in LAN** | Usually not required | **Required** — anyone in range can intercept |
| **Signal medium** | Physical cable (contained) | Electromagnetic waves (radiate outward) |

### CSMA/CA Process (Collision Avoidance)

1. Device prepares frame to send
2. **Listens** to check if the channel is free
3. If channel is **busy** → wait a random period of time → listen again
4. If channel is **free** → transmit the frame

> 💡 Optional: device may send an **RTS (Request to Send)** and wait for a **CTS (Clear to Send)** before transmitting — not required for CCNA, but know the terms.
> 

### Wireless Signal Issues

Wireless signals can be degraded by five physical phenomena:

| Phenomenon | What Happens |
| --- | --- |
| **Absorption** | Signal passes through a material (e.g., wall) and is converted to heat — weakens signal |
| **Reflection** | Signal bounces off a material (e.g., metal) — common cause of poor signal in elevators |
| **Refraction** | Signal bends when entering a medium where it travels at a different speed (e.g., glass, water) |
| **Diffraction** | Signal travels around an obstacle — can create blind spots behind the obstacle |
| **Scattering** | Signal strikes an uneven surface (dust, smog) and scatters in all directions |

> ⚠️ All five must be considered when planning access point placement.
> 

### Other Wireless Considerations

- **Interference** from neighboring wireless networks using the same channels
- **Regulatory restrictions** — permitted channels vary by country; 802.11 defines allowed frequencies
- **Signal range** — limited by physical environment and the phenomena above

---

## 2. 📻 Radio Frequency (RF)

### Electromagnetic Wave Properties

| Term | Definition |
| --- | --- |
| **Amplitude** | Maximum strength of the electric/magnetic field (height of the wave) |
| **Frequency** | Number of up/down cycles per second, measured in **Hertz (Hz)** |
| **Period** | Time duration of one cycle (Period = 1 ÷ Frequency) |

### Frequency Units

- **Hz** = cycles per second
- **KHz** = thousands of cycles/sec
- **MHz** = millions of cycles/sec
- **GHz** = billions of cycles/sec
- **THz** = trillions of cycles/sec

### Wi-Fi Frequency Bands

| Band | Actual Range | Characteristics |
| --- | --- | --- |
| **2.4 GHz** | 2.400 – 2.4835 GHz | Longer range, better wall penetration, **more interference** (more devices use it) |
| **5 GHz** | 5.150 – 5.825 GHz (4 sub-bands) | Shorter range, **less interference**, non-overlapping channels |
| **6 GHz** | (Wi-Fi 6 / 802.11ax only) | Newest addition; expanded spectrum |

> 📌 For the CCNA: memorize **2.4 GHz** and **5 GHz** as the two primary bands.
> 

---

## 3. 📊 Wi-Fi Channels

### 2.4 GHz Channels

- Divided into several channels, each **22 MHz wide**
- Channels **overlap** with adjacent channels
- To avoid interference between APs, use only **non-overlapping channels**
- **Recommended (North America): Channels 1, 6, and 11**
    - These three do not overlap with each other
    - APs arranged in a **honeycomb pattern** using channels 1, 6, 11 provide full coverage without interference
    - BSA (coverage areas) should **overlap by ~10–15%** to enable seamless roaming

### 5 GHz Channels

- Channels are **non-overlapping by design**
- Much easier to plan multi-AP deployments without interference

> 🔑 **Exam critical:** Know that **1, 6, and 11** are the non-overlapping channels in the 2.4 GHz band.
> 

---

## 4. 📋 802.11 Wi-Fi Standards

| Standard | Wi-Fi Name | Frequency | Max Speed |
| --- | --- | --- | --- |
| **802.11** (original) | — | 2.4 GHz | 2 Mbps |
| **802.11a** | — | 5 GHz | 54 Mbps |
| **802.11b** | — | 2.4 GHz | 11 Mbps |
| **802.11g** | — | 2.4 GHz | 54 Mbps |
| **802.11n** | **Wi-Fi 4** | 2.4 / 5 GHz | 600 Mbps |
| **802.11ac** | **Wi-Fi 5** | 5 GHz | 6.93 Gbps |
| **802.11ax** | **Wi-Fi 6** | 2.4 / 5 / 6 GHz | 9.6 Gbps |

> ⚠️ There is no official Wi-Fi 1, 2, or 3. Numbering starts at Wi-Fi 4 (802.11n).
⚠️ These are **theoretical maximum** data rates — real-world performance is lower.
> 

---

## 5. 🗂️ Service Sets

> All devices in a service set share the same **SSID (Service Set Identifier)** — the human-readable Wi-Fi network name.
> 

### IBSS — Independent Basic Service Set

- Wireless devices connect **directly to each other** — no AP required
- Also called: **ad hoc network**
- Example use: Apple AirDrop file transfers
- **Not scalable** — limited to a few devices

### BSS — Basic Service Set

- Clients connect via an **Access Point (AP)** — not directly to each other
- Type of **infrastructure service set**
- **BSSID** = unique identifier for the AP = the **MAC address of the AP's radio**
- **BSA (Basic Service Area)** = the physical coverage area around the AP
- All client traffic must flow **through the AP**, even between two clients in range of each other
- Clients **associate** with the BSS to join; associated clients are called **stations**

### ESS — Extended Service Set

- Multiple BSSs connected via a **wired network (switch)** to cover a larger area
- **Same SSID** across all BSSs
- **Unique BSSID** per AP
- Each BSS uses a **different non-overlapping channel**
- Supports **roaming** — clients move between APs seamlessly without reconnecting
- BSAs should overlap by **10–15%** to avoid connectivity gaps during roaming

### MBSS — Mesh Basic Service Set

- Used when running Ethernet cables to every AP is impractical
- Each mesh AP has **two radios:**
    - **Radio 1:** Provides BSS for wireless clients
    - **Radio 2:** Forms the **backhaul network** (AP-to-AP communication)
- **RAP (Root Access Point):** The AP connected to the wired network
- **MAP (Mesh Access Point):** All other APs in the mesh
- A protocol determines the best path through the mesh (similar to dynamic routing)

### Service Set Comparison

| Type | AP Required? | Wired Backbone? | Use Case |
| --- | --- | --- | --- |
| **IBSS** | ❌ No | ❌ No | Ad hoc / file transfer |
| **BSS** | ✅ Yes | ✅ Yes | Single AP coverage area |
| **ESS** | ✅ Yes (multiple) | ✅ Yes | Large area, roaming support |
| **MBSS** | ✅ Yes (mesh) | ⚠️ Partial (RAP only) | Areas where cabling is difficult |

---

## 6. 🔌 Distribution System (DS)

- The **upstream wired network** that APs connect wireless clients to
- In 802.11, this is called the **DS (Distribution System)**
- Each wireless BSS/ESS is **mapped to a VLAN** in the wired network
- An AP can support **multiple SSIDs**, each mapped to a different VLAN
- The AP connects to the switch via a **trunk link** (carries multiple VLANs)
- Each additional SSID on an AP gets a unique BSSID (usually MAC address + 1 per SSID)

**Example:**

`SSID: Jeremy's Wi-Fi  →  VLAN 10  →  BSSID ends ...3456
SSID: Guest Wi-Fi     →  VLAN 11  →  BSSID ends ...3457
AP connects to switch via trunk (allows VLAN 10 + 11)`

---

## 7. 🔧 AP Operational Modes

### Repeater Mode

- Extends the range of a BSS by **retransmitting AP signals**
- **Single-radio repeater:** Uses the same channel as the AP → **cuts effective throughput by 50%**
- **Dual-radio repeater:** Receives on one channel, retransmits on another → avoids throughput loss

### Workgroup Bridge (WGB)

- AP acts as a **wireless client** of another AP
- Bridges **wired devices** (that lack wireless capability) onto the wireless network
- **uWGB (Universal WGB):** 802.11 standard — bridges **one** wired device
- **WGB (Cisco proprietary):** Bridges **multiple** wired devices

### Outdoor Bridge

- Connects **geographically separate networks** over long distances without physical cabling
- Uses **directional antennas** to focus signal power in one direction
- **Point-to-point:** Connects two sites
- **Point-to-multipoint:** Multiple sites connect to one central site (hub-and-spoke)

---

## 8. 📝 Quiz Answers (Self-Test)

| # | Question | Answer |
| --- | --- | --- |
| 1 | Which 2.4 GHz channels should be used with multiple APs? | **1, 6, and 11** (non-overlapping) |
| 2 | What is the main purpose of an AP in a mostly wired network? | **Connect wireless devices to the wired network (DS)** |
| 3 | Which bands do wireless LANs commonly use? (pick 2) | **2.4 GHz** and **5 GHz** |
| 4 | Which statements about an ESS are true? (pick 2) | Each BSS uses a **unique BSSID** · **Roaming** provides seamless connectivity between APs |
| 5 | What is NOT true about an AP providing multiple BSSs? | "Each BSS shares the same BSSID" — **False**; each BSS must have a unique BSSID |

---

## 9. 🔑 Critical Reminders

- **CSMA/CA** = wireless (avoid collisions) · **CSMA/CD** = wired (detect & recover) — do NOT mix these up
- **Non-overlapping 2.4 GHz channels = 1, 6, 11** — this is an explicit exam topic (1.11.a)
- **SSID** = network name (human-readable, does not have to be unique) · **BSSID** = AP's radio MAC address (always unique)
- In an **ESS**: same SSID, unique BSSID per AP, different channels per AP, ~10–15% BSA overlap for roaming
- **Clients in a BSS cannot communicate directly** — all traffic flows through the AP
- **DS** = the wired network the AP bridges wireless clients onto
- A single AP can host **multiple SSIDs**, each mapped to a different VLAN via a trunk
- Wi-Fi numbering: **802.11n = Wi-Fi 4, 802.11ac = Wi-Fi 5, 802.11ax = Wi-Fi 6** — no Wi-Fi 1, 2, or 3

---

# Day 56 (Wireless Architectures)

**Key Takeaways**

- 802.11 frames have up to 4 address fields and differ significantly from 802.3 Ethernet frames
- Three AP types: Autonomous (independent), Lightweight (split-MAC with WLC), Cloud-based (autonomous managed via cloud)
- Lightweight APs use CAPWAP tunnels to a WLC — control tunnel (UDP 5246) and data tunnel (UDP 5247)
- Lightweight APs connect via access ports; autonomous APs require trunk ports
- Four WLC deployment models: Unified (6000 APs), Cloud-based (3000 APs), Embedded (200 APs), Mobility Express (100 APs)

**802.11 Frame Format**

Unlike 802.3 Ethernet, 802.11 frames have more fields and their presence depends on the message type and 802.11 version.

| Field | Size | Purpose |
| --- | --- | --- |
| **Frame Control** | 2 bytes | Message type and subtype |
| **Duration/ID** | 2 bytes | Channel reservation time or association ID |
| **Address 1–4** | Up to 4 addresses | DA (final destination), SA (original source), RA (immediate recipient), TA (immediate sender) |
| **Sequence Control** | Variable | Reassembles fragments, eliminates duplicates |
| **QoS Control** | Variable | Traffic prioritization |
| **HT Control** | Variable | High throughput operations (added in 802.11n) |
| **Frame Body** | Variable | Encapsulated packet (data) |
| **FCS** | 4 bytes | Error checking (trailer) |

**802.11 Association Process**

Three connection states:

1. Not authenticated, not associated
2. Authenticated, not associated
3. Authenticated and associated — required to send traffic through the AP

**Scanning methods:**

- **Active scanning** — station sends probe requests; AP replies with probe response
- **Passive scanning** — station listens for periodic beacon messages from AP

Process: Probe → Authentication exchange → Association request/response → Traffic can flow

**802.11 Message Types**

| Type | Purpose | Examples |
| --- | --- | --- |
| **Management** | Manage the BSS | Beacon, probe, authentication, association |
| **Control** | Control medium access; assist data/management delivery | RTS, CTS, ACK |
| **Data** | Carry actual data packets | Standard data frames |

**Autonomous APs**

Self-contained, independent — no WLC required. Configured individually via console, SSH, Telnet, or HTTP/HTTPS GUI.

- Each AP handles RF parameters, security, QoS, and policies independently
- Must connect to the wired network via a **trunk link** (one VLAN per SSID + management VLAN)
- Data traffic takes a direct path between AP and wired network
- VLANs must stretch across the entire network — leads to large broadcast domains and spanning tree issues
- Suitable for small networks only; not scalable

**Lightweight APs (Split-MAC Architecture)**

AP functions are split between the AP and a **WLC (Wireless LAN Controller)**:

- **AP handles:** RF transmission/reception, encryption/decryption, beacons and probes
- **WLC handles:** RF management, security/QoS, client authentication, association, roaming

APs communicate with the WLC using **CAPWAP** (Control and Provisioning of Wireless Access Points):

| Tunnel | Port | Traffic | Encrypted by Default? |
| --- | --- | --- | --- |
| **Control tunnel** | UDP 5246 | AP configuration and management | Yes |
| **Data tunnel** | UDP 5247 | All client traffic | No (can enable DTLS) |
- Lightweight APs connect to switches via **access ports** (trunk not required)
- WLC connects to the network via a **trunk port**
- All client traffic — even between devices on the same AP — is tunneled to the WLC first
- APs and WLCs authenticate each other using **X.509 digital certificates**

**Benefits of split-MAC:**

- Scalability for large networks
- Dynamic channel assignment and transmit power control
- Self-healing coverage (increases nearby AP power if one AP fails)
- Seamless client roaming
- Client load balancing
- Centralized security and QoS management

**Lightweight AP Operational Modes**

| Mode | Offers BSS? | Purpose |
| --- | --- | --- |
| **Local** | Yes | Default mode — standard AP operation |
| **FlexConnect** | Yes | Locally forwards traffic if WLC tunnel is down |
| **Sniffer** | No | Captures 802.11 frames and sends to Wireshark |
| **Monitor** | No | Detects rogue devices; can send de-authentication messages |
| **Rogue Detector** | No | Listens to wired ARP traffic to detect rogues (no radio used) |
| **SE-Connect** | No | RF spectrum analysis; sends data to Cisco Spectrum Expert |
| **Bridge/Mesh** | No | Dedicated bridge between sites; can form a mesh |
| **Flex+Bridge** | No | Bridge/mesh with FlexConnect local switching capability |

**Cloud-Based APs**

- Autonomous APs centrally managed via a cloud dashboard (e.g. **Cisco Meraki**)
- Management/control traffic goes to the cloud
- Regular data traffic goes directly to the wired network — not through the cloud
- Functionality is between autonomous and lightweight: independent data forwarding, centralized management

**WLC Deployment Models**

| Deployment | WLC Location | Max APs | Suitable For |
| --- | --- | --- | --- |
| **Unified** | Dedicated hardware appliance, central location | ~6,000 | Large enterprise campus |
| **Cloud-based** | VM on a server in a private data center | ~3,000 | Large/distributed networks |
| **Embedded** | Built into a network switch | ~200 | Smaller campuses |
| **Mobility Express** | Built into an AP | ~100 | Small branch offices |

Note: Cloud-based WLC deployment is different from cloud-based AP architecture — these are still lightweight APs; "cloud-based" only refers to where the WLC runs.

---

# Day 57 (Wireless Security)

# Wireless Network Security — CCNA Study Notes

*(Exam Topics: 1.11.d Encryption, 5.9 WPA/WPA2/WPA3)*

## 🎯 Key Takeaways

- Wireless networks require **encryption of all traffic** between clients and APs because wireless signals can be received by any device within range — unlike wired LANs where in-LAN encryption is rarely used.
- Three core security concepts: **Authentication** (verify identity), **Encryption** (keep traffic private), **Integrity** (ensure messages aren't tampered with in transit via a **MIC**).
- **Authentication progression:** Open (no security) → WEP (broken) → EAP-based methods (LEAP → EAP-FAST → PEAP → **EAP-TLS**, most secure). EAP works alongside **802.1X**, which uses three roles: **supplicant** (client), **authenticator** (AP/WLC), and **authentication server** (RADIUS).
- **Encryption/integrity progression:** WEP (broken) → **TKIP** (temporary WEP fix, WPA) → **CCMP/AES** (WPA2) → **GCMP** (WPA3, most secure and efficient).
- **PEAP vs. EAP-TLS:** PEAP requires a certificate only on the **authentication server**; EAP-TLS requires certificates on **both** the server and every client (more secure, harder to implement).
- **WPA versions at a glance:** WPA = TKIP; WPA2 = CCMP/AES; WPA3 = GCMP + PMF (mandatory) + SAE + forward secrecy.
- All three WPA versions support **personal mode** (PSK — password-based) and **enterprise mode** (802.1X + EAP with an auth server).

---

## 1. Why Wireless Security Matters

- Wireless signals are not contained within a cable — **any device within range can receive the traffic**.
- In wired networks, traffic within the LAN is typically sent unencrypted; in wireless networks, encryption between clients and the AP is **essential**.
- Three pillars of wireless security:
    - **Authentication** — verify who is connecting.
    - **Encryption** — ensure only intended recipients can read the traffic.
    - **Integrity** — ensure messages are not modified in transit.

---

## 2. Authentication

### Key Concepts

- All clients must be **authenticated before associating** with an AP.
- Ideally, clients also authenticate the AP (to avoid connecting to a malicious "evil twin" AP that could conduct man-in-the-middle attacks).
- Authentication methods range from passwords, to username/password combos, to **digital certificates**.

### Overview of All 7 Authentication Methods

### 1. Open Authentication

- Client sends an authentication request; AP accepts with **no credentials required**.
- **Not secure on its own**, but still used today in combination with other methods (e.g., public Wi-Fi where you associate freely, then must log in via a web portal to gain access).

### 2. WEP (Wired Equivalent Privacy)

- Part of the original 802.11 standard; provides both **authentication and encryption**.
- **Shared-key protocol** — sender and receiver must have the same pre-shared key.
- Key lengths: 40-bit or 104-bit, combined with a 24-bit initialization vector (total: 64-bit or 128-bit).
- **Authentication process:** AP sends a challenge phrase → client encrypts it with the WEP key → AP compares to its own encrypted version → match = authenticated.
- **WEP is completely broken** and must never be used on modern networks, regardless of key length.
- Can be used for encryption only (with open authentication), or for both authentication and encryption.

### 3. EAP (Extensible Authentication Protocol) + 802.1X Framework

- **EAP** is not a single protocol — it is an **authentication framework** that defines standard functions used by various EAP methods.
- **802.1X** provides port-based network access control, limiting network access until a client authenticates.
- **Three 802.1X roles** (must know):

| Role | Definition | Wireless Example |
| --- | --- | --- |
| **Supplicant** | Device wanting to connect | Laptop/client |
| **Authenticator** | Device providing network access | AP (or WLC in split-MAC) |
| **Authentication Server** | Verifies credentials, permits/denies | RADIUS server |
- **How it works in wireless:** the 802.11 association itself uses open authentication (just to associate), but the client then gets **no real network access** until it completes EAP authentication with the authentication server.

---

### EAP Methods (4 to Know)

### LEAP (Lightweight EAP)

- Developed by **Cisco** as an improvement over WEP.
- **Mutual authentication:** both the client and server send challenge phrases to each other (vs. WEP where only the server sends one).
- Uses **dynamic WEP keys** that change over time.
- **Considered vulnerable — do not use.**

### EAP-FAST (EAP Flexible Authentication via Secure Tunneling)

- Developed by **Cisco**.
- **Three phases:**
    1. Server generates a **PAC (Protected Access Credential)** and sends it to the client.
    2. PAC is used to establish a **secure TLS tunnel** between client and auth server.
    3. Client is **authenticated within the TLS tunnel**.
- Key distinction: uses a **PAC** (not a digital certificate) to establish the tunnel.

### PEAP (Protected EAP)

- Establishes a **secure TLS tunnel** like EAP-FAST, but uses a **digital certificate** on the **authentication server** (not a PAC) to authenticate the server and build the tunnel.
- Client is then **authenticated within the TLS tunnel** (e.g., using **MS-CHAP** — Microsoft Challenge Handshake Authentication Protocol).
- Only the **server** needs a certificate — clients do not.

### EAP-TLS (EAP Transport Layer Security)

- **Most secure EAP method.**
- Requires **digital certificates on both the authentication server AND every client device**.
- Client and server authenticate each other via certificates — no need to authenticate the client inside the TLS tunnel.
- The TLS tunnel is still established for **exchanging encryption key information**.
- More complex and time-consuming to implement than PEAP (every device needs a certificate).

### PEAP vs. EAP-TLS Summary

| Feature | PEAP | EAP-TLS |
| --- | --- | --- |
| Server certificate | ✅ Required | ✅ Required |
| Client certificate | ❌ Not required | ✅ Required on every device |
| Security level | High | **Highest** |
| Implementation effort | Moderate | More complex |

---

## 3. Encryption & Integrity Methods

### MIC (Message Integrity Check)

- **Ensures a message was not modified in transit.**
- Process: sender calculates a MIC → appends it to the message → encrypts both → sends frame. Receiver decrypts → independently calculates MIC → compares. If MICs match = message is intact; if not = message is **discarded**.

### TKIP (Temporal Key Integrity Protocol)

- Developed as a **temporary, backward-compatible fix** for WEP when WEP was proven broken, while new hardware was still being built.
- Based on WEP but adds multiple security improvements:
    - Uses a **MIC** to protect message integrity.
    - **Key mixing algorithm** generates a unique WEP key per frame (instead of reusing the same key).
    - Initialization vector doubled from **24-bit to 48-bit** (makes brute-force much harder).
    - MIC includes the **sender's MAC address** to prove the frame's origin.
    - **Timestamp** and **TKIP sequence number** added to prevent **replay attacks**.
- Used in **WPA (version 1)**.
- Now considered outdated — superseded by CCMP and GCMP.

### CCMP (Counter/CBC-MAC Protocol)

- Developed after TKIP; more secure. Requires hardware that supports it (old WEP/TKIP-only hardware cannot use it).
- Used in **WPA2**.
- Two algorithms:
    - **AES Counter Mode** — encryption (AES is the gold standard encryption protocol used by governments, corporations worldwide).
    - **CBC-MAC (Cipher Block Chaining Message Authentication Code)** — MIC for message integrity.

### GCMP (Galois/Counter Mode Protocol)

- **Most secure and most efficient** of the three. Higher data throughput than CCMP.
- Used in **WPA3**.
- Two algorithms:
    - **AES Counter Mode** — encryption (same as CCMP).
    - **GMAC (Galois Message Authentication Code)** — MIC for message integrity.

### Encryption/Integrity Summary

| Protocol | Used In | Encryption | Integrity (MIC) | Status |
| --- | --- | --- | --- | --- |
| **WEP** | Original 802.11 | RC4 | None | ❌ Broken |
| **TKIP** | WPA | RC4-based (per-frame key) | MIC + sequence # | ⚠️ Outdated |
| **CCMP** | WPA2 | AES (Counter Mode) | CBC-MAC | ✅ Secure |
| **GCMP** | WPA3 | AES (Counter Mode) | GMAC | ✅ Most Secure |

---

## 4. WPA Certifications (Wi-Fi Protected Access)

### Overview

- Developed by the **Wi-Fi Alliance** to create standard, certified security bundles — solving the problem of choosing which authentication, encryption, and integrity protocols to combine.
- Three certifications: **WPA**, **WPA2**, **WPA3** (no "WPA1" — the first version is just called WPA).
- Devices must be tested in authorized labs to receive WPA certification.

### Two Authentication Modes (All WPA Versions)

- **Personal Mode (PSK — Pre-Shared Key):** a password is shared between the client and AP. The PSK is never sent over the air; a **four-way handshake** uses it to generate encryption keys. Common in homes and SOHO networks.
- **Enterprise Mode:** uses **802.1X** with an authentication server (RADIUS). Any EAP method is supported (PEAP, EAP-TLS, etc.). Common in corporate environments.

### WPA Versions Comparison

| Feature | WPA | WPA2 | WPA3 |
| --- | --- | --- | --- |
| Encryption/Integrity | **TKIP** | **CCMP (AES)** | **GCMP** |
| Personal Auth | PSK | PSK | PSK + **SAE** |
| Enterprise Auth | 802.1X / EAP | 802.1X / EAP | 802.1X / EAP |
| PMF (Mgmt Frame Protection) | ❌ | Optional | ✅ Mandatory |
| Forward Secrecy | ❌ | ❌ | ✅ |
| Release | ~2003 | ~2004 | 2018 |

### WPA3 Additional Security Features

- **PMF (Protected Management Frames):** protects 802.11 management frames from eavesdropping and forgery. Optional in WPA2, **mandatory in WPA3**.
- **SAE (Simultaneous Authentication of Equals):** replaces the standard four-way handshake in personal mode with a more secure authentication exchange, protecting against offline password attacks.
- **Forward Secrecy:** prevents captured wireless frames from being decrypted later, even if the key is eventually compromised — protects against attackers who record traffic and attempt to crack it after the fact.

---

## 5. Quiz Review

**Q1:** What does GMAC provide to a secure wireless connection?

- **Answer: B — MIC (Message Integrity Check).** GMAC (Galois Message Authentication Code) is the integrity component of GCMP (WPA3), used to verify that messages haven't been tampered with.

**Q2:** Which three entities are part of 802.1X authentication architecture?

- **Answer: A (Supplicant), D (Authenticator), E (Authentication Server).** In wireless: client = supplicant; AP/WLC = authenticator; RADIUS server = authentication server.

**Q3:** Which encryption/integrity method is considered most secure?

- **Answer: C — GCMP.** Development order: WEP → TKIP → CCMP → GCMP. GCMP (WPA3) is the most secure and most efficient.

**Q4:** Which EAP method requires a certificate on both the supplicant and the authentication server?

- **Answer: D — EAP-TLS.** Both PEAP and EAP-TLS use digital certificates, but only EAP-TLS requires them on **both** the server and every client device.

**Q5:** Which WPA3 feature protects the four-way handshake in personal mode?

- **Answer: A — SAE (Simultaneous Authentication of Equals).**

---

# Day 58 (Wireless Configuration)

**📡 Wireless LAN Configuration — CCNA Study Notes**

> **Exam Topics:** 2.7, 2.8, 2.9 (configure WLAN components using GUI), 5.10 (configure WLANs using WPA2 PSK via GUI)
> 

---

**⚡ Key Takeaways**

- The CCNA requires GUI-based configuration of wireless LANs — not CLI.
- WLCs use **static LAG only** (no PAgP or LACP). Use `channel-group 1 mode on` on the switch.
- **DHCP Option 43** tells APs the IP address of their WLC — important when the WLC is not in the same subnet as the APs.
- WLC **ports** = physical. WLC **interfaces** = logical (like SVIs on a switch). Do not use these terms interchangeably for WLCs.
- **Dynamic interfaces** map WLANs to VLANs — create one per WLAN.
- Always map a WLAN to its correct dynamic interface (not the management interface).
- WPA2 PSK = Layer 2 security → WPA+WPA2 with **PSK** (not 802.1X). PSK must be ≥ 8 ASCII characters.
- QoS tiers: **Platinum** (voice) → **Gold** (video) → **Silver** (default/best effort) → **Bronze** (background).
- A **CPU ACL** limits management access to the WLC (Telnet/SSH, HTTP/HTTPS, SNMP) — does not affect data traffic passing through.

---

**🗺️ Network Topology**

| **Device** | **Role** | **Notes** |
| --- | --- | --- |
| SW1 | Core switch | DHCP server, NTP server, SVI default gateways |
| WLC1 | Wireless LAN Controller | Connected to SW1 via static LAG |
| AP1, AP2 | Access Points | Powered via PoE from switch |

**VLANs and Subnets**

| **VLAN** | **Name** | **Subnet** | **Purpose** |
| --- | --- | --- | --- |
| 10 | Management | 192.168.1.0/24 | Managing WLC, APs, switch |
| 100 | Internal | 10.0.0.0/24 | Internal wireless clients |
| 200 | Guest | 10.1.0.0/24 | Guest wireless clients |
- WLC IP: `.100` in each subnet. SW1 SVI: `.1` in each subnet.
- APs get management VLAN IPs via **DHCP**.
- Switch ports connecting to APs are **access ports in VLAN 10** (split-MAC deployment — only the WLC needs a trunk).

---

**🔧 Switch Configuration**

**Interface Setup**

```
! Create VLANs
vlan 10
 name Management
vlan 100
 name Internal
vlan 200
 name Guest

! AP-facing access ports (VLAN 10)
interface fa0/6-8
 switchport mode access
 switchport access vlan 10

! WLC-facing static LAG (WLCs only support static LAG)
interface fa0/1-2
 channel-group 1 mode on       ← NOT mode active or mode desirable

interface port-channel 1
 switchport mode trunk
 switchport trunk allowed vlan 10,100,200
```

**SVIs and DHCP**

```
! SVIs as default gateways
interface vlan 10
 ip address 192.168.1.1 255.255.255.0
interface vlan 100
 ip address 10.0.0.1 255.255.255.0
interface vlan 200
 ip address 10.1.0.1 255.255.255.0

! DHCP pools
ip dhcp pool VLAN10
 network 192.168.1.0 255.255.255.0
 default-router 192.168.1.1
 option 43 ip 192.168.1.100      ← Tells APs where the WLC is

ip dhcp pool VLAN100
 network 10.0.0.0 255.255.255.0
 default-router 10.0.0.1

ip dhcp pool VLAN200
 network 10.1.0.0 255.255.255.0
 default-router 10.1.0.1

! NTP server
ntp master
```

> **DHCP Option 43** — tells APs the WLC's IP address. Required when the WLC is in a different subnet and won't hear broadcast CAPWAP discovery messages.
> 

---

**🖥️ WLC Initial Setup (CLI / Console Wizard)**

The WLC boots into a **configuration wizard** — no CLI commands required.

| **Setting** | **Value** |
| --- | --- |
| System name | WLC1 |
| Enable LAG | Yes |
| Management IP | 192.168.1.100 / 24 |
| Default gateway | 192.168.1.1 |
| VLAN ID | 10 |
| DHCP server | 192.168.1.1 |
| Country code | Must match AP regulatory domain (e.g., FR for -E APs) |
| NTP server | 192.168.1.1 |

> **Regulatory domain warning:** The country code set on the WLC must match the regulatory domain of the APs (e.g., `-E` for Europe, `-A` for North America). A mismatch prevents APs from joining the WLC.
> 

---

**🌐 Accessing the WLC GUI**

1. Connect a PC to a switch port in the **management VLAN** (VLAN 10).
2. Open a browser and navigate to `https://192.168.1.100`.
3. Accept the SSL certificate warning (certificate authority not trusted — expected for local devices).
4. Log in with the admin credentials created during initial setup.

> The GUI **cannot** be accessed via console port. Must use HTTP/HTTPS over the network.
> 

---

**🔌 WLC Ports vs. Interfaces**

**Physical Ports**

| **Port Type** | **Purpose** |
| --- | --- |
| **Service port** | Out-of-band management only. Connects to switch **access port** (one VLAN). Used for boot/recovery tasks. |
| **Distribution system ports** | Standard network ports for regular data traffic. Connect to switch **trunk ports**. Can form a LAG. |
| **Console port** | RJ45 or USB — direct CLI access. |
| **Redundancy port** | Connects two WLCs to form an HA (high availability) pair. |

**Logical Interfaces**

| **Interface Type** | **Purpose** |
| --- | --- |
| **Management interface** | Handles Telnet/SSH, HTTP/HTTPS, RADIUS, NTP, Syslog. CAPWAP tunnels form here. |
| **Redundancy management interface** | Used to manage the standby WLC in an HA pair. |
| **Virtual interface** | Used when communicating with wireless clients (DHCP relay, web auth). Requires a dummy IP (e.g., 1.1.1.1). |
| **Service port interface** | Bound to the service port if out-of-band management is used. |
| **Dynamic interfaces** | Map a WLAN to a VLAN. Create one per WLAN. |

---

**📶 Configuring Dynamic Interfaces (GUI)**

**Path:** Controller → Interfaces → New

For each WLAN, create a dynamic interface:

| **Interface** | **VLAN** | **IP Address** | **Gateway** | **DHCP Server** |
| --- | --- | --- | --- | --- |
| Internal | 100 | 10.0.0.100 | 10.0.0.1 | 10.0.0.1 |
| Guest | 200 | 10.1.0.100 | 10.1.0.1 | 10.1.0.1 |

---

**📡 Configuring WLANs (GUI)**

**Path:** WLANs tab → New (or click WLAN ID to edit existing)

**General Tab**

| **Setting** | **Value** |
| --- | --- |
| Profile name | Internal (or Guest) |
| SSID | internal (or guest) |
| Status | Enabled |
| Interface | Select the matching dynamic interface (NOT management) |

**Security Tab → Layer 2**

| **Setting** | **Value** |
| --- | --- |
| Layer 2 security | WPA + WPA2 |
| Authentication key management | **PSK** (deselect 802.1X) |
| PSK format | ASCII |
| PSK value | ≥ 8 characters required |

> For CCNA: configure **WPA2 PSK** (personal mode). PSK must be at least 8 ASCII characters.
> 

**QoS Tab**

| **Tier** | **Use case** |
| --- | --- |
| **Platinum** | Voice (VoIP, WiFi phones) |
| **Gold** | Video |
| **Silver** | Default / best effort |
| **Bronze** | Background / lowest priority |

**Layer 3 Security (reference only)**

| **Option** | **Description** |
| --- | --- |
| Web authentication | User must enter username/password in browser after connecting. |
| Web passthrough | User must agree to terms in browser — no credentials needed. |
| Conditional / splash page redirect | Same as above but also requires 802.1X at Layer 2. |

---

**🔒 CPU ACL — Restricting Management Access**

**Path:** Security → Access Control Lists → New ACL → Apply as CPU ACL

- CPU ACLs limit which devices can reach the WLC itself (Telnet/SSH, HTTP/HTTPS, SNMP).
- Does **not** affect data traffic passing through the WLC.
- Apply the ACL under **Security → CPU Access Control Lists → Enable CPU ACL**.

---

**ℹ️ Additional WLC Settings (Reference)**

| **Feature** | **Location** | **Notes** |
| --- | --- | --- |
| AP list and details | Wireless tab | Shows IP, model, MAC, operational mode |
| AP operational mode | Wireless → AP name | Local, FlexConnect, Monitor, Rogue Detector, etc. |
| Management settings | Management tab | Enable/disable SSH, Telnet, HTTP/HTTPS, SNMP versions |
| Management via wireless | Management → Mgmt via Wireless | Allows wireless clients to manage the WLC — **disabled by default** |
| SNMP | Management tab | SNMPv1 disabled; v2 and v3 enabled by default |
| Telnet | Management tab | Disabled by default — leave disabled (not secure) |

---

**🔄 Traffic Flow in a Split-MAC Deployment**

1. Wireless client sends traffic → travels through **CAPWAP tunnel** to WLC.
2. WLC maps the WLAN to the appropriate VLAN via the **dynamic interface**.
3. WLC forwards traffic to SW1 in the correct VLAN.
4. SW1 routes or bridges traffic to its destination.

> All inter-WLAN traffic (e.g., Internal client → Guest client) passes through SW1 as the Layer 3 gateway — not directly between clients.
> 

---

# Day 59 (Intro to Network Automation (Part 1))

**🤖 Network Automation & SDN — CCNA Study Notes**

> **Exam Topics:** 6.1, 6.2, 6.3 — Explain/compare/describe network automation concepts **Weight:** Section 6.0 = **10% of the CCNA exam** **Note:** You do NOT need to automate tasks yourself — only understand the concepts.
> 

---

**⚡ Key Takeaways**

- Traditional networking manages devices one-by-one via CLI (SSH/Telnet). This is error-prone, slow, and hard to scale.
- Network automation reduces **human error**, **OpEx**, and **time-to-deploy** while improving **policy compliance** and **scalability**.
- Every network device function belongs to one of three logical planes: **Data**, **Control**, or **Management**.
- **Data plane** = forwarding user traffic (processed by **ASIC** + **TCAM** for speed).
- **Control plane** = building the tables that inform forwarding decisions (OSPF, STP, ARP). Runs on CPU.
- **Management plane** = managing and monitoring the device (SSH, Telnet, SNMP, Syslog, NTP). Runs on CPU.
- **SDN** centralizes the control plane into a software **controller**, removing it from individual devices.
- **SBI (Southbound Interface)** = controller ↔ network devices (e.g., OpenFlow, OpFlex, onePK, NETCONF).
- **NBI (Northbound Interface)** = controller ↔ applications (uses **REST API**; data returned as JSON or XML).
- SDN ≠ automation, but SDN greatly *facilitates* automation by providing centralized data and APIs.

---

**📉 Traditional Networking — Limitations**

| **Problem** | **Description** |
| --- | --- |
| **Human error** | Typos and mistakes are common when manually entering CLI commands device-by-device. |
| **Time-consuming** | Each device must be individually accessed and configured — unscalable for large networks. |
| **Configuration drift** | Over time, individual changes cause devices to deviate from the organization's standard configurations, complicating troubleshooting. |

---

**✅ Benefits of Network Automation**

| **Benefit** | **Details** |
| --- | --- |
| **Reduced human error** | Automated scripts replace manual CLI input — no typos. |
| **Greater scalability** | Deployments and changes that took hours can be done in minutes across hundreds of devices. |
| **Policy compliance** | Ensures all devices consistently meet standard configurations and correct software versions. |
| **Reduced OpEx** | Fewer engineer hours per task; engineers can focus on higher-value work. |

> **Example:** Adding a loopback interface to 200 routers manually = hours of repetitive work. A Python script = minutes.
> 

**Common Automation Tools**

- Python scripts
- Ansible / Puppet
- SDN (Software-Defined Networking)
- And many more — covered in later videos

---

**🧱 Logical Planes of Network Functions**

Network device functions are divided into three logical planes:

---

**1️⃣ Data Plane (= Forwarding Plane)**

> Everything involved in **forwarding user data** from one interface to another.
> 
- Router: receive packet → look up routing table → de-encapsulate old L2 header → re-encapsulate with new L2 header → forward out correct interface.
- Switch: receive frame → look up MAC address table → forward or flood out correct interface(s).
- Adding/removing **802.1Q VLAN tags**.
- **NAT** — modifying source/destination addresses before forwarding.
- **ACL** and **Port Security** decisions — forward or discard a message.

> **Hardware:** Processed by an **ASIC** (Application-Specific Integrated Circuit) — a chip purpose-built for fast forwarding logic. **Table storage:** MAC address tables (also called **CAM tables**) are stored in **TCAM** (Ternary Content-Addressable Memory) for extremely fast lookups. ⚠️ Remember: **CAM table = MAC address table. TCAM = the memory that stores it.**
> 

---

**2️⃣ Control Plane**

> Functions that **build and maintain the tables** used by the data plane to make forwarding decisions.
> 
- **OSPF** — builds the routing table (does not forward user data directly).
- **STP** — determines which interfaces should forward and which should block.
- **ARP** — builds the ARP table mapping IP addresses to MAC addresses.
- **BGP, EIGRP**, and other routing/switching protocols.

> These are **overhead** functions — they don't carry user data, but they *control* how user data is forwarded. **Hardware:** Processed by the device **CPU**. In traditional networking, the control plane is **distributed** — each device runs its own independently.
> 

---

**3️⃣ Management Plane**

> Protocols used to **manage and monitor** the device itself. Does not directly affect data forwarding.
> 

| **Protocol** | **Purpose** |
| --- | --- |
| **SSH / Telnet** | Remote CLI access for configuration |
| **Syslog** | Logs events occurring on the device |
| **SNMP** | Monitors device operations and status |
| **NTP** | Maintains accurate time on the device |

> **Hardware:** Processed by the device **CPU**.
> 

---

**Plane Summary Table**

| **Plane** | **Purpose** | **Hardware** | **Examples** |
| --- | --- | --- | --- |
| **Data (Forwarding)** | Forward user traffic | ASIC + TCAM | Routing, switching, NAT, ACL, VLAN tagging |
| **Control** | Build forwarding tables | CPU | OSPF, STP, ARP, BGP |
| **Management** | Manage and monitor device | CPU | SSH, SNMP, Syslog, NTP |

---

**🌐 Software-Defined Networking (SDN)**

> Also called: **Software-Defined Architecture** or **Controller-Based Networking**
> 

**What is SDN?**

SDN **centralizes the control plane** into a software application called a **controller**, rather than distributing it across individual devices.

- Traditional: each router runs OSPF independently, calculates its own routes.
- SDN: a **central controller** calculates routes and programs each device's data plane directly.

> You already know this concept from wireless: a **WLC** centralizes control functions away from APs — same idea.
> 

**How much is centralized?**

Varies by solution — some SDN implementations centralize the entire control plane; others only centralize specific functions. There is no single universal SDN model.

---

**🔗 SDN Interfaces**

**Southbound Interface (SBI)**

> Controller **↓ communicates down ↓** to network devices
> 
- Used by the controller to program and gather information from the managed network devices.
- Consists of a **communication protocol + API**.
- Gathers: device inventory, network topology, interface configurations, routing tables, etc.

| **SBI Example** | **Notes** |
| --- | --- |
| **OpenFlow** | Open standard |
| **Cisco OpFlex** | Cisco proprietary |
| **Cisco onePK** | Cisco proprietary |
| **NETCONF** | Standards-based; covered in later video |

> "Southbound" = controller is drawn at the top, network devices are drawn below (south) in diagrams.
> 

---

**Northbound Interface (NBI)**

> Applications **↑ communicate up ↑** to the controller
> 
- Used by **apps and scripts** to interact with the controller, retrieve network data, and push changes.
- The controller exposes a **REST API** (Representational State Transfer) as its northbound interface.
- Data is returned in structured formats: **JSON** or **XML** — easy for programs to parse and use.

> REST is a *type* of API, not a specific protocol — covered in detail in a later video.
> 

---

**SDN Architecture Diagram**

```
┌─────────────────────────────┐
│       Applications / Apps    │  ← Third-party apps, scripts, dashboards
└──────────────┬──────────────┘
               │  Northbound Interface (NBI)
               │  REST API → JSON / XML
┌──────────────▼──────────────┐
│         SDN Controller       │  ← Centralized control plane
└──────────────┬──────────────┘
               │  Southbound Interface (SBI)
               │  OpenFlow, NETCONF, OpFlex, onePK
┌──────────────▼──────────────┐
│     Network Devices (R1, R2) │  ← Data plane only
└─────────────────────────────┘
```

---

**⚖️ Traditional Automation vs. SDN**

| **Feature** | **Traditional (Scripts)** | **SDN** |
| --- | --- | --- |
| **How it works** | Python scripts SSH into devices, push commands, parse `show` output with regex | Controller collects data centrally; apps interact via REST API (JSON/XML) |
| **Data access** | Per-device — must SSH to each device and parse human-readable output | Network-wide view from a single controller |
| **Ease of use** | Requires scripting expertise | Many SDN tools usable without coding knowledge |
| **Third-party extensibility** | Custom scripts only | APIs allow third-party apps to interact with the controller |
| **Analytics** | Must correlate data manually from individual devices | Centralized data enables network-wide analytics out of the box |

> SDN and automation are **not the same thing** — SDN is one *component* of the broader network automation landscape.
> 

---

**🗂️ Key Terms to Memorize**

| **Term** | **Definition** |
| --- | --- |
| **ASIC** | Application-Specific Integrated Circuit — hardware chip that processes data plane operations at high speed |
| **TCAM** | Ternary Content-Addressable Memory — ultra-fast memory used to store and look up MAC/routing tables |
| **CAM table** | Another name for the MAC address table |
| **SBI** | Southbound Interface — controller ↔ network devices |
| **NBI** | Northbound Interface — controller ↔ applications |
| **REST API** | Type of API used at the NBI; returns data as JSON or XML |
| **JSON / XML** | Structured data formats that are easy for programs to parse |
| **OpEx** | Operating Expenses — reduced by network automation |
| **OpenFlow / OpFlex / onePK / NETCONF** | Examples of southbound interfaces |

---

# Day 59 (AI & Machine Learning (Part 2))

**🤖 AI & Machine Learning — CCNA Study Notes**

> **Exam Topic:** Section 6.0 — Network Automation (10% of CCNA exam) **Scope:** Understand concepts only — no hands-on AI implementation required for CCNA.
> 

---

**⚡ Key Takeaways**

- **AI** = computers simulating human-like intelligence (pattern recognition, decision-making, problem-solving).
- **Machine Learning (ML)** = a *subset* of AI that enables computers to learn from data automatically, without being explicitly programmed.
- Four main ML types to know: **Supervised**, **Unsupervised**, **Reinforcement**, and **Deep Learning**.
- **Supervised** = labeled data, high accuracy, limited to trained labels.
- **Unsupervised** = unlabeled data, finds hidden patterns, requires human interpretation.
- **Reinforcement** = learns via reward/penalty in an environment (self-driving cars, game AI).
- **Deep Learning** = uses multi-layered artificial neural networks; mimics the human brain; handles large/complex datasets.
- **Predictive AI** = analyzes historical data to forecast future outcomes (threat detection, traffic forecasting, predictive maintenance).
- **Generative AI** = creates new content based on learned patterns (ChatGPT, Midjourney, Sora).
- Cisco Catalyst Center AI features: **AI Network Analytics**, **Machine Reasoning Engine (MRE)**, **AI Endpoint Analytics**, **AI-enhanced RRM**.
- Deep learning models can be trained using *any* of the other three ML methods — they are not mutually exclusive.

---

**🧠 What is AI?**

**Artificial Intelligence (AI)** uses computers to simulate intelligence, enabling behaviors typically associated with humans:

- Recognizing patterns
- Learning from experience
- Making decisions
- Solving problems

**Examples of AI in use today**

| **Application** | **Description** |
| --- | --- |
| **Virtual assistants** | Siri, Alexa, Google Assistant — process voice commands and respond |
| **Recommendation systems** | Netflix, YouTube, Amazon — learn preferences and suggest content |
| **Self-driving cars** | Tesla FSD, Waymo — process sensor data and navigate in real time |
| **Chatbots / LLMs** | ChatGPT, Gemini — generate human-like text for support, writing, coding |
| **Game AI** | Stockfish (Chess), AlphaGo (Go) — surpass best human players |
| **Virtual concierge** | Website chatbots (e.g., cisco.com) that assist visitors |

> AI is a broad, growing field — fueled by increased computing power, big data availability, and breakthroughs like ChatGPT (2022).
> 

---

**📊 What is Machine Learning?**

**Machine Learning (ML)** is a subset of AI focused on enabling computers to **learn from data** and improve over time **without explicit programming**.

- Algorithms identify **patterns** in data.
- Use those patterns to make **predictions or decisions**.
- Cycle: Input data → Train model → Make predictions → Improve over time.

**ML is behind many everyday applications**

- Email spam filtering
- Product recommendations
- Banking fraud detection
- Natural language processing (NLP)

> ML is the driving force behind modern AI. ChatGPT wouldn't exist without it.
> 

---

**🗂️ AI / ML Hierarchy**

```
┌─────────────────────────────────────────────┐
│                    AI                        │
│  ┌───────────────────────────────────────┐  │
│  │          Machine Learning             │  │
│  │  ┌─────────────────────────────────┐  │  │
│  │  │         Deep Learning           │  │  │
│  │  │  ┌──────────┐  ┌────────────┐   │  │  │
│  │  │  │Predictive│  │ Generative │   │  │  │
│  │  │  │    AI    │  │     AI     │   │  │  │
│  │  │  └──────────┘  └────────────┘   │  │  │
│  │  └─────────────────────────────────┘  │  │
│  └───────────────────────────────────────┘  │
└─────────────────────────────────────────────┘
```

> Predictive and generative AI primarily leverage deep learning, but simpler versions exist outside it too.
> 

---

**🏷️ Types of Machine Learning**

**1. Supervised Learning**

> Trains the model on **labeled data** — correct answers are provided during training.
> 

**How it works:**

- Feed the model labeled examples (e.g., photos tagged "cat" or "dog").
- Model learns the relationship between inputs and labels.
- Once trained, classifies new, unlabeled data.

| **✅ Advantages** | **❌ Disadvantages** |
| --- | --- |
| Highly accurate when labeled data is available | Requires large, labeled datasets — expensive and time-consuming to create |
| Straightforward to understand and implement | Output limited to labels in training data — can't recognize new categories |

> Example: Train on labeled cat/dog photos → model can classify new photos correctly. But it can't identify a bird if "bird" was never a training label.
> 

---

**2. Unsupervised Learning**

> Trains the model on **unlabeled data** — discovers patterns and groupings on its own.
> 

**How it works:**

- Feed unlabeled data (no tags).
- Model groups similar data into **clusters** based on shared features.
- A human then interprets and labels those clusters.
- New data that doesn't fit any cluster triggers creation of a new one.

| **✅ Advantages** | **❌ Disadvantages** |
| --- | --- |
| No need for labeled data — saves time and cost | Human interpretation still required to make sense of results |
| Can reveal hidden patterns not obvious to humans | May be less accurate than supervised learning for specific tasks |

> Example: Feed unlabeled cat/dog photos → model groups them by visual similarity. Human labels Group 1 = "cat", Group 2 = "dog". A bird photo creates a new cluster.
> 

---

**3. Reinforcement Learning**

> Trains a model (called an **agent**) by **rewarding or penalizing** its actions in an environment.
> 

**How it works:**

- Agent takes an action in an environment.
- Receives **reward** (positive outcome) or **penalty** (negative outcome).
- Adjusts behavior to maximize rewards over time.
- Cycle repeats, continuously improving performance.

**Applications:**

- **Self-driving cars** — learns to stay in lane, avoid obstacles
- **Game AI** — Chess, Go, video games (AlphaGo, MarI/O)
- **Robotics** — learns to walk, pick up objects, assemble items

| **✅ Advantages** | **❌ Disadvantages** |
| --- | --- |
| Learns complex behaviors difficult to program manually | Resource-intensive — requires significant time and compute |
| Adapts well to dynamic, changing environments | Poorly designed reward systems can lead to short-term optimization instead of best long-term outcome |

---

**4. Deep Learning**

> A specialized subset of ML that uses **multi-layered artificial neural networks** — inspired by the human brain.
> 

**How it works:**

- Data passes through an **input layer** → multiple **hidden layers** → **output layer**.
- Each layer extracts increasingly abstract features.
- Can be trained using supervised, unsupervised, or reinforcement methods.

**Structure:**

```
[Input Layer] → [Hidden Layer 1] → [Hidden Layer 2] → ... → [Output Layer]
   (image)         (edges/shapes)    (features/objects)          ("cat")
```

**Applications:**

- Image and speech recognition
- Natural language processing (NLP)
- Autonomous driving
- Large language models (ChatGPT, Gemini)

| **✅ Advantages** | **❌ Disadvantages** |
| --- | --- |
| Excels with large, unstructured datasets (images, audio, text) | Very resource-intensive — needs large datasets and significant compute |
| State-of-the-art performance on complex tasks | "Black box" — hard to interpret how the model reaches its decisions |

---

**ML Types Summary Table**

| **Type** | **Data** | **Learns by** | **Key use case** |
| --- | --- | --- | --- |
| **Supervised** | Labeled | Comparing input to correct labels | Classification, prediction |
| **Unsupervised** | Unlabeled | Finding patterns/clusters independently | Anomaly detection, segmentation |
| **Reinforcement** | Environment feedback | Reward/penalty from actions | Game AI, robotics, self-driving |
| **Deep Learning** | Any (large volume) | Multi-layered neural networks | Image recognition, LLMs, NLP |

> These types are not mutually exclusive — deep learning can be trained with supervised, unsupervised, or reinforcement methods. Semi-supervised learning (combining labeled + unlabeled data) also exists.
> 

---

**🔮 Predictive AI vs. Generative AI**

**Predictive AI**

> Analyzes **historical data** to **forecast future outcomes**, trends, or events.
> 

**How it works:** Identifies patterns in past data → applies learned patterns to make predictions about new/future data.

**Applications:**

- Healthcare: predict patient outcomes, detect diseases in X-rays
- **Network security**: detect anomalies indicating threats or failures
- **Traffic management**: forecast congestion on roads or network links
- Business: sales forecasting, customer behavior analysis
- **Predictive maintenance**: anticipate hardware failures before they occur
- Meteorology: weather forecasting

| **✅ Advantages** | **❌ Disadvantages** |
| --- | --- |
| Enables proactive decisions before problems occur | Requires large amounts of high-quality historical data |
| Provides actionable insights | Struggles if future patterns significantly differ from past data |

---

**Generative AI**

> Learns patterns from existing data and **creates entirely new content** (text, images, audio, video).
> 

**Applications:**

| **Type** | **Examples** |
| --- | --- |
| **Text (LLMs)** | ChatGPT, Gemini, Copilot |
| **Image generation** | Midjourney, DALL-E |
| **Video generation** | OpenAI Sora, Google Veo 2 |

| **✅ Advantages** | **❌ Disadvantages** |
| --- | --- |
| Great for creative tasks and automating content creation | Risk of misuse: deepfakes, plagiarism, copyright issues |
| Can take over repetitive tasks like customer service | Output quality depends heavily on training data quality |
| — | **Hallucinations** — can generate convincing but completely incorrect information |

> ⚠️ Do not rely on ChatGPT or other LLMs for CCNA study — hallucinations make them unreliable for technical accuracy.
> 

---

**🌐 AI in Networking**

**Predictive AI applications**

- **Traffic forecasting** — analyze past traffic to predict future patterns; optimize bandwidth/QoS
- **Security threat detection** — detect anomalies indicating an attack in progress or imminent
- **Predictive maintenance** — anticipate hardware failures; replace faulty devices proactively

**Generative AI applications**

- **Documentation generation** — describe your network; AI generates documentation
- **Configuration generation** — provide requirements; AI generates device configs (always verify before applying)
- **Network design** — AI suggests topology designs tailored to specific needs
- **Troubleshooting assistance** — paste log files or error messages; AI explains causes and remediation steps
- **Script generation** — automatically generate Python or other automation scripts

---

**🛠️ AI Features in Cisco Catalyst Center**

> Formerly known as **DNA Center**. These features are good to know but not all are explicitly listed in CCNA exam topics.
> 

**1. AI Network Analytics**

- Umbrella term for ML-based features that establish the **baseline behavior** of the network ("normal").
- Provides insights and recommendations for optimization.
- Monitors for **anomalies** (deviations from baseline) — e.g., abnormal DHCP lease times, authentication delays, Wi-Fi association times.

**2. Machine Reasoning Engine (MRE)**

- Uses AI to perform **root-cause analysis** of network issues.
- Suggests resolutions or takes **automated corrective actions**.
- Reduces downtime by identifying and resolving issues faster than traditional manual troubleshooting.

**3. AI Endpoint Analytics**

- **Identifies and classifies** devices connected to the network.
- Detects unauthorized devices or unusual/malicious behavior.
- Automates **device profiling and segmentation** — when a device connects, Catalyst Center identifies it and applies the appropriate security policy.

**4. AI-Enhanced Radio Resource Management (RRM)**

- Optimizes **Wi-Fi performance** by dynamically adjusting AP radio settings.
- Balances load among APs.
- Reduces interference via intelligent channel selection.
- Automatically adjusts **transmit power** of each AP to improve coverage.

---

**📋 Key Terms to Memorize**

| **Term** | **Definition** |
| --- | --- |
| **AI** | Computers simulating human intelligence (pattern recognition, learning, decisions) |
| **ML** | Subset of AI; computers learn from data without explicit programming |
| **Deep Learning** | ML subset using multi-layered artificial neural networks; mimics the brain |
| **Neural network** | Computational model of interconnected nodes (neurons) in layers |
| **Labeled data** | Training data with correct answers pre-assigned (used in supervised learning) |
| **Unlabeled data** | Training data with no pre-assigned answers (used in unsupervised learning) |
| **Agent** | The model in reinforcement learning that interacts with the environment |
| **LLM** | Large Language Model — a type of generative AI for text (e.g., ChatGPT) |
| **Hallucination** | When an AI generates confident but factually incorrect output |
| **MRE** | Machine Reasoning Engine — Catalyst Center AI feature for root-cause analysis |
| **Predictive AI** | Analyzes historical data to forecast future outcomes |
| **Generative AI** | Creates new content (text, images, audio) based on learned patterns |

---

# Day 60 (JSON, XML, & YAML)

**📄 JSON, XML & YAML — CCNA Study Notes**

> **Exam Topic:** 6.7 — Interpret JSON encoded data **Scope:** JSON is the primary focus. XML and YAML are secondary (know basic characteristics only).
> 

---

**⚡ Key Takeaways**

- **Data serialization** converts data into a standardized format so different applications (written in different languages) can exchange it.
- **JSON** = the primary format to know for the CCNA. Used heavily by REST APIs. Human-readable and machine-readable.
- JSON has **4 primitive types**: `string`, `number`, `boolean`, `null` — and **2 structured types**: `object` (curly brackets `{}`) and `array` (square brackets `[]`).
- **Strings** always use double quotes. **Numbers** and **booleans** do NOT use quotes. `true`/`false` are lowercase.
- **Objects** = unordered key-value pairs inside `{}`. Keys must always be strings. Values can be any valid JSON type.
- **Arrays** = ordered list of values inside `[]`. Values don't need to be the same type.
- **No trailing comma** after the last key-value pair in an object or last value in an array — this is a common JSON error.
- **Whitespace is insignificant in JSON and XML**, but **significant in YAML** (indentation matters).
- XML uses `<key>value</key>` tags — similar to HTML. Also used by REST APIs.
- YAML is used by **Ansible**. Starts with `--`. Lists use . Key-value pairs use `key: value`.

---

**🔄 What is Data Serialization?**

**Data serialization** = converting data into a standardized format that can be:

- **Stored** (e.g., saved as a file)
- **Transmitted** (e.g., sent over a network)
- **Reconstructed** by a different application

**Why it's needed**

- Different applications may be written in different languages (Python, Java, etc.) and store data differently.
- A standard format like JSON lets them communicate without misunderstanding each other.

**Without serialization**

```
App (Python) → sends raw variables → Server (Java) → ❌ can't understand the data
```

**With serialization (JSON)**

```
App → GET request → API converts variables to JSON → Server receives JSON → ✅ converts to its own format
```

> Variables are containers that store values. e.g., `interface_name = "GigabitEthernet1/1"` or `status = "up"`.
> 

---

**🟡 JSON — JavaScript Object Notation**

**JSON** is an open standard data interchange format that uses **human-readable text** to store and transmit data objects.

- Standardized in **RFC 8259**
- Derived from JavaScript but **language-independent**
- Widely used by **REST APIs** (reason Cisco includes it in the CCNA)
- **Whitespace is insignificant** — spaces and line breaks have no meaning; used only for readability

---

**🔢 JSON Data Types**

**Primitive Types (4 total)**

| **Type** | **Description** | **Example** | **Notes** |
| --- | --- | --- | --- |
| **String** | Text value | `"GigabitEthernet1/1"` | Always wrapped in **double quotes** |
| **Number** | Numeric value | `1000`, `5` | No quotes — `"5"` is a string, `5` is a number |
| **Boolean** | True or false | `true`, `false` | No quotes, always **lowercase** |
| **Null** | No value / intentional absence | `null` | No quotes, lowercase — `"null"` in quotes is a string |

> ⚠️ `"true"` in double quotes = a string (the word "true"). `true` without quotes = a boolean value.
> 

---

**Structured Types (2 total)**

**Object `{}`**

> An **unordered** collection of **key-value pairs** (variables).
> 

**Rules:**

- Wrapped in **curly brackets** `{}`
- **Key** must always be a **string** (double quotes)
- **Value** can be any valid JSON data type (string, number, boolean, null, object, array)
- Key and value separated by a **colon** `:`
- Multiple pairs separated by **commas** `,`
- **No comma after the last key-value pair**
- Also called a **dictionary** (same thing, different name)

**Example:**

```
{
  "interface": "GigabitEthernet1/1",
  "is_up": true,
  "ip_address": "192.168.1.1",
  "netmask": "255.255.255.0",
  "speed": 1000
}
```

**Nested objects** = an object as the value of another object:

```
{
  "device": {
    "name": "R1",
    "vendor": "Cisco",
    "model": "1101"
  },
  "interface": {
    "name": "GigabitEthernet0/0",
    "status": "up"
  }
}
```

---

**Array `[]`**

> An **ordered** series of values separated by commas.
> 

**Rules:**

- Wrapped in **square brackets** `[]`
- Values do NOT need to be the same data type
- Values separated by **commas**
- **No comma after the last value**

**Example:**

```
{
  "interfaces": ["GigabitEthernet1/1", "GigabitEthernet1/2", "GigabitEthernet1/3"],
  "misc": ["Hi", 5]
}
```

> Values in an array can be strings, numbers, booleans, nulls, objects, or even other arrays.
> 

---

**✅ JSON Quick Reference**

| **Rule** | **Detail** |
| --- | --- |
| Strings | Double quotes `"value"` |
| Numbers | No quotes `42` |
| Booleans | `true` or `false` — no quotes, lowercase |
| Null | `null` — no quotes, lowercase |
| Object | `{ "key": value }` — curly brackets |
| Array | `[ value1, value2 ]` — square brackets |
| Key-value separator | Colon `:` |
| Multiple pairs/values | Separated by commas `,` |
| Last item in object/array | **No trailing comma** |
| Whitespace | Insignificant — for human readability only |
| Object = | Also called a **dictionary** |

---

**❌ Common JSON Errors to Watch For**

| **Error** | **Example** |
| --- | --- |
| Comma instead of colon between key and value | `"interfaces" , ["Gi1/1"]` ← should be `:` |
| Colon after a value (not between key-value) | `5:`  ← invalid |
| Missing comma between array values | `["Gi1/1" "Gi1/2"]` ← needs `,` |
| Trailing comma after last item | `{"key": "value",}` ← remove the last `,` |
| Missing closing bracket | `{"routes": [` ← needs `]` |
| Boolean in quotes | `"true"` ← this is a string, not a boolean |

---

**🔵 XML — eXtensible Markup Language**

- Originally a **markup language** (like HTML); now used as a general-purpose data serialization format.
- Also commonly used by **REST APIs** (same as JSON).
- **Less human-readable** than JSON, but still manageable when formatted with indentation.
- **Whitespace is insignificant** (same as JSON).
- Key-value pairs use **HTML-like tags**:

```
<interface>GigabitEthernet0/0</interface>
<ip-address>192.168.1.1</ip-address>
<status>up</status>
```

**Format:** `<key>value</key>`

On Cisco IOS, you can display output in XML:

```
show ip interface brief | format
```

---

**🟢 YAML — YAML Ain't Markup Language**

> Recursive acronym (the acronym contains itself). Originally "Yet Another Markup Language" — renamed to clarify it's a data serialization language, not a markup language.
> 

**Key facts:**

- Used by **Ansible** (network automation tool — covered in a later video)
- Most **human-readable** of the three formats
- **Whitespace IS significant** — indentation is critical and cannot be arbitrary
- YAML files start with `--` (three hyphens)
- Lists indicated by  (one hyphen)
- Key-value pairs: `key: value`

**Example:**

```
---
interfaces:
  - name: GigabitEthernet0/0
    ip_address: 192.168.1.1
    status: up
  - name: GigabitEthernet0/1
    ip_address: 10.0.0.1
    status: down
```

---

**📊 Comparison Table**

| **Feature** | **JSON** | **XML** | **YAML** |
| --- | --- | --- | --- |
| **Whitespace** | Insignificant | Insignificant | **Significant** (indentation required) |
| **Human readability** | High | Medium | Highest |
| **Used by REST APIs** | ✅ Yes | ✅ Yes | ❌ Not commonly |
| **Used by Ansible** | ❌ | ❌ | ✅ Yes |
| **Key-value format** | `"key": value` | `<key>value</key>` | `key: value` |
| **Structured data** | `{}` objects, `[]` arrays | Nested tags | Indentation + `-` lists |
| **CCNA exam focus** | ⭐⭐⭐ Primary | Secondary | Secondary |

---

**📋 Key Terms**

| **Term** | **Definition** |
| --- | --- |
| **Data serialization** | Converting data to a standard format for storage or transmission |
| **Variable** | A named container that holds a value |
| **JSON** | JavaScript Object Notation — primary data format for REST APIs |
| **Object** | JSON structured type using `{}` — unordered key-value pairs; also called a dictionary |
| **Array** | JSON structured type using `[]` — ordered list of values |
| **Nested object** | An object as the value inside another object |
| **Boolean** | JSON type with only two values: `true` or `false` |
| **Null** | JSON type representing intentional absence of a value |
| **REST API** | Uses JSON (and XML) to exchange data between applications |
| **Ansible** | Network automation tool that uses YAML |
| **Recursive acronym** | An acronym where the acronym itself is part of its own definition (e.g., YAML) |

---

# Day 61 (REST APIs (Part 1))

**🔌 REST APIs — CCNA Study Notes**

> **Exam Topic:** 6.5 — Describe characteristics of REST-based APIs **Scope:** Foundational understanding only — no mastery of REST API development required.
> 

---

**⚡ Key Takeaways**

- An **API (Application Programming Interface)** is a software interface that allows two applications to communicate with each other.
- **REST APIs** use **HTTP** as their communication protocol and are the most common choice for the **Northbound Interface (NBI)** in SDN architecture.
- **CRUD** = Create, Read, Update, Delete — the four operations performed via REST APIs, each mapped to an HTTP verb.
- HTTP verb mapping: `POST` = Create, `GET` = Read, `PUT/PATCH` = Update, `DELETE` = Delete.
- HTTP response code classes: **1xx** = Informational, **2xx** = Success, **3xx** = Redirect, **4xx** = Client Error, **5xx** = Server Error.
- **404** = resource not found. **200** = success. **201** = created. **403** = authentication required.
- Three key REST constraints: **Client-Server**, **Stateless**, **Cacheable**.
- **Stateless** = each API request is independent; the server does not store state from previous requests; each request must re-authenticate.
- REST is a *framework for APIs*, not a specific protocol. HTTP is the protocol REST APIs commonly use — they are not the same thing.
- A **URI** has three main parts: scheme (protocol), authority (address/hostname), and path (resource location).

---

**🔄 What is an API?**

An **API (Application Programming Interface)** is a software interface that allows two applications to communicate with each other.

- Essential for network automation — allows programs to interact with network devices and controllers.
- In SDN architecture:
    - **Northbound Interface (NBI)**: apps ↔ SDN controller — typically uses **REST APIs**
    - **Southbound Interface (SBI)**: SDN controller ↔ network devices — uses protocols like NETCONF, RESTCONF, OpenFlow, etc.

---

**📋 CRUD Operations**

**CRUD** describes the four fundamental operations performed when interacting with data via REST APIs:

| **Letter** | **Operation** | **Description** |
| --- | --- | --- |
| **C** | **Create** | Create a new variable and set its initial value |
| **R** | **Read** | Retrieve the value of a variable |
| **U** | **Update** | Change the value of an existing variable |
| **D** | **Delete** | Remove a variable entirely |

---

**🌐 HTTP Verbs (Methods)**

REST APIs use HTTP, and HTTP uses **verbs** that map directly to CRUD operations. When an HTTP client sends a request, it includes a verb indicating the desired action.

| **HTTP Verb** | **CRUD Operation** | **Description** |
| --- | --- | --- |
| **POST** | Create | Create a new resource |
| **GET** | Read | Retrieve a resource |
| **PUT** | Update | Replace/update an existing resource (can also create) |
| **PATCH** | Update | Partially update an existing resource |
| **DELETE** | Delete | Remove a resource |

> **Note:** `PUT` is typically an update operation, though it can sometimes create new resources.
> 

---

**📨 HTTP Request Structure**

When a REST client sends a request, it includes:

| **Component** | **Description** | **Example** |
| --- | --- | --- |
| **HTTP Verb** | The action to perform | `GET` |
| **URI** | The resource being accessed | `https://api.example.com/devices` |
| **Headers** | Additional metadata for the server | `Accept: application/json` |
| **Body** | Optional data payload (e.g., for POST/PUT) | JSON-formatted data |

**URI Structure**

```
https://sandboxdnac.cisco.com/dna/intent/api/v1/network-device
│────┘  │───────────────────┘ │──────────────────────────────┘
Scheme       Authority                     Path
```

| **Part** | **Description** |
| --- | --- |
| **Scheme** | The protocol used — typically `http` or `https` |
| **Authority** | The address or hostname of the server |
| **Path** | The specific resource being accessed |

> A **URL** (Uniform Resource Locator) is a specific type of **URI** (Uniform Resource Identifier). The terms are often used interchangeably.
> 

---

**📬 HTTP Response Codes**

The server's response includes a **status code** indicating the outcome of the request.

| **Code Class** | **Category** | **Meaning** |
| --- | --- | --- |
| **1xx** | Informational | Request received; still processing |
| **2xx** | ✅ Success | Request was successfully received, understood, and accepted |
| **3xx** | Redirection | Further action needed to complete the request |
| **4xx** | ❌ Client Error | Error in the request; cannot be fulfilled |
| **5xx** | ⚠️ Server Error | Request was valid but server could not fulfill it |

**Common Response Codes to Memorize**

| **Code** | **Name** | **Meaning** |
| --- | --- | --- |
| **102** | Processing | Server received the request and is processing it |
| **200** | OK | Request succeeded |
| **201** | Created | Request succeeded and a new resource was created (response to POST) |
| **301** | Moved Permanently | Resource has been moved; new location provided |
| **403** | Unauthorized (Unauthenticated) | Client must authenticate to receive a response |
| **404** | Not Found | The requested resource does not exist on the server |
| **500** | Internal Server Error | Server encountered an unexpected error |

> **Memory tip:** 2xx = good. 4xx = your fault (client error). 5xx = their fault (server error). 404 = page not found — you've likely seen this on the web.
> 

---

**🏗️ REST API Characteristics**

**REST** = Representational State Transfer. REST is a **framework** (set of rules/constraints) for building APIs — not a specific protocol.

Also called: **REST-based APIs** or **RESTful APIs**

**The 6 REST Constraints (know these 3 for CCNA)**

**1. Client-Server**

- REST APIs use a **client-server architecture**.
- The client accesses resources on the server via API calls (HTTP requests).
- Client and server are **independent** — they can evolve separately without breaking the interface between them.

**2. Stateless**

- Each API exchange is a **separate, independent event**.
- The server does **not store information** about previous requests.
- Every request must be **self-contained** — if authentication is required, the client must authenticate with every single request.

| **Stateful** | **Stateless** |
| --- | --- |
| TCP — tracks sequence numbers, connections | UDP — no connection or tracking |
| Maintains state between exchanges | Each exchange is independent |

> ⚠️ Even though REST APIs use HTTP (which uses TCP at Layer 4), the application-layer REST/HTTP exchange is stateless. OSI layers are independent.
> 

**3. Cacheable (or Non-Cacheable)**

- REST APIs must **support caching** — storing data for future use to improve performance and reduce server load.
- Not all resources must be cacheable, but resources that are cacheable must be **declared as cacheable** when sent to the client.

**Other constraints (less critical for CCNA)**

- **Uniform Interface** — standard way of interacting with resources
- **Layered System** — the client doesn't need to know whether it's talking to the final server or an intermediate layer
- **Code-on-demand** (optional) — server can send executable code to the client to extend functionality

---

**🆚 REST vs. HTTP Clarification**

| **Concept** | **What it is** |
| --- | --- |
| **REST** | A framework/set of rules for designing APIs |
| **HTTP** | A communication protocol — the most common protocol used by REST APIs |
| **HTTPS** | Encrypted version of HTTP — more commonly used in practice |

> REST APIs do **not have to use HTTP**, but HTTP (or HTTPS) is by far the most common choice. For CCNA purposes, assume REST APIs use HTTP/HTTPS.
> 

---

**🔧 Cisco DevNet & Postman (Practical Reference)**

**Cisco DevNet** (developer.cisco.com) — free developer resources: courses, labs, sandboxes, API documentation.

**Postman** — platform for building and testing REST API calls (free, desktop or browser).

**Sample REST API Call to Cisco DNA Center (DevNet Sandbox)**

**Step 1: Get an auth token (POST)**

| **Field** | **Value** |
| --- | --- |
| Verb | `POST` |
| URL | `https://sandboxdnac.cisco.com/dna/system/api/v1/auth/token` |
| Auth type | Basic Auth |
| Username | `devnetuser` |
| Password | `Cisco123!` |
| Expected response | `200 OK` + JSON with `Token` key |

**Step 2: Get device inventory (GET)**

| **Field** | **Value** |
| --- | --- |
| Verb | `GET` |
| URL | `https://sandboxdnac.cisco.com/dna/intent/api/v1/network-device` |
| Header key | `X-Auth-Token` |
| Header value | Token from Step 1 |
| Expected response | `200 OK` + JSON device inventory |

> Response is JSON-formatted data — device family, role, hostname, platform ID, uptime, etc. Applications can parse this data, modify it, and push changes back via the API.
> 

---

**📋 Key Terms to Memorize**

| **Term** | **Definition** |
| --- | --- |
| **API** | Application Programming Interface — software interface enabling two apps to communicate |
| **REST** | Representational State Transfer — framework for building APIs |
| **CRUD** | Create, Read, Update, Delete — the four data operations |
| **HTTP verb** | The action included in an HTTP request (GET, POST, PUT, PATCH, DELETE) |
| **URI** | Uniform Resource Identifier — identifies the resource being accessed |
| **URL** | Uniform Resource Locator — a specific type of URI |
| **Stateless** | Each API request is independent; server stores no prior state |
| **Caching** | Storing data locally for future use to reduce load and improve performance |
| **NBI** | Northbound Interface — typically uses REST APIs (apps ↔ SDN controller) |
| **SBI** | Southbound Interface — uses NETCONF, RESTCONF, OpenFlow, etc. (controller ↔ devices) |
| **Cisco DevNet** | Cisco's free developer resource and sandbox environment |
| **DNA Center** | Cisco's SDN controller — provides REST API access to network data |

---

# Day 61 (REST API Authentication (Part 2))

**🔐 REST API Authentication — CCNA Study Notes**

> **Exam Topic:** 6.5 — Describe characteristics of REST-based APIs (includes authentication methods) **Scope:** Understand the four main REST authentication methods conceptually — no implementation required.
> 

---

**⚡ Key Takeaways**

- **Authentication** = validating the identity of a user/system to ensure only legitimate access to API resources.
- Without authentication, anyone can send API requests, potentially reading or modifying sensitive data.
- **Four authentication methods to know:** Basic, Bearer, API Key, and OAuth 2.0.
- **Basic auth** = username + password encoded in Base64 (not encrypted!) sent with every request. Simple but least secure.
- **Bearer auth** = token obtained from an auth server, included in every request. More secure — tokens expire automatically.
- **API key auth** = static key issued by the API provider. Good for usage tracking. Less secure — does not expire automatically.
- **OAuth 2.0** = access delegation framework. Third-party apps get limited access to resources without ever seeing the user's credentials. Used by "Log in with Google."
- **Base64 is encoding, not encryption** — it is easily reversible. Always use **HTTPS** with any REST API auth method.
- **Bearer and OAuth 2.0** both involve an **auth server issuing tokens**. Basic auth and API keys do not.
- **API keys should never be included in URLs** — always use the HTTP Authorization header.
- **OAuth 2.0 refresh tokens** allow continued access without repeated user logins when access tokens expire.

---

**🔒 Why REST API Authentication Matters**

- APIs expose applications and data to other programs — without authentication, anyone can access or modify that data.
- **Authentication asks: "Who are you?"** — If it fails, the request is denied.
- Some APIs are intentionally open (public data, no auth required). Most APIs should be secured.
- APIs are also used for **usage tracking and billing** (e.g., OpenAI charges per token usage — API keys enable this tracking).

---

**🔑 Method 1: Basic Authentication**

**How it works:**

- Sends a **username and password** in the HTTP `Authorization` header with every API request.
- Credentials are encoded in **Base64** format: `username:password` → Base64 string.
- Example: `jeremy:ccna` → Base64 encoded string in the header.

```
Authorization: Basic amVyZW15OmNjbmE=
```

> ⚠️ **Base64 is encoding, not encryption.** It is easily reversible — not secure on its own.
> 

**Flow:**

```
Client → API request with Authorization header (Base64 credentials) → Server
Server → Validates credentials → Responds with data (or denies)
```

| **✅ Advantages** | **❌ Disadvantages** |
| --- | --- |
| Simple and easy to implement | Credentials sent with every request — vulnerable if connection isn't secured |
| No token management needed | If credentials are stolen, full access is granted |
| — | Must use HTTPS — credentials are not encrypted at the application layer |

---

**🪙 Method 2: Bearer Authentication**

**How it works:**

- Uses a **token** (called a bearer token) instead of credentials in every request.
- Client first **authenticates with an auth server** (using basic auth or another method) to obtain the token.
- Token is included in the `Authorization` header for each subsequent API request.
- The term **"bearer"** means anyone who possesses the token can use it.

```
Authorization: Bearer <token>
```

**Flow:**

```
1. Client → Auth server: Request token (with credentials)
2. Auth server → Client: Issues bearer token
3. Client → Resource server: API request with bearer token in header
4. Resource server → Client: Responds with resource
```

> Tokens have an **expiration time** — typically 15 minutes to weeks depending on security requirements.
> 

| **✅ Advantages** | **❌ Disadvantages** |
| --- | --- |
| More secure than basic auth — credentials not sent repeatedly | Stolen token grants access until it expires |
| Tokens expire automatically | Token refresh adds implementation complexity |
| — | Must use HTTPS |

---

**🗝️ Method 3: API Key Authentication**

**How it works:**

- The API provider issues a **unique, static key** to each client.
- Client includes the key in each API request.
- Key **does not expire automatically** — remains valid until manually revoked.

**Three ways to send the key (recommended first):**

| **Method** | **Notes** |
| --- | --- |
| ✅ HTTP Authorization header | Recommended — secure |
| ⚠️ URL parameter (query string) | Not recommended — URLs are logged by servers, proxies, and browsers, exposing the key |
| ⚠️ Cookie | Sometimes used for browser-based APIs |

```
Authorization: <api-key>
```

**Flow:**

```
Client → API request with API key in Authorization header → Server
Server → Validates key → Responds with resource
```

> **Third-party API example:** A chatbot app calls OpenAI's API to get ChatGPT responses. The developer's API key identifies them as the "second party" — OpenAI (the third party) tracks usage and charges via that key.
> 

| **✅ Advantages** | **❌ Disadvantages** |
| --- | --- |
| Easier to implement than bearer auth | Static — does not expire automatically |
| Excellent for usage tracking and billing | Stolen key grants full access until manually revoked |
| Common in cloud services and third-party APIs | Must use HTTPS |

---

**🔓 Method 4: OAuth 2.0**

**Purpose:** Access **delegation** — allows third-party applications to access a user's resources **without exposing the user's credentials**.

> Classic example: **"Log in with Google"** — the third-party website never sees your Google password. Google confirms your identity and grants limited access.
> 

**Four Parties in OAuth 2.0**

| **Party** | **Role** | **Example** |
| --- | --- | --- |
| **Resource owner** | The account owner granting access | You (Google account owner) |
| **Client app** | Third-party app requesting access | A scheduling app |
| **Auth server** | Issues access tokens after approval | Google's OAuth service |
| **Resource server** | Hosts the protected resource | Google Calendar server |

**OAuth 2.0 Flow (6 steps)**

```
1. Client app → Resource owner: Requests authorization
2. Resource owner → Client app: Grants authorization (user logs into Google and approves)
3. Client app → Auth server: Exchanges authorization grant for access token
4. Auth server → Client app: Issues access token (limited scope, e.g., read-only calendar)
5. Client app → Resource server: API request with access token
6. Resource server → Client app: Validates token and responds with requested resource
```

**Key Token Concepts**

| **Token** | **Purpose** |
| --- | --- |
| **Access token** | Temporary key granting limited access to a specific resource (e.g., read-only calendar). Expires quickly. |
| **Refresh token** | Issued by the auth server; allows the client to request a new access token automatically without requiring the user to log in again. |

> OAuth 2.0 is defined in **RFC 6749**.
> 

| **✅ Advantages** | **❌ Disadvantages** |
| --- | --- |
| User credentials are never shared with third-party apps | More complex to implement than other methods |
| Fine-grained access control (scope) | Requires an auth server infrastructure |
| Access tokens expire + refresh tokens avoid repeated logins | — |
| Widely used and trusted in modern web apps | — |

---

**📊 Comparison Table**

| **Feature** | **Basic Auth** | **Bearer Auth** | **API Key** | **OAuth 2.0** |
| --- | --- | --- | --- | --- |
| **Credential type** | Username + password | Token | Static key | Access token + refresh token |
| **Sent with every request?** | ✅ Yes | ✅ Yes (token) | ✅ Yes (key) | ✅ Yes (token) |
| **Auth server required?** | ❌ No | ✅ Yes | ❌ No | ✅ Yes |
| **Expires automatically?** | ❌ No | ✅ Yes | ❌ No | ✅ Yes (access token) |
| **Good for usage tracking?** | ❌ | ❌ | ✅ Yes | ❌ |
| **Supports access delegation?** | ❌ | ❌ | ❌ | ✅ Yes |
| **Security level** | Lowest | Medium | Medium-Low | Highest |
| **Complexity** | Lowest | Medium | Low | Highest |
| **Must use HTTPS?** | ✅ Yes | ✅ Yes | ✅ Yes | ✅ Yes |

---

**⚠️ Universal Rules for All Methods**

- Always use **HTTPS** (HTTP + TLS) — never send authentication credentials or tokens over plain HTTP.
- Base64 encoding ≠ encryption. It is reversible and provides no security on its own.
- API keys must **never be included in URLs** — use the HTTP `Authorization` header instead.

---

**📋 Key Terms**

| **Term** | **Definition** |
| --- | --- |
| **Authentication** | Verifying the identity of a user or system to control access |
| **Base64** | An encoding scheme (not encryption) — easily reversible |
| **Bearer token** | A token where possession = permission to access the resource |
| **API key** | A static, unique key issued by an API provider — does not auto-expire |
| **Access token** | Short-lived token granting limited access in OAuth 2.0 |
| **Refresh token** | OAuth 2.0 token used to get new access tokens without user re-login |
| **Auth server** | Server that issues tokens (used in bearer auth and OAuth 2.0) |
| **Resource server** | Server hosting the protected resource the client wants to access |
| **Access delegation** | Granting a third party limited access to resources on behalf of the owner — core purpose of OAuth 2.0 |
| **Scope** | The specific level of access granted by an OAuth 2.0 token (e.g., read-only) |
| **HTTPS** | HTTP + TLS — provides encryption; required for all REST API authentication |

---

# Day 62 (Software-Defined Networking)

> **Exam Topics:** 6.3 and 6.4 — SDN architecture, Cisco SD-Access, DNA Center **Scope:** Conceptual understanding — no hands-on SDN configuration required for CCNA.
> 

---

**⚡ Key Takeaways**

- SDN centralizes the control plane into a **controller**. Three SDN architecture layers: **Application → Control → Infrastructure**.
- **Underlay** = physical network of switches and connections (provides IP connectivity, e.g., via IS-IS).
- **Overlay** = virtual network of tunnels built on top of the underlay (VXLAN tunnels in SD-Access).
- **Fabric** = underlay + overlay combined — the complete SD-Access network.
- Three SD-Access switch roles: **Edge node** (connects end hosts), **Border node** (connects outside the fabric), **Control node** (runs LISP for the control plane).
- SD-Access uses three key protocols: **LISP** (control plane), **VXLAN** (data plane / tunnels), **Cisco TrustSec** (policy / QoS / security).
- In an optimal SD-Access underlay (greenfield): all links are **Layer 3**, **IS-IS** is the routing protocol, **no STP**, and **access switches are the default gateway** (routed access layer).
- **DNA Center** is the SDN controller for SD-Access AND can be used as a general network management platform in traditional networks.
- DNA Center enables **intent-based networking (IBN)** — engineer states the desired behavior, DNA Center handles the implementation.
- DNA Center's southbound interface supports: **NETCONF, RESTCONF, SSH, Telnet, SNMP**.

---

**🔄 SDN Review**

**Software-Defined Networking (SDN)** centralizes the control plane into a software **controller**.

| **Traditional (Distributed)** | **SDN (Centralized)** |
| --- | --- |
| Each device runs its own control plane | Control plane functions centralized in the controller |
| Devices use OSPF to share routing info with each other | Devices share info with the controller, which calculates routes |
| Policies (ACLs, etc.) configured per-device | Policies defined centrally, pushed to devices |

**SDN interfaces:**

- **Southbound Interface (SBI)** — controller ↔ network devices
- **Northbound Interface (NBI)** — apps/scripts ↔ controller (REST API)

---

**🏗️ SDN Architecture Layers**

> These are SDN-specific layers — not the OSI model.
> 

| **Layer** | **What's in it** | **Description** |
| --- | --- | --- |
| **Application layer** (top) | Scripts, apps, tools | Communicates desired network behavior to the controller |
| **Control layer** (middle) | SDN controller | Receives and processes instructions; houses the centralized control plane |
| **Infrastructure layer** (bottom) | Network devices (switches, routers) | Responsible for actually forwarding traffic |

---

**🧩 Cisco SDN Solutions (Overview)**

| **Solution** | **Purpose** |
| --- | --- |
| **SD-Access** | Automates **campus LANs** (wired + wireless office networks) |
| **ACI** (Application Centric Infrastructure) | Automates **data center networks** (uses spine-leaf architecture) |
| **SD-WAN** | Automates **WAN connections** |

---

**📐 SD-Access Architecture**

**Cisco SD-Access** = Cisco's SDN solution for campus LANs. **Controller:** Cisco **DNA Center** (Digital Network Architecture Center) sits in the control layer.

```
┌──────────────────────────────────┐
│      Application Layer            │  Scripts, apps, DNA Center GUI
└──────────────┬───────────────────┘
               │ NBI (REST API)
┌──────────────▼───────────────────┐
│         Control Layer             │  DNA Center
└──────────────┬───────────────────┘
               │ SBI (NETCONF, RESTCONF, SSH, etc.)
┌──────────────▼───────────────────┐
│      Infrastructure Layer         │  Campus LAN switches (SD-Access fabric)
└──────────────────────────────────┘
```

---

**🔻 SD-Access Underlay**

> The **underlay** is the **underlying physical network** — the switches and their connections that provide IP connectivity.
> 

**Underlay in a Greenfield Deployment (DNA Center configures it)**

| **Feature** | **Value** |
| --- | --- |
| All switch links | **Layer 3** (routed ports — no Layer 2) |
| Routing protocol | **IS-IS** |
| STP required? | ❌ No — no Layer 2 loops possible |
| FHRP required? | ❌ No |
| Default gateway | **Edge nodes (access switches)** — "routed access layer" |

**Traditional LAN vs. SD-Access Underlay**

| **Traditional LAN** | **SD-Access Underlay** |
| --- | --- |
| STP used to prevent Layer 2 loops | No STP needed — all links are Layer 3 |
| HSRP/VRRP at distribution layer for redundant gateway | Edge switches ARE the default gateway |
| OSPF typically used for routing | IS-IS used for routing |

**Three Switch Roles in SD-Access**

| **Role** | **Description** |
| --- | --- |
| **Edge node** | Connects to end hosts (like a traditional access layer switch) |
| **Border node** | Connects to devices outside the SD-Access domain (e.g., WAN router) |
| **Control node** | Runs **LISP** for control plane functions |

> ⚠️ There is no "management node" — a common distractor answer.
> 

**Brownfield vs. Greenfield**

| **Type** | **Description** |
| --- | --- |
| **Greenfield** | Brand-new network built for SD-Access. DNA Center configures the optimal underlay. |
| **Brownfield** | SD-Access added onto an existing network. DNA Center does NOT configure the underlay (too risky for production). |

---

**🔼 SD-Access Overlay**

> The **overlay** is the **virtual network** built on top of the physical underlay using tunnels.
> 

**Three Key Protocols**

| **Protocol** | **Role** | **Details** |
| --- | --- | --- |
| **LISP** (Locator ID Separation Protocol) | **Control plane** | Maintains mappings of EIDs → RLOCs. DNS-like system for locating hosts. |
| **VXLAN** (Virtual Extensible LAN) | **Data plane** | Creates the tunnels that carry traffic. "Extensible" = supports many features. |
| **Cisco TrustSec (CTS)** | **Policy** | QoS and security policy enforcement across the fabric. |

**LISP Terms**

| **Term** | **Meaning** |
| --- | --- |
| **EID** (Endpoint Identifier) | Identifies an end host connected to an edge switch |
| **RLOC** (Routing Locator) | Identifies the edge switch used to reach that end host |

**How VXLAN + LISP Work Together (Example)**

```
1. PC2 (connected to SW2) registers with SW3 (control node): "PC2 is reachable via SW2"
2. PC1 wants to send traffic to PC2 → sends to default gateway SW1
3. SW1 asks SW3: "How do I reach PC2?"
4. SW3 replies: "PC2 is reachable via SW2"
5. SW1 forwards traffic over a VXLAN tunnel to SW2 → SW2 delivers to PC2
```

---

**🔷 Fabric = Underlay + Overlay**

```
Fabric = Underlay (physical switches + connections) + Overlay (VXLAN tunnels)
```

| **Term** | **Definition** |
| --- | --- |
| **Underlay** | Physical network providing IP connectivity |
| **Overlay** | Virtual tunnel network built on top of the underlay |
| **Fabric** | The complete SD-Access network = underlay + overlay |

---

**🖥️ Cisco DNA Center**

**DNA Center** has two roles:

1. **SDN controller** for SD-Access
2. **General network management platform** for traditional networks (without SD-Access)

> DNA Center runs on **Cisco UCS server hardware** and exposes a **REST API** (northbound).
> 

**Southbound Interface Protocols**

- NETCONF
- RESTCONF
- SSH
- Telnet
- SNMP

**Intent-Based Networking (IBN)**

> Engineer expresses **desired network behavior** → DNA Center translates it into device configurations.
> 

Example: Instead of writing complex ACLs, define group-based policies:

- "Developers group can access Test_Servers"
- "Guest group cannot access any internal servers"

DNA Center implements the required policies across the entire fabric automatically.

**DNA Center GUI Sections**

| **Section** | **Purpose** |
| --- | --- |
| **Design** | Build network hierarchy, manage IPs, subnets, DHCP, DNS |
| **Policy** | Define group-based access control and QoS policies |
| **Provision** | Manage device inventory, add new devices, assign to sites |
| **Assurance** | Monitor device health, performance, and network status |

---

**⚖️ DNA Center vs. Traditional Network Management**

| **Aspect** | **Traditional Management** | **DNA Center Management** |
| --- | --- | --- |
| **Configuration method** | One device at a time via SSH/console | Centrally managed from DNA Center GUI or REST API |
| **New device deployment** | Manual pre-configuration before deployment | Devices auto-receive configs from DNA Center |
| **Policy management** | Distributed — per-device ACLs, manual effort | Centralized intent-based policies |
| **Software updates** | Manual, per-device | Centrally managed; DNA Center monitors for new versions and updates devices |
| **Error risk** | High — manual effort = more human error | Lower — automation reduces configuration mistakes |
| **Deployment speed** | Slow — manual work required | Fast — automated provisioning |
| **Compliance visibility** | Limited | DNA Center flags non-compliant software versions and security advisories |

---

**📋 Key Terms to Memorize**

| **Term** | **Definition** |
| --- | --- |
| **SDN** | Software-Defined Networking — centralizes the control plane into a controller |
| **Underlay** | Physical network of devices and connections (provides IP connectivity) |
| **Overlay** | Virtual tunnel network built on top of the underlay |
| **Fabric** | Underlay + overlay combined |
| **Edge node** | SD-Access switch that connects to end hosts |
| **Border node** | SD-Access switch that connects to devices outside the fabric |
| **Control node** | SD-Access switch that runs LISP for control plane functions |
| **LISP** | Control plane protocol — maps EIDs (host identifiers) to RLOCs (switch locators) |
| **VXLAN** | Data plane protocol — creates virtual tunnels in the overlay |
| **Cisco TrustSec** | Provides QoS and security policy enforcement in SD-Access |
| **DNA Center** | Cisco's SDN controller for SD-Access; also a general network management platform |
| **IBN** | Intent-Based Networking — engineer specifies desired behavior, controller implements it |
| **Greenfield** | New network built specifically for SD-Access; DNA Center configures the underlay |
| **Brownfield** | Existing network with SD-Access added on top; DNA Center does NOT configure the underlay |
| **Routed access layer** | Access switches act as Layer 3 default gateways for end hosts |

---

# Day 63 (Ansible, Puppet, & Chef (Part 1))

> **Exam Topic:** 6.6 — Recognize the capabilities of configuration management mechanisms **Scope:** Understand the purpose and basic characteristics of each tool — no hands-on required.
> 

---

**⚡ Key Takeaways**

- **Configuration management tools** (Ansible, Puppet, Chef) automate the configuration and management of large numbers of network devices (and servers/VMs).
- They solve two key problems: **configuration drift** (devices drifting from standard configs over time) and **configuration provisioning** (applying changes at scale).
- All three use **templates + variables** to generate device configurations.
- **Ansible**: Python, agentless, **push** model, uses SSH, files in YAML. Most popular for network devices.
- **Puppet**: Ruby, typically agent-based, **pull** model, TCP 8140, proprietary language, uses Manifests. Server = "Puppet master."
- **Chef**: Ruby, agent-based, **pull** model, TCP 10002, Ruby-based DSL, uses cookbooks/recipes. Least popular for network devices.
- All three use a **client-server model**.
- Puppet and Chef communicate via **HTTP/REST API**. Ansible uses **SSH**.
- All three were originally designed for VM/server management, not specifically for network devices.

---

**🔀 Why Configuration Management Tools Are Needed**

**Problem 1: Configuration Drift**

**Configuration drift** = when individual changes made over time cause a device's configuration to deviate from the company's standard templates.

- Every device has some standard config (SNMP, Syslog, AAA, interface settings) and some unique config (hostname, IP addresses).
- Engineers making individual changes (troubleshooting, testing) can cause configs to drift.
- Records of changes and reasons are often not kept — makes it hard to audit or revert.
- Manual approaches (saving config files to a shared folder) don't scale in large networks and depend on engineers remembering to do them.

**Problem 2: Configuration Provisioning**

**Configuration provisioning** = how configuration changes are applied to devices (new devices or existing ones).

- Traditional method: SSH/console into each device one-by-one. Not scalable for hundreds/thousands of devices.
- Configuration management tools apply changes at mass scale with minimal time and effort.

**Core Mechanism: Templates + Variables**

All three tools use this pattern:

```
Template (shared config structure)  +  Variables (device-specific values)
         ↓
     Generated config → pushed/pulled to device
```

> Example: Template defines OSPF config structure; variable file specifies the process ID, area, and interface for each specific device.
> 

---

**📊 Comparison Table (Memorize This)**

| **Feature** | **Ansible** | **Puppet** | **Chef** |
| --- | --- | --- | --- |
| **Written in** | Python | Ruby | Ruby |
| **Agent required?** | ❌ Agentless | ✅ Agent-based (can be agentless via proxy) | ✅ Agent-based |
| **Model** | **Push** | **Pull** | **Pull** |
| **Protocol** | SSH | HTTP / REST API | HTTP / REST API |
| **Server name** | Control node | Puppet master | Chef server |
| **Config port** | SSH (22) | TCP **8140** | TCP **10002** |
| **File language** | YAML + Jinja2 | Proprietary language | Ruby-based DSL |
| **Key file type** | Playbook | Manifest | Cookbook / Recipe |
| **Popularity (network mgmt)** | ⭐⭐⭐ 1st | ⭐⭐ 2nd | ⭐ 3rd |

---

**🔵 Ansible**

**Owner:** Red Hat | **Language:** Python | **Model:** Push | **Agent:** None (agentless)

**How it works**

- The **control node** (Ansible server) connects directly to managed devices via **SSH**.
- No special software needs to be installed on the managed devices — this is the biggest advantage.
- Control node pushes configurations to devices.

**Key Files (all written in YAML, except templates)**

| **File** | **Purpose** | **Format** |
| --- | --- | --- |
| **Playbook** | Blueprint of automation tasks — defines logic and actions | YAML |
| **Inventory** | Lists managed devices and their characteristics (role, etc.) | INI or YAML |
| **Template** | Configuration file with placeholders for variables | Jinja2 |
| **Variable file** | Provides values for variables to fill in templates | YAML |

**Ansible Flow**

```
Inventory + Templates + Variables → Playbook → SSH → Managed Devices
```

**Why Ansible is Most Popular for Networks**

- Agentless → no need for compatible agent software on Cisco devices
- Uses SSH → already supported by virtually all network devices
- YAML → easy to read and write

---

**🟠 Puppet**

**Owner:** Puppet, Inc. | **Language:** Ruby | **Model:** Pull | **Agent:** Typically required

**How it works**

- Managed devices have a **Puppet agent** installed, which pulls configurations from the **Puppet master**.
- Not all Cisco devices support a Puppet agent.
- Can run **agentless** using an external proxy agent that connects to devices via SSH on behalf of the Puppet master.
- Clients communicate with Puppet master over **TCP port 8140**.
- Files use a **proprietary language** (not YAML).

**Key Files**

| **File** | **Purpose** |
| --- | --- |
| **Manifest** | Defines the **desired configuration state** of a network device |
| **Template** | Helps generate Manifests for devices |

**Puppet Flow**

```
Puppet Master (Manifests + Templates) ← Pull ← Puppet Agent (on managed device)
                                      ← Pull ← External Proxy Agent (agentless mode)
```

---

**🟢 Chef**

**Owner:** Progress Software | **Language:** Ruby | **Model:** Pull | **Agent:** Required

**How it works**

- Managed devices (**Chef clients**) have a **Chef agent** installed.
- Clients pull configurations from the **Chef server** over **TCP port 10002**.
- Most Cisco devices do NOT support a Chef agent — least popular for network device management.
- Files use a **DSL (Domain-Specific Language) based on Ruby**.
- Admins work on a **Chef workstation** to create cookbooks and recipes, which are stored on the Chef server.

**Key Files (Chef uses a cooking metaphor)**

| **File** | **Description** |
| --- | --- |
| **Resource** | Like an ingredient — defines a configuration object managed by Chef |
| **Recipe** | Outlines logic and actions performed on resources |
| **Cookbook** | A set of related recipes grouped together |
| **Run-list** | Ordered list of recipes run to bring a device to the desired state |

**Chef Flow**

```
Chef Workstation → creates → Cookbook/Recipe → stored on Chef Server
Chef Client (managed device) ← pulls ← Chef Server (TCP 10002)
```

---

**🔑 Push vs. Pull Model**

| **Model** | **Who initiates the connection** | **Used by** |
| --- | --- | --- |
| **Push** | Server pushes configs to clients | **Ansible** |
| **Pull** | Clients connect to server to receive configs | **Puppet**, **Chef** |

---

**📋 Key Terms to Memorize**

| **Term** | **Definition** |
| --- | --- |
| **Configuration drift** | Deviation of a device's config from the company's standard template over time |
| **Configuration provisioning** | Applying configuration changes to devices (new or existing) |
| **Agentless** | No special software required on managed devices (Ansible) |
| **Agent-based** | Requires specific software installed on managed devices (Puppet, Chef) |
| **Push model** | Server initiates and pushes configs to devices (Ansible) |
| **Pull model** | Managed devices connect to server and pull configs (Puppet, Chef) |
| **Playbook** | Ansible's automation blueprint — written in YAML |
| **Manifest** | Puppet's file defining the desired config state of a device |
| **Cookbook** | Chef's collection of related recipes |
| **Recipe** | Chef's file defining tasks performed on resources |
| **Template** | A configuration structure with variable placeholders (used by all three tools) |
| **Jinja2** | Template language used by Ansible |
| **Control node** | Ansible's server |
| **Puppet master** | Puppet's server |
| **Chef server** | Chef's server |

---

# Day 63 (Terraform (Part 2))

**Key Takeaways**

- Infrastructure as Code (IaC) manages infrastructure through machine-readable files instead of manual CLI/GUI configuration
- Terraform is primarily a **provisioning** tool; Ansible, Puppet, and Chef are primarily **configuration management** tools
- Terraform uses an **immutable** and **declarative** approach; Ansible/Chef use a **mutable** and **procedural** approach
- Terraform workflow: **Write → Plan → Apply**
- Terraform Core is written in **Go**; configuration files are written in **HCL** (HashiCorp Configuration Language)
- Terraform is **agentless** and uses a **push model**, similar to Ansible

---

## Infrastructure as Code (IaC)

**Definition:** The practice of provisioning and managing infrastructure using machine-readable configuration files instead of manual CLI or GUI configuration.

**Examples of IaC tools:** Ansible, Puppet, Chef, Terraform

**Benefits:** Consistency, scalability, repeatability, and automation across deployments

---

## Provisioning vs. Configuration Management

|  | **Configuration Management** | **Infrastructure Provisioning** |
| --- | --- | --- |
| **Tools** | Ansible, Puppet, Chef | Terraform |
| **Focus** | Managing existing infrastructure | Creating, modifying, deleting infrastructure |
| **Examples** | Install software, configure settings, maintain state | Deploy VMs, cloud resources, network infrastructure |
| **Starting point** | Existing systems | From scratch |

> Terraform and Ansible are often used **together** — Terraform provisions the infrastructure, Ansible configures and maintains it.
> 

---

## Mutable vs. Immutable Infrastructure

**Mutable Infrastructure** (Ansible, Puppet, Chef)

- Infrastructure **can be modified** after deployment
- Changes are made **in place** — existing resources are updated, not replaced
- Risk of **configuration drift** over time as individual changes accumulate

**Immutable Infrastructure** (Terraform)

- Infrastructure **cannot be changed** after deployment
- Changes require **replacing** the old resource with a new, updated version
- Virtually **eliminates configuration drift** — every deployment starts from a fresh, predefined state

---

## Procedural vs. Declarative Approach

**Procedural (Imperative)** — Ansible, Chef

- Defines **explicit steps in a specific order**
- The engineer must specify exactly what to do and in what order
- Greater control, but more demanding on the user

**Declarative** — Terraform, Puppet

- Defines the **desired end state**
- The tool figures out the steps needed to achieve that state
- Easier to maintain and ensures consistency across deployments

> Example: Instead of writing `hostname R1`, `ip address...`, `no shutdown` step-by-step, you tell Terraform "create a router named R1 with this IP, interface enabled" and it handles the rest.
> 

---

## Terraform Overview

**Developer:** HashiCorp (acquired by IBM in 2025)
**Type:** Open-source IaC provisioning tool
**Model:** Push model, agentless (no agent required on managed infrastructure)

---

## Terraform Key Components

| Component | Description |
| --- | --- |
| **Terraform Core** | Main software (written in Go) that processes configs and interacts with providers |
| **Configuration Files** | Written in HCL; define the desired end state of infrastructure |
| **State File** | Tracks the current state of deployed infrastructure; used to compare current vs. desired state |
| **Providers** | Platforms where infrastructure is deployed (AWS, Azure, GCP, Kubernetes, Cisco IOS XE, ACI, Catalyst Center, 1000+ supported) |

---

## Terraform Workflow

**Step 1 — Write**
Define the desired end state of your infrastructure in HCL configuration files.

**Step 2 — Plan**
Terraform analyzes configuration files and shows exactly what changes will be made. Review before applying.

**Step 3 — Apply**
Terraform executes the plan and provisions/manages the infrastructure.

*(Optional Step 4 — **Destroy**: Delete resources when no longer needed)*

---

## HCL — HashiCorp Configuration Language

- A **Domain-Specific Language (DSL)** designed specifically for Terraform
- Simpler than general-purpose languages like Python or Go
- Requires learning separately, but relatively straightforward
- Used to write Terraform configuration files that define the desired infrastructure state

---

## Tool Comparison Summary

| Feature | Ansible | Puppet | Chef | Terraform |
| --- | --- | --- | --- | --- |
| Primary use | Config management | Config management | Config management | Provisioning |
| Approach | Procedural | Declarative | Procedural | Declarative |
| Infrastructure | Mutable | Mutable | Mutable | Immutable |
| Agent required | No (agentless) | Yes | Yes | No (agentless) |
| Model | Push | Pull | Pull | Push |
| Language | YAML (playbooks) | Puppet DSL | Ruby DSL | HCL |

---