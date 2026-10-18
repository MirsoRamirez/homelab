# Stage 1B (addendum): IPv6 firewall on the hAP

Date: 2026-10-17 · Devices: hAP ax lite LTE6 and L009UiGS-RM (RouterOS 7.23.7)

## Goal
Check whether IPv6 was filtered on both routers, since the IPv4 firewall from stage 1B does not cover it.

## What I found
- IPv6 is enabled and forwarding on both routers.
- Neither router has a global IPv6 address: only link-local addresses. The ISP does not provide IPv6 (my line is behind CGNAT), so there is no path from the internet.
- L009: the factory IPv6 firewall is in place. It accepts established traffic, ICMPv6, DHCPv6 and IPsec, and its last rule in each chain drops everything that does not come from the LAN.
- hAP: the IPv6 firewall was empty, so it accepted everything. Any device on the network upstream could reach the router over IPv6 link-local with no filter.

## What I did
Built an 8-rule IPv6 firewall on the hAP with WinBox, modelled on the L009's factory design and following the same logic as the IPv4 rules:

| # | Chain | Action | Condition |
| --- | --- | --- | --- |
| 0 | input | accept | established, related, untracked |
| 1 | input | drop | invalid |
| 2 | input | accept | icmpv6 |
| 3 | input | drop | enters from the WAN |
| 4 | forward | accept | established, related, untracked |
| 5 | forward | drop | invalid |
| 6 | forward | accept | icmpv6 |
| 7 | forward | drop | enters from the WAN |

ICMPv6 is accepted before the drops on purpose: IPv6 needs it for Neighbor Discovery (the equivalent of ARP), so blocking it would break the network.

## Verification
- Read back the full rule list from the router and checked every chain, action and connection state.
- Within minutes, rule 3 had already dropped 5 packets (about 1 KB) that arrived from the upstream network. Before this change the router would have accepted them.
- My IPv4 access (SSH, WinBox) and internet kept working, as these rules only affect IPv6.
- Took a backup after the change and stored it in a private repository.

## What I learned
- An IPv4 firewall says nothing about IPv6: they are separate rule sets on MikroTik.
- Even without an IPv6 address from the ISP, every interface has a link-local address that devices on the same network can reach.
- Do not block ICMPv6 as a whole; it is part of how IPv6 works.
- Factory defaults can be a good reference design: the L009's was the model for the hAP.

## Next
- Firewall for the L009 (IPv4) using the same `gestion` list.
- Learn IPv6 properly: addressing, SLAAC, DHCPv6 and Prefix Delegation.

## Resumen en español
Revisé si el IPv6 estaba filtrado en los dos routers. El L009 ya traía firewall IPv6 de fábrica; el hAP lo tenía vacío, así que acepta todo. Creé 8 reglas siguiendo el mismo modelo que el IPv4 (y aceptando ICMPv6, necesario para que IPv6 funcione). Al poco tiempo la regla de la WAN ya había descartado paquetes de la red de arriba. Verifiqué la configuración, mi acceso por IPv4 sigue normal y guardé un respaldo.