# Lab 3: Controlling Traffic with ACLs (Packet Tracer)

[← Previous lab](../02-dhcp) · [Series index](../README.md)

| File | Content |
|---|---|
| `tp03.pkt` | Packet Tracer file for this lab |
| `configs/` | ACL configurations (standard and extended) |
| `images/` | Screenshots |

📝 LinkedIn article: [FR](#) · [EN](#)

**Level:** Beginner → intermediate
**Prerequisites:** Lab 1 (inter-network routing) and Lab 2 (DHCP)
**Objective:** Use the router's Access Control Lists (ACLs) to filter traffic between the two networks: first a standard ACL, then an extended ACL.
**Estimated time:** 60–75 minutes

---

## 1. Topology: the Lab 2 setup plus a server

```
                          [Server0]
                              |
        [PC0]                 |                  [PC2]
          |                   |                     |
        [Switch0] ------ [Router0] ------ [Switch1]
          |                                         |
        [PC1]                                    [PC3]
```

Add **a server** (Server-PT) connected to **Switch0** (network 1):

| Device | Network | IP address | Configuration |
|---|---|---|---|
| Server0 | 192.168.1.0/24 | **192.168.1.5** (static) | mask 255.255.255.0, gateway 192.168.1.1 |

> **Link with Lab 2:** 192.168.1.5 belongs to the range excluded from DHCP (.1 to .9). That is exactly the purpose of exclusions: reserving addresses for fixed-IP devices.

The server's **HTTP** service is enabled by default in Packet Tracer (tab *Services → HTTP*). Make sure it is set to "On".

**Before you start:** check that everything works (the 4 PCs get an IP through DHCP, PC0 can ping PC2 and the server). Never filter a network you have not tested first.

---

## 2. What is an ACL?

An ACL is a list of "permit / deny" rules that the router reads **from top to bottom** for each packet, on a given interface and in a given direction (`in` = inbound, `out` = outbound).

Three rules to remember:
1. **The first matching rule wins.** The router stops there, so order is crucial.
2. **Implicit deny at the end.** If no rule matches, the packet is dropped (an invisible `deny any` ends every ACL).
3. **An ACL does nothing until it is applied** to an interface with `ip access-group`.

| Type | Filters on | Where to place it |
|---|---|---|
| **Standard** (numbers 1–99) | the **source** IP address only | **close to the destination** |
| **Extended** (numbers 100–199, or named) | source, destination, protocol, port | **close to the source** |

**The wildcard mask:** in an ACL you do not write 255.255.255.0 but its inverse, 0.0.0.255. A `0` means "this bit must match", and a `255` means "don't care". So `192.168.2.0 0.0.0.255` = the whole 192.168.2.x network, and `host 192.168.2.11` = that machine only.

---

## 3. Part A: Standard ACL, block one host

**Scenario:** PC3 (192.168.2.11) must no longer reach network 1. The other machines in network 2 keep their access.

A standard ACL only looks at the source, so we place it **close to the destination**: on Gi0/0, in the **outbound** direction (toward network 1).

```
enable
configure terminal

access-list 10 deny host 192.168.2.11
access-list 10 permit any

interface GigabitEthernet0/0
 ip access-group 10 out
 exit

end
```

The `permit any` line is essential: without it, the implicit deny would block **everyone**.

### Verification
- From **PC3**: `ping 192.168.1.10` (or any PC in network 1) → must **fail**.
- From **PC2**: same ping → must **succeed**.
- On the router: `show access-lists` → each rule shows a packet counter (`matches`). This is how you check that a rule really works.
- `show ip interface GigabitEthernet0/0` → line "Outgoing access list is 10".

### Cleanup before Part B
```
configure terminal
interface GigabitEthernet0/0
 no ip access-group 10 out
 exit
no access-list 10
end
```

---

## 4. Part B: Extended ACL, allow only web traffic to the server

**Scenario:** hosts in network 2 (guests) may only reach the server's **web service (HTTP)** at 192.168.1.5. Everything else toward network 1 is forbidden.

We filter close to the source: on Gi0/1, in the **inbound** direction (traffic from network 2 arriving at the router).

```
configure terminal

ip access-list extended FILTRE_INVITES
 permit udp any eq 68 any eq 67
 permit tcp 192.168.2.0 0.0.0.255 host 192.168.1.5 eq 80
 deny ip any any
 exit

interface GigabitEthernet0/1
 ip access-group FILTRE_INVITES in
 exit

end
write memory
```

### Why the first line (`permit udp any eq 68 any eq 67`)?
It is the clients' **DHCP** request (source port 68, destination port 67). Without it, the ACL would block DHCP requests from network 2: a PC renewing its lease would no longer receive an address. This illustrates an important principle: **an ACL can break services you did not think about.**

### Verification
- From **PC2**, *Desktop → Web Browser*: `http://192.168.1.5` → the server's page **is displayed**.
- From **PC2**: `ping 192.168.1.5` → **fails** (ICMP is not allowed).
- From **PC2**: `ping 192.168.1.10` (PC0) → **fails**.
- `show access-lists` → the counters of `permit tcp` and `deny ip any any` increase as you test.

> Depending on the Packet Tracer version, a denied packet shows as "Request timed out" or "Destination host unreachable". Either way, the ACL is doing the blocking.

---

## 5. Classic pitfalls

1. **ACL created but not applied.** Without `ip access-group`, nothing happens.
2. **Wrong direction (`in` / `out`) or wrong interface.** Always ask yourself: "where does the packet come from, and where do I intercept it?"
3. **Rule order.** If a broad `permit` comes before a specific `deny`, the `deny` is never reached.
4. **The implicit deny.** An ACL containing only `deny` rules blocks everything. Remember the final `permit`.
5. **ACLs are stateless.** In Part B, PC0 (network 1) can no longer ping PC2: the request leaves fine, but PC2's **reply** comes back through Gi0/1 inbound, and the ACL rejects it. A classic ACL does not "remember" that a conversation was started from the other side. This is the limit that stateful firewalls (such as **FortiGate**, covered later in the series) solve.
6. **Editing a numbered ACL** is painful: you cannot easily insert a line in the middle. **Named** ACLs (like `FILTRE_INVITES`) are easier to maintain.

---

## 6. Key takeaways

- An ACL is a list of rules read top to bottom: **first match wins**, with an **implicit deny** at the end.
- Standard = filters on the source, placed close to the destination. Extended = fine-grained filtering (protocol, ports), placed close to the source.
- An ACL must be **applied** to an interface, in a specific direction.
- Always test **before** and **after**, and read `show access-lists`.

---

## 7. Next lab

**VLANs**: segmenting a network directly at the switch level, before the router.

---

## Screenshots

<!-- Add your screenshots to the images/ folder, then uncomment the lines below:
![Topology](images/topology.png)
![ACL counters](images/access-lists.png)
-->
