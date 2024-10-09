# 5 Novedades de Zabbix 7.0.0

Consulte los [cambios de última hora](https://www.zabbix.com/documentation/devel/en/manual/installation/upgrade_notes_700#breaking-changes) de esta versión.

## Pollers Asíncronos
Se han añadido nuevos procesos poller capaces de ejecutar varias comprobaciones al mismo tiempo:
* agent poller
* http agent poller
* snmp poller (for walk[OID] and get[OID] items)

Estos pollers son asíncronos - son capaces de iniciar nuevas comprobaciones sin necesidad de esperar respuesta, con concurrencia configurable hasta 1000 comprobaciones concurrentes.

Los pollers asíncronos se han desarrollado porque, en comparación, los procesos poller síncronos sólo pueden ejecutar una comprobación al mismo tiempo y la mayor parte de su tiempo se dedica a esperar la respuesta. Así pues, la eficiencia podría incrementarse iniciando nuevas comprobaciones paralelas mientras se espera la respuesta de la red, y los nuevos pollers así lo hacen.

