# IP and VLAN Plan

## Current state (October 2026)

| Network           | Subnet          | Gateway               | Purpose                                   |
| ----------------- | --------------- | --------------------- | ----------------------------------------- |
| ISP ONU LAN       | 192.168.1.0/24  | ISP ONU               | Upstream of the home router               |
| Home LAN          | 192.168.26.0/24 | Reyee router (.1)     | Home network and uplink of the lab        |
| Management Wi-Fi  | 192.168.22.0/24 | hAP (.1)              | Wireless administration of the lab        |
| Lab LAN           | 192.168.88.0/24 | L009 (.1), hAP (.253) | Workstation, servers and L009-hAP link    |
| OLT management    | 172.16.10.0/24  | hAP (.1)              | VSOL OLT management, untagged             |
| Customer VLAN 100 | 192.168.20.0/24 | hAP (.1)              | Customer services, DHCP served by the hAP |
| PPPoE test pool   | 10.10.10.0/24   | L009 (10.10.10.1)     | Test clients on ether4 (pool-pppoe, profile plan-10M); temporary, replaced by Customer VLAN 100 over sfp1 |

Static hosts on the lab LAN:

| Host       | Address        | Notes                                                                                          |
| ---------- | -------------- | ---------------------------------------------------------------------------------------------- |
| labserver  | 192.168.88.252 | Ubuntu Server, Docker; DHCP static lease on L009, gateway 192.168.88.253 (hAP) until Phase 1   |

Unused test networks on the hAP, to be removed in Phase 1: 192.168.19.0/24 (VLAN 500), 172.100.100.0/24 (VLAN 20) and 192.168.14.0/24.

## Target design (end of Phase 1)

| Network           | Subnet          | Gateway   | Change                                             |
| ----------------- | --------------- | --------- | -------------------------------------------------- |
| Customer VLAN 100 | 192.168.20.0/24 | L009      | PPPoE server on the L009 replaces DHCP on the hAP  |
| OLT management    | 172.16.10.0/24  | L009      | Gateway moves to the L009; isolated from customers |
| Lab LAN           | 192.168.88.0/24 | L009 (.1) | Unchanged                                          |
| Management Wi-Fi  | 192.168.22.0/24 | hAP (.1)  | Unchanged; allowed to reach all lab networks       |
| IoT (reserved)    | 192.168.30.0/24 | L009      | Future VLAN for ESP32 sensors                      |

## Rules

- Customers can reach the Internet only; they cannot reach management or the lab LAN.
- Management is allowed only from the management Wi-Fi and the lab LAN.
- Only private addresses are documented here; no public IPs or credentials.