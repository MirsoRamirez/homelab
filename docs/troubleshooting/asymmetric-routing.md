# # Server could not reach Docker Hub or accept SSH from Wi-Fi

*Resumen en español: el servidor tenía dos rutas de salida distintas para ida y vuelta del tráfico y el firewall del L009 bloqueaba las respuestas. Se arregló usando el hAP como gateway del servidor.*

## Problem

The Ubuntu server could ping the internet, but HTTPS to Docker Hub failed with "connection reset". SSH from a laptop on Wi-Fi also failed.

## Cause

Outgoing traffic went through the L009, but replies came back through the hAP. The L009 firewall dropped replies to connections it had never seen. This is called asymmetric routing.

## Fix

I set the hAP (192.168.88.253) as the server's default gateway with a netplan file, applied with `sudo netplan try`.

## Lesson

Two routers on the same network can split the path of a connection. A firewall that tracks connections will break it. In Phase 1 the L009 will become the single gateway.