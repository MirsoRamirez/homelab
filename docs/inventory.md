# Lab Inventory

| Device | Model | Role |
| --- | --- | --- |
| Core router | MikroTik L009UiGS-RM | ISP core: VLANs, PPPoE, routing |
| Edge router | MikroTik hAP ax lite LTE6 | Internet uplink, NAT, management Wi-Fi |
| Spare router | MikroTik hAP ax lite | Reserved for future use |
| OLT | VSOL V1600GS-F/ZF | GPON access, 1 PON port |
| ONU | Huawei EG8147X6-10 (x2) | PPPoE client router |
| Optical network | NAP box, 1:2 and 1:16 splitters, patch cords, SFP | Distribution network |
| Optical amplifier | EDFA | Power and overlay tests |
| Home router | Ruijie Reyee RG-EG105G | Home network (not part of the lab) |
| Microcontrollers | Arduino UNO, ESP32, ESP32-C5 | Sensors and telemetry |
| Server | i3 laptop, 4 GB RAM, 1 TB HDD | Linux, monitoring |
| Workstation | MacBook Pro M2 Pro | Administration and automation |

## Server laptop

| Item | Detail |
| --- | --- |
| Model | HP Pavilion x360 |
| CPU | Intel Core i3, 8th generation |
| RAM | 4 GB |
| Storage | 1 TB HDD (SMART health: Healthy, 3191 power-on hours, 0 read/write errors) |
| Planned OS | Ubuntu Server 24.04 LTS |
| Role | Always-on lab server: Docker, monitoring, NetBox |
| Network | Wi-Fi; wired link via USB Ethernet adapter (no built-in RJ45, to be confirmed) |
| Known limits | HDD is slow; SSD and 8 GB RAM upgrade planned |

*Resumen en español: laptop que será el servidor 24/7 del laboratorio; disco en buen estado, con mejora a SSD planeada.*