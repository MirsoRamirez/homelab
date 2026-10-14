# Stage 1A: closing insecure services on the MikroTik routers

Date: 2026-10-11 · Devices: hAP ax lite LTE6 and L009UiGS-RM (RouterOS 7.23.7)

## Goal
Leave open only the management ports that are needed, and limit them to the management networks.

## Before
- telnet, ftp, www, api, api-ssl and reverse-proxy enabled on both routers (www-ssl was already disabled).
- ssh and winbox reachable from any network (Available From empty).

## What I did (with WinBox, Safe Mode on)
1. Backup of each router first (export + backup), stored in a private repository.
2. IP > Services: disabled telnet, ftp, www, api, api-ssl and reverse-proxy.
3. Limited ssh and winbox to 192.168.22.0/24 and 192.168.88.0/24.
4. Tested a new SSH session and only then turned Safe Mode off.
5. Second backup with the final configuration.

## Verification
Port check from my Mac with `nc -z`, on both routers: telnet (23), ftp (21), www (80) and 443 closed; ssh (22) open.

## What I learned
- Safe Mode undoes my changes if I lose the connection; I turn it off only after I have confirmed I can still get in.
- Available From must include the network I manage from, or I lock myself out.
- telnet and ftp send passwords in clear text; ssh encrypts them.
- reverse-proxy and www-ssl both use port 443. I thought 443 was already closed because www-ssl was disabled, but reverse-proxy was still listening. Checking every row of the services table, and testing with `nc`, caught it.
- Available From does not hide a port: the connection is still accepted and then access is refused. Closing the service (or a firewall rule, in stage 1B) is what makes the port look closed or filtered.

## Next
- Stage 1B: hAP firewall (done, see `1B-firewall-hap.md`).

## Resumen en español
Cerré los servicios inseguros de los dos MikroTik (telnet, ftp, www, api, api-ssl y reverse-proxy) y limité ssh y winbox a las redes de gestión. Lo hice por WinBox con respaldo y Safe Mode, y lo verifiqué con `nc`: solo ssh queda abierto. Aprendí que reverse-proxy y www-ssl comparten el puerto 443 y que hay que revisar cada fila y probar, no suponer.