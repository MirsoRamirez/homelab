# Etapa 1A: cerrar servicios inseguros en los MikroTik

Fecha: 2026-10-11 · Equipos: hAP ax lite LTE6 y L009UiGS-RM (RouterOS 7.23.7)

## Objetivo
Dejar abiertas solo las puertas de administración necesarias y limitarlas a las redes de gestión.

## Antes
- telnet, ftp, www, api y api-ssl activos en ambos routers.
- ssh y winbox accesibles desde cualquier red (Available From vacío).

## Qué hice (por WinBox, con Safe Mode)
1. Respaldo previo de cada router (export + backup), guardado en un repo privado.
2. IP > Services: deshabilité telnet, ftp, www, api y api-ssl.
3. Limité ssh y winbox a 192.168.22.0/24 y 192.168.88.0/24.
4. Probé SSH con una sesión nueva y recién entonces apagué Safe Mode.
5. Segundo respaldo con la configuración final.

## Verificación
Escaneo de puertos desde mi Mac con `nc -z`: telnet (23), ftp (21) y www (80) cerrados; ssh (22) abierto, en ambos routers.

## Lo que aprendí
- Safe Mode deshace los cambios si pierdo la conexión; se apaga solo cuando ya comprobé que sigo entrando.
- Available From debe incluir la red desde la que administro, o me dejo fuera yo mismo.
- telnet y ftp envían contraseñas en texto plano; ssh las cifra.

## Pendiente
- Revisar el servicio reverse-proxy (443), aún activo.
- Etapa 1B: firewall del hAP.