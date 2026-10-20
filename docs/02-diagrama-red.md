# 02-diagrama-red

## Objetivo
Diseñar la red del laboratorio antes de instalar servicios,definiendo: el nombre del nodo, la interfaz de gestión, los bridges, la red de laboratorio, las IPs de las VMs/LXC, el gateway, DNS, puertos y medidas de aislamiento. 

## Comandos usados

```bash

```

## Tabla de estructura de red

| Elemento | Nombre | IP | Funcion | Recursos |
|---|---|---|---|---|
| Nombre del nodo Proxmox | pve-proxmox | 192.168.56.10/24 | Hipervisor | Maquina virtual |
| Servidor ASO | srv-aso | 192.168.56.13/24 | Servicio principal | Maquina Virtual |
| Cliente | cli-linux| 192.168.56.23/24 | Pruebas | Maquina Virtual |
| Interfaz de gestión | enp0s3 | 192.168.56.1/24 | Gestion | Interfaz de Maquina Virtual |
| Bridge de Proxmox | vmbr0 | 10.0.2.15/24 | Red exterior del nodo, gateway 10.0.2.2 | Interfaz bridge de Proxmox |
| Bridge de Proxmox | vmbr1 | 192.168.56.10/24 | Administración y gateway de los invitados | Interfaz bridge de Proxmox |
| Red o VLAN de laboratorio | Laboratorio | 192.168.56.0/24, Sin VLAN | Red interna de pruebas entre nodo, srv-aso y cli-linux | Red de Virtual Box |
| Gateway y DNS | Gateaway y DNS | Gateaway: 192.168.56.10; nodo: gateaway 10.0.2.2; DNSNATVirtualbox: 10.0.2.3; DNS: Google 8.8.8.8 | Salida a internet | Servidor DNS |
| Puertos necesarios | 8006/tcp , 22/tcp | - | Web de Proxmox, conexion SSH | Proxmox y SSH  |

## Evidencias

## Incidencias
Nulas
## Conclusion

Finalmente, tras completar la tabla, asi es como deberia de ser la estructura de red que seguira mi proyecto intermodular, en principio, sin errores y funcional. Ahora solo queda aplicarlo cuando cree el Proxmox.

[Volver al indice](../README.md)
