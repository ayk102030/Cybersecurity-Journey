# 🌐 Lab 04: Understanding IPv4 Addressing — Public vs. Private IPs & NAT

Welcome to the fourth lab! In this module, we dive into the network layer to break down how devices are identified and routed across local networks and the global internet. We will analyze the core structural differences between **Public** and **Private** IP addresses, map the reserved private address ranges, and explore how **NAT (Network Address Translation)** bridges the two.

---

## 🌎 1. Public IP Address vs. Private IP Address

An IP address (IPv4) is a unique numerical identifier assigned to devices on a network. However, to conserve the limited number of available IPv4 addresses, the system is split into two distinct types:

### 🏠 A. Private IP Address
* **The Concept:** Used exclusively for communication *within* a local private network (like your home, office, or lab network).
* **Routing:** These addresses are **non-routable** on the global internet. Routers on the internet will instantly drop any packet originating from or destined directly to a private IP.
* **Uniqueness:** They only need to be unique within their specific local network. Different households or companies around the world can reuse the exact same private IP addresses simultaneously without any conflict.

### 🌐 B. Public IP Address
* **The Concept:** Used for communication across the global internet. It identifies your entire local network or public server to the outside world.
* **Routing:** Fully **routable** across global internet routers.
* **Uniqueness:** Must be globally unique. No two devices connected directly to the internet can share the same public IP address at any given time.

---

## 📊 2. Reserved Private IP Address Ranges (RFC 1918)

To ensure private addresses never leak into the public internet, specific blocks of the IPv4 address space were officially set aside for private networks across three primary classes:

| Class | Private IP Range | Subnet Mask / CIDR | Commonly Used In |
| :--- | :--- | :--- | :--- |
| **Class A** | `10.0.0.0` – `10.255.255.255` | `255.0.0.0` (`/8`) | Large enterprise networks and cloud environments. |
| **Class B** | `172.16.0.0` – `172.31.255.255` | `255.240.0.0` (`/12`) | Medium-sized corporate networks. |
| **Class C** | `192.168.0.0` – `192.168.255.255` | `255.255.0.0` (`/16`) | Home networks and small offices (e.g., `192.168.1.1`). |

> **⚠️ Security Note:** Any IP address falling *outside* of these defined ranges is considered a **Public IP** address (excluding other technical reservations like loopback `127.0.0.1` or link-local `169.254.x.x`).

---

## 🔄 3. Network Address Translation (NAT)

Since private IP addresses cannot communicate on the internet, how does your phone or laptop (which holds a private IP like `192.168.1.5`) load a website from a public internet server? This is achieved via **NAT (Network Address Translation)**.

* **The Role of the Router:** Your local router sits at the edge between your private network and the public internet. It is assigned **one public IP address** by your Internet Service Provider (ISP).
* **The Translation Process:** 1. When a local device sends a request to the internet, the router intercepts the packet.
  2. The router modifies the packet's headers, replacing the internal **Private IP** with its own **Public IP**.
  3. The router keeps track of this mapping in a internal dynamic table (**NAT Forwarding Table**).
  4. When the internet server replies, the router receives the data on its public IP, looks up the mapping table, changes the destination back to the device's **Private IP**, and forwards the packet inside the local network.

---

## 🏴‍☠️ 4. The Cybersecurity Angle: Network Reconnaissance

From a penetration testing perspective, understanding the boundary between public and private spaces is fundamental:

* **Internal Scouting:** When an attacker compromises a machine inside a network, running commands like `ipconfig` or `ifconfig` reveals the private IP footprint. This helps the attacker understand the internal subnet architecture and begin lateral movement/pivoting.
* **External Blinding:** External attackers scanning a company from the public internet can only see the edge public IP address or firewall. NAT acts as a basic shield because internal machines are hidden behind a single public gateway, preventing direct external connection attempts unless explicit port-forwarding is configured.
