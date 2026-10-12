# Zero to Expert — Networking & Linux Labs

[![LinkedIn](https://img.shields.io/badge/LinkedIn-Follow%20the%20series-0A66C2?style=flat-square)](https://www.linkedin.com/in/YOUR-LINKEDIN/)
![Status](https://img.shields.io/badge/status-in%20progress-brightgreen?style=flat-square)
![Labs](https://img.shields.io/badge/labs-3-blue?style=flat-square)
![Tools](https://img.shields.io/badge/tools-Packet%20Tracer%20%7C%20Cisco%20IOS-049FD9?style=flat-square)

A public, hands-on journey from absolute beginner to expert in **networking, telecoms and Linux system administration**, over 5 years. Every lab is documented here and explained in a LinkedIn article (published in French and English).

> **What comes after:** once the networking and Linux foundations are solid, the series continues with **Fiber Optics, DevOps, DevSecOps, Cybersecurity, and Web & Mobile Development**, using the same lab → article → repository format. See the [roadmap](#-roadmap).

---

## 📚 Labs

| # | Lab | Level | Tools 
|:---:|---|:---:|---|
| [01](./simple-lan) | First LAN: two subnets and a router | Beginner | Packet Tracer | 
| [02](./02-dhcp) | Automating addressing with DHCP | Beginner | Packet Tracer | 
| [03](./03-access-list) | Controlling traffic with ACLs | Beginner → Intermediate | Packet Tracer | 
| 04 | VLANs | Beginner → Intermediate | Packet Tracer | *coming soon* |

---

## 🗂️ Repository structure

```
.
├── README.md                  ← you are here
├── 01-reseau-lan-simple/
│   ├── README.md              ← lab guide (objective, topology, steps, pitfalls)
│   ├── configs/               ← device configurations (text)
│   ├── images/                ← screenshots
│   └── tp01.pkt               ← Packet Tracer file
├── 02-dhcp/
├── 03-acl/
└── linux/                     ← Linux labs (coming soon)
```

Each lab folder follows the same layout, so you can jump into any of them.

---

## ▶️ How to use a lab

1. Open the lab's `README.md` and follow the guide step by step.
2. Open the `.pkt` file with [Cisco Packet Tracer](https://www.netacad.com/) to see the finished topology, or rebuild it yourself from the guide (recommended).
3. Compare your configuration with the files in `configs/`.

---

## 🧭 Roadmap

| Year | Focus |
|:---:|---|
| **1** | Fundamentals: LAN, addressing, DHCP, ACL, VLAN, routing basics · Linux basics |
| **2** | Intermediate: dynamic routing, STP, VPN, GNS3 · MikroTik intro · Linux servers |
| **3** | Advanced: BGP, HA, IPsec · MikroTik · Fortinet intro · Docker, Ansible, monitoring |
| **4** | Expert: SD-WAN, VXLAN, automation · Fortinet advanced · Terraform, Kubernetes |
| **5** | Mastery: multi-vendor capstone, certifications |

**Beyond networking** (next chapters, same format): Fiber optics · DevOps · DevSecOps · Cybersecurity · Web development · Mobile development.

---

## 🤝 Feedback

Found a mistake, or have a better way to do something? Open an **issue** or a **pull request**. Corrections are welcome: this series is a learning journey, and I learn from every remark.

---

## 📬 Contact

[LinkedIn](https://www.linkedin.com/in/YOUR-LINKEDIN/) · [GitHub profile](https://github.com/YOUR-USERNAME)
