## 11 Value cache

### Visión general

Para que el cálculo de expresiones de activación, elementos calculados y algunas macros sea mucho más rápido, el servidor Zabbix soporta una opción de caché de valores.

Esta caché en memoria puede utilizarse para acceder a datos históricos, en lugar de realizar llamadas SQL directas a la base de datos. Si los valores históricos no están presentes en la caché, los valores que faltan se solicitan a la base de datos y la caché se actualiza en consecuencia.

Para habilitar la funcionalidad de caché de valores, el archivo de [configuración](https://www.zabbix.com/documentation/6.0/en/manual/appendix/config/zabbix_server) del servidor Zabbix soporta un parámetro opcional **ValueCacheSize**.

Se admiten dos elementos internos para supervisar la caché de valores: **zabbix[vcache,buffer,<mode>]** y **zabbix[vcache,cache,<parameter>]**. Ver más detalles con [elementos internos](https://www.zabbix.com/documentation/6.0/en/manual/config/items/itemtypes/internal).
