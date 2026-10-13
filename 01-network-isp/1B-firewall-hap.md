# Etapa 1B: firewall del hAP (cadenas input y forward)

Fecha: 2026-10-12 · Equipo: hAP ax lite LTE6 (RouterOS 7.23.7)

## Objetivo
Pasar de un router sin ninguna regla de firewall a uno que solo acepta administración desde las redes de gestión y bloquea todo lo que llega sin haber sido pedido desde la WAN.

## Antes
- Filter Rules vacía: el router aceptaba todo.
- NAT: solo el masquerade de salida por la interfaz WAN.

## Diseño
- La WAN es la interfaz que recibe IP por DHCP del router de arriba.
- Lista de direcciones `gestion`: 192.168.22.0/24 (Wi-Fi de gestión) y 192.168.88.0/24 (LAN del L009 y servidor).
- Orden de las reglas: primero las que aceptan, al final las que descartan.
- Solo se bloquea lo que entra por la WAN, así DHCP, DNS y las redes internas no se ven afectados.

## Reglas
| # | Cadena | Acción | Condición |
| --- | --- | --- | --- |
| 0 | input | accept | established, related, untracked |
| 1 | input | drop | invalid |
| 2 | input | accept | origen en la lista `gestion` |
| 3 | input | drop | entra por la WAN |
| 4 | forward | accept | established, related, untracked |
| 5 | forward | drop | invalid |
| 6 | forward | drop | new, entra por la WAN |

## Cómo lo hice
Por WinBox, con respaldo previo y Safe Mode encendido. Probé una sesión SSH nueva, internet desde mi Mac y el ping al servidor antes de apagar Safe Mode. Después hice un segundo respaldo con la configuración final. Los respaldos están en un repositorio privado.

## Verificación
- Los contadores de las reglas subieron con tráfico real (la regla 3 ya descartaba tráfico de la red de arriba).
- Internet y el acceso al servidor siguen funcionando.

## Lo que aprendí
- `input` protege al router; `forward` protege a los equipos que están detrás.
- Estados de conexión: new (primer paquete de una conversación, venga de donde venga), established, related, untracked e invalid.
- Las reglas se leen de arriba hacia abajo: lo conocido se acepta primero, lo inválido se descarta, y el bloqueo general va al final.
- Una lista de direcciones se escribe una vez y se reutiliza en varias reglas.
- Si algún día abro un puerto hacia un servidor, no se borra la regla 6: se le agrega la excepción para el tráfico redirigido (dstnat).

## Pendiente
- Comprobar el firewall desde la red de arriba (escaneo antes y después).
- Firewall propio del L009 con la misma lista `gestion`.