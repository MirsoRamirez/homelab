# Runbook: PPPoE server on a MikroTik over a VLAN

Tested on a L009 (RouterOS 7.23.7 LT) with a Huawei ONU as the client, behind a GPON OLT. Done in WinBox. Use Safe Mode for anything that can cut your own session.

## Before you start

- Take an export and a binary backup, and keep them in the private repo only.
- Read the current state first: `/interface vlan print`, `/interface bridge vlan print`, `/ip firewall filter print`, `/ppp profile print`.
- Do not put the server on a physical port that also carries management traffic. Put it on a VLAN interface.

## Steps

1. **Pool.** IP > Pool: a range that does not overlap any existing network.
2. **Profile.** PPP > Profiles: local address (the router side), remote address = the pool, rate limit, DNS server.
3. **Secret.** PPP > Secrets: user, password (never in the repo), service `pppoe`, the profile above.
4. **VLAN interface.** Enable the VLAN interface on the bridge, and check the bridge VLAN table includes the bridge itself as a tagged member.
5. **Server.** PPP > PPPoE Servers: interface = the VLAN interface, default profile, one session per host. Allow only `chap` and `mschap2`.
6. **Firewall** (Safe Mode on):
   - Address list with the pool's range.
   - NAT: masquerade for that range, out through the interface that reaches the internet.
   - Input: accept DNS (udp and tcp 53) from the list, placed above the factory "drop all not coming from LAN".
   - Forward: drops from the list toward the lab, management and OLT networks.
   - Fasttrack: edit the factory rule to exclude the list (`!` in source and destination), or the rate limit is never applied.
7. **Client (ONU).** Change the WAN to PPPoE, same VLAN, enter the user and password. Check that the Wi-Fi binding is still ticked afterwards.

## Verify

- `/ppp active print`: the user is there with an address from the pool.
- `/log print where message~"pppoe"`: authenticated and connected.
- `/queue simple print stats`: the dynamic queue counts the traffic and drops packets during a speedtest. If it counts almost nothing, fasttrack is skipping it.
- Speedtest is close to the plan rate.
- From the client side, SSH to the router times out (not refused) and the counter of the factory drop goes up.

## Troubleshooting

| Symptom | Check |
| --- | --- |
| Session never appears | Log shows nothing: problem before the router (ONU, OLT, VLAN). Log shows a failed login: user or password |
| Server flagged invalid | The interface has no link |
| Speed above the plan | Fasttrack skipping the queue |
| Client reaches the lab | Forward drops missing or below fasttrack |

## Rollback

On the ONU: IPoE and DHCP again, Apply. On the router: restore the pre-change backup, or disable the server.