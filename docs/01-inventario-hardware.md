# Inventario investigado y viabilidad

## Contexto de la observacion

- Fecha y sistema utilizado: 13/10/26 sistema Linux (uso de Debian Live para utilizar comandos que requieren sudo)
- Modalidad, equipo anfitrion y VM VirtualBox si procede: Fisica, Proxmox se instalará directamente sobre el equipo (APD ALDA+, placa ASUS PRIME B250M-A).
- Version de Proxmox prevista: PENDIENTE. (La ultima version estable)

## Inventario inicial

| Area | Dato observado y evidencia | Especificacion y fuente | Limite o diferencia | Justificacion |
|---|---|---|---|---|
| CPU |Intel Pentium G4560 @ 3,50 GHz, x86_64, 2 núcleos / 4 hilos, VT-x |Intel ARK: 2 núcleos, 4 hilos, 3,5 GHz, 54 W, VT-x, VT-d y EPT (fuente 3) |Solo 4 hilos lógicos; el 2.º par de hilos es Hyper-Threading, no núcleos reales  |Cumple lo exigido por Proxmox (64 bits + VT-x), pero deja poco margen de CPU |
| RAM |2 × 4 GB DDR4 a 2400 MT/s (Crucial CT4G4DFS824A.C8FBD2); el SO ve ~7,6 GiB |Placa: 4 ranuras, máx. 64 GB, DDR4 2400/2133 no ECC (fuente 4). CPU: máx. 64 GB, 2 canales (fuente 3) |8 GB en total, 2 ranuras libres. La diferencia 8 GB → 7,6 GiB es de unidades y de memoria reservada |Es el recurso más escaso del proyecto; se puede ampliar con 2 módulos más |
| Almacenamiento |1 SSD SATA de 447,1 GB (lsblk, ROTA=0); sda1 en / con 65 GB usados de 440 GB |Placa: 6 puertos SATA 6 Gb/s y 1 M.2 (fuente 4). Proxmox: SSD recomendado, PLP recomendado (fuente 1) |Un único disco: sin RAID ni redundancia. SSD de consumo |  Capacidad suficiente; el riesgo es la falta de redundancia y de PLP |
| Red |1 NIC Realtek RTL8111/8168/8411 Gigabit (lspci); eth0 aparece **inactiva**  |Placa: Realtek RTL8111H 1 Gb/s (fuente 4). Proxmox exige una NIC (fuente 1) |Una sola NIC. Interfaces virbr0 (192.168.122.0/24), docker0 (172.17.0.0/16) y br-e2dbdff0d154 (172.19.0.0/16) pertenecen al Linux actual, no al hardware  |Basta para el laboratorio; hay que comprobar cable/DHCP y evitar solapar esas subredes |
| Plataforma |Placa ASUS PRIME B250M-A, BIOS AMI 0810 (09/08/2017); gráfica integrada Intel HD 610 |Placa Intel B250, socket LGA1151, Micro-ATX (fuente 4) |BIOS de 2017.UEFI |Modo de arranque UEFI y VT-x activo |

## 1. Puedo instalar Proxmox en este equipo?

Contrastar el inventario con requisitos oficiales de la version prevista.
Responder por separado y justificar con datos y fuentes:

| Componente o requisito | Dato de mi equipo y evidencia | Requisito consultado y fuente | Cumple, no cumple o pendiente |
|---|---|---|---|
| CPU, arquitectura y núcleos/hilos | x86_64, 2 núcleos / 4 hilos (lscpu) | CPU de 64 bits Intel 64 o AMD64 (fuente 1) | **Cumple** |
| Soporte de virtualización y configuración | lscpu indica VT-x; ARK confirma VT-x y EPT | Intel VT / AMD-V para KVM (fuente 1) | **Cumple** |
| RAM | 8 GB instalados, 7,6 GiB visibles | Mínimo de evaluación: 1 GB; recomendado: 2 GB para SO y servicios, más memoria para invitados (fuente 1) | **Cumple** el mínimo; el margen para invitados es limitado |
| Almacenamiento | SSD SATA de 447,1 GB | Un disco; recomendado SSD con PLP y redundancia (fuente 1) | **Cumple** el mínimo; **no cumple** la recomendación de redundancia/PLP (no necesaria en laboratorio) |
| Red | NIC Gigabit Realtek, eth0 inactiva ahora | Al menos 1 NIC; recomendadas NIC Gigabit redundantes (fuente 1) | **Cumple**; pendiente comprobar enlace y DHCP |
| Plataforma, firmware y arranque | BIOS AMI 0810; modo UEFI | Instalación desde USB (ISO híbrida) (fuente 1) | **Cumple** |

- Puedo instalar y arrancar Proxmox: Sí, con condiciones. Los datos medidos (CPU de 64 bits con VT-x, 7,6 GiB de RAM, SSD y NIC Gigabit) superan los requisitos mínimos de evaluación de Proxmox. Las condiciones son activar VT-x en la BIOS, confirmar el modo de arranque y **hacer copia de seguridad del Linux actual**: el instalador de Proxmox reparticiona el disco y borra lo que haya en él (65 GB usados, incluyendo Docker y libvirt).
- Puedo ejecutar el servidor ASO y el cliente del proyecto: Sí, con condiciones. Hay que distinguir lo que exige Proxmox (mínimo de evaluación) de lo que necesita mi laboratorio (hipervisor + `srv-aso` + `cli-linux` + una plantilla + copias). Con 7,6 GiB, la cuenta de la pregunta 2 deja unos 2,1 GiB de saldo, así que es viable si no se ejecuta todo a la vez con cargas pesadas, si se usa LXC donde sea posible y si no se usa ZFS (su caché ARC consume RAM). Con el cliente en modo gráfico completo y varias VM simultáneas el equipo quedaría justo.
- Condiciones o comprobaciones pendientes: .

## 2. Que limitaciones tendre con este hardware?

Lo medido es: 4 hilos de CPU, 7,6 GiB de RAM visibles, 447,1 GB de SSD y 1 NIC Gigabit.
Lo estimado son los consumos de cada invitado, que se comprobarán en las sesiones siguientes.

### Reparto de recursos

| Elemento | CPU/vCPU | RAM | Disco | Justificacion |
|---|---|---|---|---|
| Proxmox |4 hilos compartidos (sin reserva fija) |2,0 GiB |30 GiB |2 GB es la recomendación oficial para SO y servicios; 30 GiB para raíz y logs |
| srv-aso |2 vCPU |1,0 GiB |30 GiB |Servidor sin entorno gráfico; el servicio concreto (PENDIENTE) puede necesitar más disco |
| cli-linux |2 vCPU | 1,5 GiB | 25 GiB | Cliente ligero (LXC o VM mínima); no se espera entorno de escritorio completo |
| Plantilla, ISOs y copias | No aplica | No aplica |90 GiB (10 plantilla + 20 ISOs + 60 copias locales) | Plantilla parada, ISOs de Proxmox y Linux, y copias vzdump locales |
| Margen de operacion | Reserva en ráfagas | 1,0 GiB | 100 GiB (~22 %) | Evita agotar la RAM del anfitrión y llenar el disco (el almacenamiento thin se degrada al llenarse) |

Mostrar calculos de RAM y disco, reparto de vCPU, unidades y supuestos.
Explicar las cargas que pueden funcionar simultaneamente.

**Cálculo de RAM** (supuesto: 7,6 GiB utilizables, ZFS no utilizado, sin swap contada como RAM):
2,0 (Proxmox) + 1,0 (srv-aso) + 1,5 (cli-linux) + 1,0 (margen) = **5,5 GiB reservados**.
Saldo = 7,6 − 5,5 = **2,1 GiB libres**. La plantilla no consume RAM mientras está apagada.

**Cálculo de disco** (supuesto: 447,1 GiB según lsblk, que usa múltiplos de 1024):
30 + 30 + 25 + 90 + 100 = **275 GiB asignados**.
Saldo = 447,1 − 275 = **172,1 GiB sin asignar**. Es una reserva para crecimiento de los discos de las VM y para clonaciones; la capacidad virtual aprovisionada no es espacio físico garantizado si se usa aprovisionamiento ligero.

**CPU:** el equipo tiene 2 núcleos físicos y 4 hilos. Las vCPU no equivalen a núcleos exclusivos: 2 vCPU en `srv-aso` + 2 vCPU en `cli-linux` suman 4, igual que los hilos del anfitrión, y Proxmox necesita CPU para sí mismo. Con ambas cargas activas a la vez habrá competencia por la CPU.

**Cargas simultáneas previstas:** Proxmox + srv-aso + cli-linux a la vez, sin problema en reposo.


| Limitacion | Evidencia o estimacion | Consecuencia | Medida y limite que permanece |
|---|---|---|---|
| RAM total de 8 GB | free -h: ~7,6 GiB (medido) | Si se asigna toda la RAM a invitados, el anfitrión usa swap o se activa el OOM-killer | Reservar ≥ 1 GiB de margen y usar LXC. Resuelve el consumo; no resuelve la falta de RAM para muchas VM. Ampliable a 64 GB con las 2 ranuras libres |
| CPU de 2 núcleos / 4 hilos | lscpu (medido) | Clonar, hacer copias y usar el servicio a la vez ralentiza todo; sin sobreasignar vCPU | Limitar a 2 vCPU por invitado y evitar tareas simultáneas. No añade capacidad; solo reparte la existente |
| Un único SSD sin RAID | lsblk (medido) | Un fallo del disco pierde nodo, invitados y copias locales | Copias de las VM a un disco/USB externo. Reduce el riesgo, pero no da alta disponibilidad |
| SSD de consumo, sin PLP | La documentación de Proxmox recomienda PLP | Corte de luz durante escrituras puede dañar datos; el rendimiento puede caer en escrituras sostenidas | Apagado ordenado y, si es posible, SAI. No elimina el riesgo de corrupción |
| Copias y clonaciones en el mismo disco | Estimación: copias de 60 GiB reservadas | Compiten por E/S con las VM y comparten el mismo punto de fallo | Hacer las copias con las VM en reposo. No protege frente a pérdida del disco |
| Una sola NIC Gigabit | lspci y ip -br addr (medido); eth0 inactiva | Sin redundancia; gestión, invitados y copias comparten el mismo enlace | Puente vmbr0 único y segmentación por IP. No ofrece tolerancia a fallos|
| Interfaces y subredes del Linux actual | virbr0 192.168.122.0/24, docker0 172.17.0.0/16 | Posible que solape con la red de laboratorio | Elegir una subred distinta y tras instalar Proxmox desaparecerán esas interfaces |


## 3. Que componentes son mas criticos en mi servidor y por que?

Ordenar al menos tres componentes para este equipo y su servicio.
Distinguir cuello de botella, disponibilidad e integridad de los datos.

| Prioridad y componente | Funcion | Efecto si es insuficiente | Efecto si falla | Evidencia y justificacion |
|---|---|---|---|---|
| 1. RAM (8 GB) | Alberga el hipervisor y la memoria de cada VM/LXC | Swap, lentitud extrema y procesos terminados por falta de memoria; limita cuántos invitados caben | Cuelgues o reinicios de todo el nodo; no pierde datos persistentes por sí sola | Es el cuello de botella: tras reservar 5,5 GiB quedan 2,1 GiB. La unidad es barata de ampliar (2 ranuras libres) |
| 2. SSD SATA (447,1 GB, único) | Guarda Proxmox, discos de los invitados, plantilla, ISOs y copias locales | Sin espacio: las VM se bloquean y no caben copias ni clonaciones | Se pierde todo el laboratorio y las copias locales: mayor impacto en disponibilidad e integridad | Un solo disco, sin RAID, y con PLP sin confirmar. El fallo aquí es el más grave |
| 3. CPU (2 núcleos / 4 hilos) | Ejecuta el hipervisor y las vCPU | Latencia al clonar, copiar o atender al servicio a la vez | El equipo no arranca; no se pierden los datos del disco | lscpu y ARK: 4 hilos. Es suficiente si se limitan las tareas simultáneas |

Componente que mejoraria primero y motivo para priorizarlo frente a otro:
Ampliaría la RAM (por ejemplo, de 8 a 16 GB con 2 módulos DDR4-2400 más). Es lo que más limita el proyecto, es la mejora más barata, la placa y la CPU la admiten (hasta 64 GB) y permite más invitados sin riesgo para el anfitrión. La prioridad frente al SSD es por rendimiento: un segundo disco (o disco externo para copias) sería lo siguiente que mejoraria, ya que reduce el riesgo de pérdida de datos, pero no resuelve la falta de memoria.

## Fuentes tecnicas

Consultar al menos tres fuentes primarias, incluyendo Proxmox y
fabricante o plataforma virtual. Relacionarlas con las tres respuestas.

| Fuente y URL | Version/modelo y fecha de consulta | Afirmacion respaldada | Aplicacion al nodo |
|---|---|---|---|
|1. Proxmox VE, Installation (documentación oficial) — https://pve.proxmox.com/pve-docs/chapter-pve-installation.html | Proxmox VE; 15/10/2026 | El instalador particiona los discos (ext4, XFS, BTRFS, ZFS); instalación recomendada desde USB | El instalador borra el disco actual |
|2. Proxmox VE, System Requirements — https://pve.proxmox.com/wiki/System_Requirements y https://www.proxmox.com/en/proxmox-virtual-environment/requirements | Proxmox VE (versión de la ISO PENDIENTE); 15/10/2026 | CPU 64 bits con Intel VT/AMD-V; mínimo de evaluación 1 GB de RAM; recomendados 2 GB para SO y servicios más memoria de invitados; SSD, PLP y NIC Gigabit recomendados | Preguntas 1, 2 y 3: requisitos y recomendaciones |
|3. ASUS, PRIME B250M-A (ficha del fabricante) — https://www.asus.com (buscar «PRIME B250M-A») | PRIME B250M-A, BIOS 0810; 15/10/2026 | 4 DIMM DDR4 2400/2133 no ECC, máx. 64 GB; 6 × SATA 6 Gb/s; 1 × M.2; LAN Realtek RTL8111H Gigabit | Preguntas 1, 2 y 3. Valores contrastados en fichas de distribuidores|

## Dictamen de viabilidad

Apto, apto con condiciones o no apto para la propuesta actual: Apto con condiciones.
Sintetizar las tres respuestas y las condiciones pendientes de comprobar.

El equipo cumple los requisitos de instalación de Proxmox VE (64 bits, VT-x, 7,6 GiB de RAM, SSD y NIC Gigabit) y permite el proyecto con srv-aso y cli-linux si se reserva margen de RAM, se evita ZFS, se usa LXC cuando proceda y no se solapan tareas pesadas (clonar, copiar y probar el servicio a la vez). Condiciones pendientes: activar VT-x en la BIOS, confirmar el arranque UEFI/Legacy, copiar los datos del Linux actual antes de instalar, verificar red y modelo/salud del SSD, y contrastar las fichas de Intel y ASUS en las páginas de los fabricantes. La mejora prioritaria sería ampliar la RAM.

## Evidencias y revision docente

Enlazar salidas revisadas desde ../evidencias/. Un listado automatico no
sustituye la investigacion. Registrar la pregunta de defensa y respuesta.

[Volver al indice](../README.md)
