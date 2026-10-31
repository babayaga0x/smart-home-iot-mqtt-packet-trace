# VLAN y direccionamiento

El ruteo inter-VLAN se realiza con **router-on-a-stick** en R1 (`GigabitEthernet0/1` con subinterfaces 802.1Q).

| VLAN | Nombre | Subred | Gateway (R1) | Direccionamiento | Función |
|---|---|---|---|---|---|
| 10 | PCs | 192.168.10.0/24 | 192.168.10.1 (Gi0/1.10) | DHCP (.1–.20 excluidas) | Equipos administrativos y estaciones de trabajo |
| 20 | WIFI | 192.168.20.0/24 | 192.168.20.1 (Gi0/1.20) | DHCP (.1–.20 excluidas) | Dispositivos cliente conectados mediante puntos de acceso inalámbricos |
| 30 | IOT | 192.168.30.0/24 | 192.168.30.1 (Gi0/1.30) | DHCP (.1–.20 excluidas) | Dispositivos IoT y equipos de adquisición de datos |
| 40 | – | 192.168.40.0/24 | 192.168.40.1 (Gi0/1.40) | Estático (sin pool DHCP) | Servicios de infraestructura, incluido el broker MQTT (192.168.40.10) |

Los pools DHCP `VLAN10`, `VLAN20` y `VLAN30` están en R1 y entregan el gateway y el DNS `8.8.8.8`.

## Enlace entre routers

| Dispositivo | Interfaz | IP |
|---|---|---|
| R1 | Gi0/0 | 10.0.0.1/30 |
| R2 | Gi0/0 | 10.0.0.2/30 |

## Ruteo (estático)

- **R1:** ruta por defecto `0.0.0.0/0` vía `10.0.0.2` (R2).
- **R2:** una ruta estática por cada subred de la LAN (`192.168.10/20/30/40.0/24`) vía `10.0.0.1` (R1).

## Equipos principales

| Dispositivo | IP | VLAN |
|---|---|---|
| Broker MQTT (Server-PT) | 192.168.40.10 | 40 |
| SBC2 (publicador, Wi-Fi vía AP3-EXTERIOR) | DHCP | 20 |
| Laptop1 – Sala Técnica (suscriptor) | 192.168.10.23 (DHCP) | 10 |

El publicador (VLAN 20), el suscriptor (VLAN 10) y el broker (VLAN 40) están en VLAN distintas: R1 rutea el tráfico MQTT entre ellas (router-on-a-stick).

Configuraciones: [`configs/R1.txt`](../../configs/R1.txt), [`configs/R2.txt`](../../configs/R2.txt).

## Justificación de la segmentación VLAN

La placa SBC2 se conecta de forma inalámbrica mediante AP3-EXTERIOR y obtiene su dirección del pool DHCP de la VLAN 20. Esto permite que el dispositivo se comunique con los equipos de su segmento de red.

La SBC2 actúa como intermediaria con una conexión de tipo cliente (la cámara está conectada a su puerto IoT), por eso se ubica en la VLAN de clientes, mientras que la VLAN 30 se destina a los demás dispositivos IoT y la VLAN 40 aloja los servicios de infraestructura, incluido el broker MQTT.
