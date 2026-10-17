# Stage 1C: remote access to the lab (Tailscale subnet router)

Date: 2026-10-17 · Devices: labserver (Ubuntu Server), hAP and L009 (RouterOS 7.23.7)

## Goal
Manage the whole lab (SSH to the server, WinBox/SSH to both MikroTik routers) from anywhere, without opening any port to the internet.

## The problem
The ISP's ONU does PPPoE and NAT, and the Reyee router and the hAP add two more NAT layers. I compared the WAN address shown by the ONU with the public address the internet sees: they are different, so the connection is behind CGNAT. Port forwarding is not possible, so a classic WireGuard server at home would not be reachable.

## Design
An outbound-only tunnel. The labserver runs Tailscale (WireGuard-based) and works as a subnet router for the two management networks, 192.168.88.0/24 and 192.168.22.0/24. My laptop joins the same private network (tailnet) and reaches those networks through the labserver.

Why it works with the existing firewall: subnet-routed traffic leaves the labserver with its own LAN address (192.168.88.252), which is already in the `gestion` address list on the hAP and in the ssh/winbox "Available From" ranges. No firewall rule had to change.

## What I did
1. Created the account (Google login) and enabled two-step verification there.
2. Installed Tailscale on the labserver and logged it in.
3. Enabled IPv4 forwarding on the server (`net.ipv4.ip_forward = 1`).
4. Advertised the two routes with `tailscale up --advertise-routes=...`.
5. Approved the routes in the admin console and disabled key expiry for the server, so it does not drop off the network after a few months.
6. Installed Tailscale on my Mac and logged in with the same account.

## Verification
From my Mac on mobile data (outside my home network), with Tailscale on: SSH to the labserver worked, and so did a connection to both MikroTik routers. No inbound port is open on any device.

## What I learned
- CGNAT can be detected by comparing the WAN address of the first router with the public address seen from the internet.
- With CGNAT you need a tunnel that starts from inside, not a port forward.
- A subnet router lets devices that cannot run the client (like routers) be reached through the tunnel.
- The tailnet account is now the key to the lab, so it needs two-step verification.

## Next
- Stage 1D: firewall for the L009 using the same `gestion` list.
- Check the IPv6 firewall on both routers.

## Resumen en español
El lab está detrás de CGNAT, así que no se pueden abrir puertos. Usé Tailscale en el labserver como "puerta" hacia las redes de gestión, sin abrir nada hacia internet. Probé desde mi Mac con datos móviles: SSH al servidor y conexión a los dos MikroTik funcionaron. Como las conexiones salen con la IP del servidor (ya permitida en `gestion`), no cambié el firewall.