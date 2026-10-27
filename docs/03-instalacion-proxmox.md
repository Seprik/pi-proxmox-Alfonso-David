# 03-instalacion-proxmox

## Objetivo

El objetivo de esta fase es instalar Proxmox VE dentro de una VM de VirtualBox, configurando la RAM, CPU, adaptadores RED correspondientes siguiendo el diagrama de red creado en la fase anterior y el almacenamiento .vdi.
Para finalmente acceder a la web proxmox por la ip del nodo de proxmox desde el buscador.

## Procedimiento

1-Instalar la iso de proxmox 9.2.2 y montarla en VirtualBox con sistema operativo Linux.

2-Configurar los recursos de la VM (CPU, RAM, ALMACENAMIENTO) y sus adaptadores de red correspondientes (NAT y adaptador solo anfitrion).

3-Dentro de la VM Proxmox, configurar la red del laboratorio: Desde la consola de la VM se edita /etc/network/interfaces para añadir vmbr1 sobre nic0 con 192.168.56.10/24 y se aplica con ifreload -a, vmbr1 hace de gateway de los invitados, con reenvío de IP y NAT hacia vmbr0, tal como indica la tabla.

4-Acceder al panel web de Proxmox mediante la ip configurada: Desde el navegador del equipo anfitrión: https://192.168.56.10:8006 (usuario root, reino «Linux PAM»)

## Comandos usados

```bash
ip -br link
ip-br addr
ip route
pveversion
df -h

nano /etc/network/interfaces
ifreload -a

nano /etc/hosts
hostname -f

reboot
```

## Evidencias

**Recursos y Red:**

![Procesador](/evidencias/capturas-o-salidas/s03-Procesador.png)

![Almacenamiento](/evidencias/capturas-o-salidas/s03-Almacenamiento.png)

![RAM](/evidencias/capturas-o-salidas/s03-RAM.png)

![Adaptadores](/evidencias/capturas-o-salidas/s03-Adaptadores.png)

![Interfaces](/evidencias/capturas-o-salidas/s03-interfaces.png)

**Salida de: hostnamectl, ip -br addr, pveversion, df -h**

![Panel proxmox](/evidencias/capturas-o-salidas/s03-hostnamectl.png)

![ip -br addr](/evidencias/capturas-o-salidas/s03-ip_addr.png)

![pveversion](/evidencias/capturas-o-salidas/s03-pveversion.png)

![df -h](/evidencias/capturas-o-salidas/s03-df.png)

**Panel de Proxmox:**

![Panel proxmox](/evidencias/capturas-o-salidas/s03-Panel_proxmox.png)


## Incidencias

**Nulas**

## Conclusion

Ya esta montado el Proxmox en una VM con sus recursos establecidos y la red configurada, esta comprobado que funcione el panel web de Proxmox. Ahora solo falta que tenga capacidad de salida a internet, que se hara en el siguiente paso.

[Volver al indice](../README.md)
