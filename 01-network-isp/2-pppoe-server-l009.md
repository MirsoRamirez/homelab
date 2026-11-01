# Stage 2: PPPoE server on the L009, over sfp1

2026-10-26 · RouterOS 7.23.7 LT · WinBox, Safe Mode on for the firewall changes

The goal was to make the L009 the PPPoE server for the GPON side, with the Huawei ONU as the client, so I get per-user login, speed plans and a place to cut customers later (the base for the ISP program).

## Design decision

The OLT hangs off sfp1, and sfp1 already carries the OLT management untagged plus VLAN 100 for customer traffic, all inside the `LAB_Switch` bridge (VLAN filtering on, no IP of its own). If I put the PPPoE server on the physical port, I would cut the OLT management. So the server listens on the `vlan100` interface instead, and the untagged traffic is untouched.

## What I built

- Pool `pool-pppoe`: 10.10.10.2-10.10.10.254 (temporary test range, the IP plan says 192.168.20.0/24 in the end).
- Profile `plan-10M`: local address 10.10.10.1, rate limit 10M/10M, DNS 10.10.10.1.
- One test secret on that profile.
- Server `isp-gpon` on `vlan100`, one session per host, CHAP and MS-CHAPv2 only (no PAP).
- A first server on ether4 (`isp-lab`) as a cable-free test spot. It stays flagged invalid because ether4 has no link.
- ONU: WAN changed from IPoE/DHCP to PPPoE, same VLAN 100, route mode. The form unticked SSID1 when I switched, so I had to tick it again. The old DHCP WAN was deleted.

The OLT needed no changes: it already passed VLAN 100 for the ONU's DHCP, and the PPPoE session came up on the first try once the credentials were right.

## Firewall for the clients

Address list `pppoe-clients` (10.10.10.0/24), then:

- NAT: masquerade for that list's range, out through `bridge`, since the L009 reaches the internet through the hAP.
- Input: accept DNS (udp and tcp 53) from the list. Order matters here: they sit above the factory "drop all not coming from LAN", because PPPoE sessions are dynamic interfaces that are not in the LAN list.
- Forward: three drops from the list toward the lab LAN, the management Wi-Fi and the OLT network, above fasttrack.

## The fasttrack problem

The speedtest gave about 30 Mbps on a 10M plan. The queue that RouterOS creates for the session counted almost nothing (about 0.2 MB), while the fasttrack dummy rule grew by about 64 MB in the same period. Fasttracked packets skip queues, so the limit was never applied.

Fix: I edited the factory fasttrack rule to exclude the list in both directions (`src-address-list=!pppoe-clients`, `dst-address-list=!pppoe-clients`). The rest of the lab keeps fasttrack. After that the queue counted about 26 MB each way and started dropping packets, which is the limit working. I did not write down the new speedtest number.

## Checks

- `/ppp active` shows the session with an address from the pool; the log shows authenticated and connected.
- Ping from the ONU's Wi-Fi to the lab server answered, but that device had Tailscale on, so it went through the tunnel. The "no lab LAN" drop counter did go up (140 to 147 packets) on traffic coming from the client.
- SSH from the client to 10.10.10.1 hangs (timeout, not refused), and the counter of "drop all not coming from LAN" went up by 12 packets during the test. So the router itself is closed to the clients.

Backups in the private repo: `pre-2sfp-L009`, `post-2fw-L009`, `post-etapa2-L009`.

## What I haven't proven

- `1D: drop rest` still has a counter of 0. The factory drop catches non-LAN traffic first, so this rule only sees LAN traffic outside `gestion`.
- The two DNS accept rules also sit at 0. The ONU is not asking the L009 for DNS. I guess it hands its own DNS to its clients, but I haven't looked.
- I only measured the drops toward three networks. Clients might still reach other networks behind the hAP (the plan says customers get the Internet only). That is a deduction from the routes, not a test.
- `isp-lab` still allows PAP and MS-CHAPv1.
- Moving to the final pool (192.168.20.0/24) means the hAP has to stop serving DHCP on VLAN 100.

## Resumen en español

Puse el servidor PPPoE del L009 sobre la interfaz `vlan100` (no sobre el puerto sfp1) para no cortar la gestión de la OLT. Creé pool, perfil de 10M, usuario de prueba y el servidor `isp-gpon` con CHAP/MS-CHAPv2; la ONU Huawei pasó de DHCP a PPPoE y conectó sin tocar la OLT. En el firewall: lista `pppoe-clients`, masquerade, DNS de entrada antes del drop de fábrica y tres drops de forward hacia la LAN, la gestión y la OLT. El speedtest daba 30 Mbps porque el fasttrack se saltaba la cola; excluí a los clientes PPPoE de esa regla y la cola ya cuenta y limita. Probé que el router no acepta SSH desde el cliente. Sin probar: `1D: drop rest`, el DNS (reglas en 0) y el acceso a otras redes detrás del hAP.