# Bitacora de sesiones

La bitacora es el registro cronologico del trabajo. Anade una entrada al final
de cada sesion y conserva las anteriores. Escribe datos comprobables y enlaza
las evidencias utiles; evita frases generales como "he trabajado en Proxmox".

## Sesion 01 - 06/10/26

- Objetivo: iniciar GitHub y la documentacion Markdown.
- Acciones realizadas: PENDIENTE: configuraciones, cambios o investigacion.
- Decisiones y motivos: PENDIENTE: eleccion realizada y alternativas valoradas.
- Pruebas y resultados: PENDIENTE: resultado esperado, comando o metodo y resultado obtenido.
- Documentos o evidencias: [Ficha inicial](00-ficha-inicial.md).
- Incidencias y solucion o estado pendiente: PENDIENTE.
- Siguiente paso: PENDIENTE: accion concreta para la proxima sesion.

Anadir una entrada con esta estructura en cada sesion. El historial de
GitHub registra los commits; no hace falta incluir el hash del propio
commit dentro de su entrada de bitacora.

## Sesion 02 - 13/10/26

- Objetivo: Investigacion de los componentes fisicos y analisis de viabilidad.
- Acciones realizadas: inventario del equipo con el Linux actual (CPU, RAM, disco, red y placa); completada la ficha inicial; redactadas las tres preguntas de viabilidad en 01-inventario-hardware.md; contrastados los requisitos oficiales de Proxmox VE, la ficha de la CPU y la de la placa.
- Decisiones y motivos: modalidad fisica; evitar ZFS por el consumo de RAM de su cache; usar LXC cuando proceda; dictamen provisional "apto con condiciones".
- Pruebas y resultados: comprobar que se cumplan los requisitos minimos. Metodo: comandos del informe de hardware. Resultados: CPU de 64 bits con VT-x, ~7,6 GiB de RAM, SSD de 447,1 GB y NIC Gigabit; saldo calculado de ~2,1 GiB de RAM y ~172 GiB de disco tras el reparto previsto.
- Documentos o evidencias: [Ficha inicial](00-ficha-inicial.md).
- Incidencias y solucion o estado pendiente: eth0 aparece inactiva (comprobar cable y DHCP); modelo exacto y salud del SSD sin comprobar; fichas de Intel y ASUS pendientes de verificar en las webs de los fabricantes.
- Siguiente paso: Diseño de red.

Anadir una entrada con esta estructura en cada sesion. El historial de
GitHub registra los commits; no hace falta incluir el hash del propio
commit dentro de su entrada de bitacora.

[Volver al indice](../README.md)
