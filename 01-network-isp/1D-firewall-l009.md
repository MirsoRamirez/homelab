# 1D: IPv4 input firewall on the L009 core router

Date: 2026-10-18 · RouterOS 7.23.7 (long-term) · Configured with WinBox, Safe Mode on

## Goal

Add a second layer of protection on the core router (L009), using the same
pattern as the edge router (see `1B-firewall-hap.md`): only the management
networks can talk to the router itself. If something ever gets past the edge
router, the core router still protects its own services.

## Starting point (read-only audit)

Before changing anything I read the current state:

- Firewall: untouched factory defaults (defconf). Input chain: accept
  established/related/untracked, drop invalid, accept ICMP, accept loopback,
  drop everything not coming from the LAN list. Forward chain: IPsec, fasttrack,
  accept established/related, drop invalid, drop new from WAN without dstnat.
- NAT: one masquerade rule on the WAN list.
- Address lists: none.
- Interface lists: LAN = bridge, WAN = ether1 (unused).
- DHCP server and DNS (allow-remote-requests = yes) both serve the LAN.

Pre-change backup: `pre-1D-L009` (private repo only).

## What I changed

Address list `gestion` (same as on the edge router):

| List | Address |
| --- | --- |
| gestion | 192.168.22.0/24 (management Wi-Fi) |
| gestion | 192.168.88.0/24 (core LAN) |

Three new input rules, added at the end of the chain (the defconf rules were
not modified):

| # | Chain | Match | Action | Comment |
| --- | --- | --- | --- | --- |
| 12 | input | UDP dst-port 67, in-interface-list LAN | accept | 1D: accept DHCP |
| 13 | input | src-address-list gestion | accept | 1D: accept gestion |
| 14 | input | (anything else) | drop | 1D: drop rest |

Why the DHCP exception: a device asking for an address has no IP yet, so its
request leaves with source 0.0.0.0, which is not in `gestion`. Without rule 12,
rule 14 would drop it and new devices would never get an address.

Why the order matters: RouterOS reads rules top to bottom and the first match
wins, so the drop rule must be last.

## Verification

- All three rules and both list entries confirmed with `print` over SSH after
  turning Safe Mode off (changes persisted).
- Rule 12 counter increased (DHCP passing through the exception).
- Rule 13 counter increased (my own WinBox session, source in `gestion`).
- SSH and WinBox kept working after the change.
- Post-change backup: `post-1D-L009` (private repo only).

## Findings and limits (honest notes)

- **Rule 14 has not been tested with real blocked traffic.** Its counter is 0,
  which only shows that nothing outside `gestion` has reached the router yet.
  A real test needs a TCP connection from a network not in `gestion` (e.g. the
  other networks behind the edge router). Planned for when clients exist there.
- **ICMP still passes.** The factory rule "accept ICMP" sits before my rules, so
  ping from any network is still accepted. This follows from the rule order; I
  did not measure it. It is normal defconf behavior and I left it as is.
- `gestion` includes the whole core LAN, so LAN devices can still reach the
  router. The new rules mainly restrict the other networks and the WAN.
- When PPPoE clients are added (Stage 2), they will need their own rules.

## Lessons

- Read the existing firewall before writing rules.
- Services that answer the LAN (DHCP, DNS) can break if a "drop the rest" rule
  is added without thinking about them.
- Safe Mode protects against locking yourself out, but changes only persist
  once it is turned off.

## Resumen en español

Antes de tocar nada leí el firewall del L009: era el de fábrica, sin lista de
direcciones. Creé la lista `gestion` y añadí 3 reglas al final de la cadena
input: aceptar DHCP (excepción necesaria porque un equipo sin IP sale con
origen 0.0.0.0), aceptar `gestion` y descartar el resto. Verifiqué por SSH y con
los contadores de las reglas 12 y 13. Límites: la regla 14 aún no se probó con
tráfico bloqueado real, y el ping sigue pasando por una regla de fábrica
anterior (deducido por el orden, no medido).