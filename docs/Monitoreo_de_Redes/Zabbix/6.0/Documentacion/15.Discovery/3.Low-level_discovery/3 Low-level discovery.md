# 3 Low-level discovery

# Visión general

El descubrimiento de bajo nivel proporciona una manera de crear automáticamente elementos, disparadores y gráficos para diferentes entidades en un equipo. Por ejemplo, Zabbix puede empezar a monitorizar automáticamente sistemas de archivos o interfaces de red en su máquina, sin necesidad de crear elementos para cada sistema de archivos o interfaz de red manualmente. Además, es posible configurar Zabbix para que elimine automáticamente las entidades innecesarias basándose en los resultados reales del descubrimiento realizado periódicamente.

Un usuario puede definir sus propios tipos de descubrimiento, siempre que sigan un protocolo JSON concreto.

La arquitectura general del proceso de descubrimiento es la siguiente.

En primer lugar, un usuario crea una regla de descubrimiento en la columna  "Configuration" → "Templates" → "Discovery". Una regla de descubrimiento consta de (1) un elemento que descubre las entidades necesarias (por ejemplo, sistemas de archivos o interfaces de red) y (2) prototipos de elementos, activadores y gráficos que deben crearse en función del valor de ese elemento.

Un ítem que descubre las entidades necesarias es como un ítem normal visto en otros sitios: el servidor pide a un agente Zabbix (o cualquiera que sea el tipo del ítem) un valor de ese ítem, el agente responde con un valor textual. La diferencia es que el valor con el que responde el agente debe contener una lista de entidades descubiertas en formato JSON. Aunque los detalles de este formato sólo son importantes para los implementaciones de comprobaciones de descubrimiento personalizadas, es necesario saber que el valor devuelto contiene una lista de pares macro → valor. Por ejemplo, el elemento "net.if.discovery" podría devolver dos pares: "{#IFNAME}" → "lo" y "{#IFNAME}" → "eth0".

Estas macros se utilizan en nombres, claves y otros campos prototipo donde luego se sustituyen por los valores recibidos para crear elementos reales, disparadores, gráficos o incluso hosts para cada entidad descubierta. Consulte la lista completa de [opciones](https://https://www.zabbix.com/documentation/6.0/en/manual/config/macros/lld_macros) para utilizar las macros LLD.

Cuando el servidor recibe un valor para un ítem de descubrimiento, mira los pares macro → valor y para cada par genera ítems reales, disparadores y gráficos, basados en sus prototipos. En el ejemplo anterior con "net.if.discovery", el servidor generaría un conjunto de elementos, disparadores y gráficos para la interfaz loopback "lo", y otro conjunto para la interfaz "eth0".

Tenga en cuenta que desde **Zabbix 4.2**, el formato del JSON devuelto por las reglas de descubrimiento de bajo nivel ha cambiado. Ya no se espera que el JSON contenga el objeto "data". El descubrimiento de bajo nivel ahora aceptará un JSON normal que contenga un array, con el fin de soportar nuevas características como el preprocesamiento de valores de elementos y rutas personalizadas a valores de macros de descubrimiento de bajo nivel en un documento JSON.

Las claves de descubrimiento incorporadas se han actualizado para devolver un array de filas LLD en la raíz del documento JSON. Zabbix extraerá automáticamente una macro y un valor si un campo de array utiliza la sintaxis {#MACRO} como clave. Cualquier nueva comprobación de descubrimiento nativa utilizará la nueva sintaxis sin los elementos "data". Al procesar un valor de descubrimiento de bajo nivel primero se localiza la raíz (array en $. o $.data).

Aunque se ha eliminado el elemento "data" de todos los elementos nativos relacionados con el descubrimiento, por compatibilidad con versiones anteriores Zabbix seguirá aceptando la notación JSON con un elemento "data", aunque se desaconseja su uso. Si el JSON contiene un objeto con un único elemento de matriz "data", extraerá automáticamente el contenido del elemento utilizando JSONPath $.data. El descubrimiento de bajo nivel acepta ahora macros LLD opcionales definidas por el usuario con una ruta personalizada especificada en sintaxis JSONPath.

**ADVERTENCIA**:

```
Como resultado de los cambios anteriores, los agentes más nuevos ya no podrán trabajar con un servidor Zabbix antiguo.
```
Véase también: [Entidades descubiertas (Discovered entities)](https://https://www.zabbix.com/documentation/6.0/en/manual/discovery/low_level_discovery#discovered-entities)

##  Configuración low-level discovery