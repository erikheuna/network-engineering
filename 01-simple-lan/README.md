# Lab 1: My First LAN (Packet Tracer)

[Series index](../README.md) · [Next lab →](../02-dhcp)

| File | Content |
|---|---|
| `tp01.pkt` | Packet Tracer file for this lab |
| `configs/` | Device configurations (text) |
| `images/` | Screenshots |

📝 LinkedIn article: [FR](#) · [EN](#)

**Level:** Absolute beginner
**Objective:** Build a small local network with 2 switches and 1 router, assign IPv4 addresses, and verify end-to-end connectivity.
**Estimated time:** 45–60 minutes

---

## 1. Topology

```
        [PC0]                                   [PC2]
          |                                        |
        [Switch0] ---- [Router0] ---- [Switch1]
          |                                        |
        [PC1]                                   [PC3]
```

Devices to place in Packet Tracer:
- 1 router: **Router 2911** (or equivalent)
- 2 switches: **Switch 2960**
- 4 generic PCs
- Cables: **Copper Straight-Through** for PC ↔ Switch and Switch ↔ Router

> **Tip:** if you hesitate on the cable type, use the automatic cabling mode at first. An orange icon on a link usually means the wrong cable was chosen.

---

## 2. IP addressing plan

Two separate subnets, one per switch, joined by the router (inter-network routing).

| Device | Interface | IP address | Mask | Gateway |
|---|---|---|---|---|
| Router0 | Gi0/0 (Switch0 side) | 192.168.1.1 | 255.255.255.0 | n/a |
| Router0 | Gi0/1 (Switch1 side) | 192.168.2.1 | 255.255.255.0 | n/a |
| PC0 | NIC | 192.168.1.10 | 255.255.255.0 | 192.168.1.1 |
| PC1 | NIC | 192.168.1.11 | 255.255.255.0 | 192.168.1.1 |
| PC2 | NIC | 192.168.2.10 | 255.255.255.0 | 192.168.2.1 |
| PC3 | NIC | 192.168.2.11 | 255.255.255.0 | 192.168.2.1 |

---

## 3. PC configuration

On each PC: **Desktop → IP Configuration**, then enter the IP address, mask and default gateway from the table above (Static, not DHCP, for this lab).

---

## 4. Router configuration (CLI)

Click Router0 and open the **CLI** tab:

```
enable
configure terminal

interface GigabitEthernet0/0
 ip address 192.168.1.1 255.255.255.0
 no shutdown
 exit

interface GigabitEthernet0/1
 ip address 192.168.2.1 255.255.255.0
 no shutdown
 exit

end
write memory
```

`no shutdown` is the step almost every beginner forgets. Without it the interface stays administratively down, even if everything else is correct.

---

## 5. Verification

On the router:
```
show ip interface brief
```
Both interfaces must show **up / up**.

From PC0 (command prompt):
```
ping 192.168.1.1      → must succeed (local gateway)
ping 192.168.2.1      → must succeed (router, other interface)
ping 192.168.2.10     → must succeed (PC2, inter-network routing)
```

If the last ping fails while the first two succeed, check that both router interfaces are up/up and that PC2 and PC3 have the correct default gateway. This is the most common mistake at this stage.

---

## 6. Key takeaways

- A switch only forwards frames inside one network (one IP subnet).
- For a PC in 192.168.1.0/24 to talk to a PC in 192.168.2.0/24, a router is required: this is **inter-network routing**.
- The default gateway is the address a PC contacts whenever it needs to leave its own subnet.

---

## 7. Next lab

[Lab 2](../02-dhcp): replace the static addressing with **DHCP** served by the router.

---

## Screenshots

<!-- Add your screenshots to the images/ folder, then uncomment the lines below:
![Topology](images/topology.png)
![Verification](images/verification.png)
-->
