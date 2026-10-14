# Stage 1B: hAP firewall (input and forward chains)

Date: 2026-10-12/13 · Device: hAP ax lite LTE6 (RouterOS 7.23.7)

## Goal
Go from a router with no firewall rules to one that only accepts management traffic from my management networks and blocks everything that arrives unrequested from the WAN.

## Before
- Filter Rules was empty: the router accepted everything.
- NAT: only the outgoing masquerade on the WAN interface.

## Design
- The WAN is the interface that gets its IP by DHCP from the upstream router.
- Address list `gestion` (management): 192.168.22.0/24 (management Wi-Fi) and 192.168.88.0/24 (L009 LAN and server).
- Rule order: accept rules first, drop rules last.
- Only traffic entering from the WAN is blocked, so DHCP, DNS and the internal networks are not affected.

## Rules
| # | Chain | Action | Condition |
| --- | --- | --- | --- |
| 0 | input | accept | established, related, untracked |
| 1 | input | drop | invalid |
| 2 | input | accept | source in `gestion` list |
| 3 | input | drop | enters from the WAN |
| 4 | forward | accept | established, related, untracked |
| 5 | forward | drop | invalid |
| 6 | forward | drop | new, enters from the WAN |

## How I did it
With WinBox, after a backup and with Safe Mode on. Before turning Safe Mode off I tested a new SSH session, internet access from my Mac and a ping to the server. Then I took a second backup with the final configuration. Backups live in a private repository.

## Verification
- Rule counters went up with real traffic (rule 3 was already dropping traffic from the upstream network).
- Internet and server access keep working.

## What I learned
- `input` protects the router; `forward` protects the devices behind it.
- Connection states: new (first packet of a conversation, from either side), established, related, untracked and invalid.
- Rules are read top to bottom: known traffic is accepted first, invalid traffic is dropped, and the general block goes last.
- An address list is written once and reused in several rules.
- If I ever open a port towards a server, rule 6 is not deleted: I add an exception for redirected (dstnat) traffic.

## Audit from outside (2026-10-13)

I scanned the hAP's WAN address from another network (the Reyee Wi-Fi upstream) before and after enabling rule 3 (drop all input from the WAN). I used the same commands both times: `ping -c 3` and `nmap -Pn -p 22,23,80,443,2000,8291,8728,8729`.

| Test | Before (rule 3 off) | After (rule 3 on) |
| --- | --- | --- |
| Ping | 3 of 3 replies | 100% packet loss |
| 22 (ssh), 8291 (winbox) | open | filtered |
| 2000 (bandwidth test) | open | filtered |
| 23, 80, 443, 8728, 8729 | closed | filtered |

Findings:

- Before the rule, any device on that network could see the router and reach ssh and winbox.
- The bandwidth-test server (TCP 2000) was open and I had not planned it. I disabled it on both routers and confirmed with `nc` that it is now closed.
- "closed" means the router answers "nothing here". "filtered" means the firewall drops the packet silently, so the router reveals nothing.
- I had predicted everything would look filtered even before the rule. That was wrong; measuring is what showed the real difference.

## Next
- Firewall for the L009 using the same `gestion` list.

## Resumen en español
Pasé el hAP de no tener ninguna regla de firewall a tener 7: acepta administración solo desde mis redes de gestión y descarta lo que llega sin pedirlo por la WAN. Lo hice por WinBox con respaldo y Safe Mode, y lo verifiqué escaneando desde otra red: antes respondía ping y tenía abiertos ssh, winbox y el bandwidth test; después no responde a nada. El bandwidth test lo encontré abierto sin planearlo y lo cerré en los dos routers. Falta el firewall del L009.