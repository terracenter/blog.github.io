---
title: Introducción
---

La API AbuseIPDB le permite utilizar nuestra base de datos mediante programación. Esto se hace más comúnmente a través de Fail2Ban, que viene preempaquetado con una configuración AbuseIPDB. Obtén una nueva clave API en el panel de control de tu cuenta.


Configuración de Fail2Ban
-------------------------

Siga nuestro [tutorial APIv1](../fail2ban.md). APIv2 funciona de la misma manera excepto por ligeras alteraciones en la solicitud cURL. En APIv2, `actionban` comando en abuseipdb.conf de Fail2Ban se verá como:

* El parámetro `category` se ha renombrado a `categories`.
* Su clave de API todavía se puede pasar en la consulta como clave, pero se recomienda utilizar el encabezado HTTP.
* El valor IP debe codificarse en url porque las direcciones IPv6 tienen dos puntos.
* Solicite JSON con "Accept: application/json".

Ya está. Ha migrado.

Punto final CHECK
-----------------

El punto `check` de comprobación acepta una única dirección IP (v4 o v6).Opcionalmente, puede establecer el parámetro `maxAgeInDays` para que solo devuelva informes de los últimos x días.

Los datos deseados se almacenan en la propiedad `data`. Aquí puede inspeccionar los detalles relativos a la dirección IP consultada, como la versión, el país de origen, el tipo de uso, el ISP y el nombre de dominio. Y, por supuesto, están los valiosos informes abusivos

La geolocalización, el tipo de uso, el ISP y el nombre de dominio proceden de [IP2Location™ IP Address Geolocation Database](https://www.ip2location.com/database/ip2location). Si buscas una base de datos de IP de alto rendimiento para geolocalización, utiliza directamente su producto.


La propiedad `isWhitelisted` refleja si la IP está en alguna de nuestras listas blancas. Nuestras listas blancas dan el beneficio de la duda a muchas IPs, por lo que generalmente no debería usarse como base para la acción. El `abuseConfidenceScore` es una mejor base para la acción, porque no es binario y permite matices. La propiedad `isWhitelisted` puede ser nula si no se ha realizado una búsqueda en la lista blanca.




