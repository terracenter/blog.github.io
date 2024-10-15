---
tiltle: Fail2Ban IDS + abuseIPDS
---

Integración de AbuseIPDB con Fail2Ban 
======================================

Denuncia automática de IPs maliciosas
-------------------------------------

**AbuseIPDB ofrece una API gratuita para denunciar y comprobar direcciones IP**. Todos los días, webmasters, administradores de sistemas y otros profesionales de TI utilizan nuestra API para denunciar en tiempo real miles de direcciones IP comprometidas con el spam, el hacking, el escaneo de vulnerabilidades y otras actividades maliciosas.


**Fail2Ban** es un [software de código abierto](https://github.com/fail2ban/fail2ban) que escanea archivos de registro como /var/log/auth.log y bloquea las direcciones IP que tienen demasiados intentos fallidos de inicio de sesión. Para ello, actualiza las reglas del cortafuegos del sistema para rechazar nuevas conexiones desde esas direcciones IP durante un periodo de tiempo configurable. También puede detectar y bloquear IPs implicadas en intentos de exploits web, portscanning y otras actividades abusivas.

En este tutorial, aprenderemos a integrar Fail2Ban con tu cuenta de AbuseIPDB para que los intentos de intrusión contra tu sistema sean reportados automáticamente a través de la API de AbuseIPDB, ayudando a contribuir a la base de datos distribuida de amenazas IP de [AbuseIPDB](). Si ya tienes instalada la última versión de Fail2Ban, ¡sólo te llevará unos minutos!

## Tutorial de integración de AbuseIPDB + Fail2Ban
1. Requisitos previos - Antes de comenzar este tutorial
2. Verifique que la Acción de Denuncia de AbuseIPDB de Fail2Ban esté instalada
3. Activación de la acción de informe de abuso de IPDB de Fail2Ban
3. Solución de problemas

Requisitos previos - Antes de empezar este tutorial
---------------------------------------------------
1. Instale Fail2Ban en su servidor
  
    Antes de comenzar este tutorial, asumimos que dispone de un servidor Linux con el sistema de detección de intrusos Fail2Ban instalado. Fail2Ban es software libre disponible en [https://github.com/fail2ban/fail2ban](https://github.com/fail2ban/). 
    
    Por favor, consulta la [documentación de Fail2Ban](http://www.fail2ban.org/wiki/index.php/MANUAL_0_8) o el [tutorial de instalación de Fail2Ban de DigitalOcean](https://www.digitalocean.com/community/tutorials/how-to-protect-ssh-with-fail2ban-on-centos-7) para instalar Fail2Ban y establecer la configuración básica de la jaula para detectar intentos de intrusión en SSH, Apache, etc.

2. Crear una clave API AbuseIPDB
    También asumimos que tienes una cuenta registrada en AbuseIPDB, y que has verificado tu dominio y creado una [clave API](https://www.abuseipdb.com/account). El uso de la API es gratuito, pero tienes que [crear una cuenta](https://www.abuseipdb.com/register).


Compruebe que la acción de denuncia de abusos de Fail2BanIPDB está instalada
----------------------------------------------------------------------------

La habilidad de reportar IPs abusivas directamente a AbuseIPDB fue añadida al repositorio maestro de Fail2Ban en v0.10.0 (Enero 2017). Si tienes una versión anterior de Fail2Ban instalada en tu servidor, tendrás que actualizar Fail2Ban o instalar tú mismo el archivo de acción abuseipdb.conf. Para comprobar qué versión de Fail2Ban tienes instalada, ejecuta el siguiente comando:

```bash
fail2ban-client -V
```

Puedes verificar que tu instalación de Fail2Ban soporta AbuseIPDB comprobando que el archivo de configuración de acciones /etc/fail2ban/action.d/abuseipdb.conf existe. Si no existe, puedes añadirlo manualmente copiando el último archivo de configuración de [Fail2Ban Github](https://github.com/fail2ban/fail2ban/blob/0.11/config/action.d/abuseipdb.conf).


Tu archivo ***/etc/fail2ban/jail.local*** (la versión personalizable de jail.conf) también debería contener la siguiente definición para "action_abuseipdb":

```
# Report ban via abuseipdb.com.
#
# See action.d/abuseipdb.conf for usage example and details.
#
action_abuseipdb = abuseipdb
```

Si este código de definición de acción no existe, añádelo a tu jail.local antes de la línea `action = %(action_)s`. ¡Eso es todo lo que necesitas para soportar la acción AbuseIPDB Fail2Ban!

### ¿Qué hará la acción AbuseIPDB?

La acción AbuseIPDB hará que la dirección IP que desencadenó un jail de Fail2Ban sea reportada automáticamente a la API de AbuseIPDB a través de un comando cURL. Por defecto, este informe incluirá el fragmento del archivo de registro que activó el bloqueo de Fail2Ban en el campo de comentarios del informe. El comentario puede ser modificado o desactivado buscando el fragmento '**comment=<matches>**' en ***action.d/abuseipdb.conf*** de la siguiente manera:

```
actionban = curl --tlsv1.0 --fail 'https://api.abuseipdb.com/api/v2/report' \
    -H 'Accept: application/json' \
    -H 'Key: <abuseipdb_apikey>' \
    --data-urlencode 'ip=<ip>' \
    --data-urlencode 'comment=<matches>' \
    --data 'categories=<abuseipdb_category>'
```

Activar la acción de denuncia de AbuseIPDB
------------------------------------------

Puedes invocar la acción AbuseIPDB desde algunas o todas las cárceles configuradas en jail.local. La acción debe ser invocada con dos parámetros - [tu clave API AbuseIPDB](https://www.abuseipdb.com/account), y la [categoría de abuso (o categorías)](https://www.abuseipdb.com/categories) por la que quieres reportar la IP. Si estos parámetros faltan o no son válidos, tus reportes fallarán.


`%(action_abuseipdb)s[abuseipdb_apikey="my-api-key", abuseipdb_category="18,22"]`

Esta línea de código debe añadirse a cada jaula para la que quieras activar los informes de AbuseIPDB. A continuación se muestra un ejemplo de cómo configurar la acción de informe de AbuseIPDB para que se ejecute, además de las acciones de bloqueo predeterminadas, cuando se activa la jaula de fuerza bruta sshd:

```
[sshd]
enabled = true

# To use more aggressive sshd modes set filter parameter "mode" in jail.local:
# normal (default), ddos, extra or aggressive (combines all).
# See "tests/files/logs/sshd" or "filter.d/sshd.conf" for usage example and details.
#mode   = normal
port    = ssh
logpath = %(sshd_log)s
backend = %(sshd_backend)s

# Ban IP and report to AbuseIPDB for SSH Brute-Forcing
action = %(action_)s
         %(action_abuseipdb)s[abuseipdb_apikey="my-api-key", abuseipdb_category="18,22"]
```

Le recomendamos que añada la acción AbuseIPDB individualmente a sus jaulas para que pueda personalizar las [categorías del informe AbuseIPDB](https://www.abuseipdb.com/categories) y hacer sus informes más específicos. Sin embargo, la acción AbuseIPDB también se puede añadir a la lista de acciones Fail2Ban en el global [DEFAULT] como se muestra a continuación, lo que hará que se ejecute en todas las jaulas sin una acción especificada:

```
# Choose default action.  To change, just override value of 'action' with the
# interpolation to the chosen action shortcut (e.g.  action_mw, action_mwl, etc) in jail.local
# globally (section [DEFAULT]) or per specific section
action = %(action_)s
         %(action_abuseipdb)s[abuseipdb_apikey="my-api-key", abuseipdb_category="18"]

```

A continuación encontrará una tabla con algunas de las [categorías de informes](https://www.abuseipdb.com/categories) más populares de AbuseIPDB para personalizar sus informes:

```
FTP Brute-Force     Port Scan 	Hacking  Brute-Force    Bad Web Bot	    SSH	    Web App Attack
     5	               14	      15	     18	             19	         22	        21
```

Una vez que haya actualizado la configuración de jail.local, guarde el archivo y reinicie o recargue el servicio Fail2Ban para asegurarse de que su configuración funciona:

`fail2ban-client reload`

Si tu configuración es correcta, Fail2Ban debería empezar a ejecutar la acción AbuseIPDB cada vez que una nueva IP es baneada. Inicia sesión y revisa tu [página de IPs reportadas](https://www.abuseipdb.com/account/reports), ¡y observa como Fail2Ban comienza a reportar automáticamente IPs a AbuseIPDB bajo tu cuenta!


Solución de problemas
---------------------

Si los informes no se transmiten automáticamente a su cuenta, compruebe el archivo de registro de Fail2Ban con sudo `less /var/log/fail2ban.log`. Puedes verificar que las IPs están siendo baneadas correctamente por tus jaulas, y comprobar si hay errores cURL que puedan estar causando que tus informes fallen.


### Errores en la clave y los parámetros de la API:

Los problemas con el envío de la API, como una clave de API no válida o categorías no válidas, pueden no aparecer como error en el archivo de registro de Fail2Ban. Debe verificar que sus parámetros son correctos consultando la [documentación de la API](https://www.abuseipdb.com/api.html) y ejecutando un informe de prueba a través de la API. Los informes de prueba pueden ser borrados en su [página de IPs reportadas](https://www.abuseipdb.com/account/reports).


### Límites y estrangulamiento de la API:

Por defecto, los límites de uso de la API están limitados a 1.000 informes por día. Estos límites se incrementan a 3.000 para webmasters verificados y a 5.000 para [colaboradores](https://www.abuseipdb.com/account/contributor), lo que es muy recomendable para los usuarios de Fail2Ban (especialmente si tiene Fail2Ban funcionando en varios servidores informando a la misma clave API).

También impedimos que la misma IP sea reportada más de una vez cada 15 minutos para evitar reportes duplicados. Le recomendamos que aumente su configuración de `bantime` para evitar que estas IPs malas ataquen repetidamente su sistema. Un valor de 900 (15 minutos en segundos) le prevendrá de golpear nuestra ventana de 15 minutos.

* **Nota: si tiene previsto establecer la opción de configuración** `findtime` **para una jaula, no supere los 60 días (5.184.000 segundos).** * Esto se ajusta a nuestra [política de informes](https://www.abuseipdb.com/reporting-policy).

**Informes duplicados al reiniciar:**

Cada vez que Fail2Ban se reinicia, llama a la función `actionban` para cada IP almacenada en el archivo de base de datos. Esto causa reportes duplicados a AbuseIPDB. Si reinicias tu servidor a menudo, tenemos un script que evitará que esto ocurra.

1. Cree el archivo **abuseipdb-fail2ban-report.sh**

    Introduzca `touch abuseipdb-fail2ban-report.sh` en su terminal.

2. Copie el siguiente código en el archivo creado, actualizando la ruta al archivo `REPORTED_IP_LIST_FILE` si es necesario

    ```
    #!/bin/bash

    REPORTED_IP_LIST_FILE=/home/pi/abuseipdb-reported-ip-list
    FAIL2BAN_SQLITE_DB=/var/lib/fail2ban/fail2ban.sqlite3

    APIKEY=$1
    COMMENT=$2
    IP=$3
    CATEGORIES=$4
    BANTIME=$5

    ipMatch=`grep -Fe "IP=$IP L=[0-9\-]+" $REPORTED_IP_LIST_FILE`

    shouldBanIP=1
    currentTimestamp=`date +%s`

    if [ -z $ipMatch ] ; then
        banLength=`echo $ipMatch | sed -E 's/.*L=([0-9\-]+)/\1/'`
        timeOfBan=`sqlite3 $FAIL2BAN_SQLITE_DB "SELECT timeofban FROM bans WHERE ip = '$IP'"`

        if (((banLength == -1 && banLength == BANTIME) || (timeOfBan > 0 && timeOfBan + banLength > currentTimestamp))) ; then
            shouldBanIP=0
        else
            sed -i "/^IP=$IP.*$/d" $REPORTED_IP_LIST_FILE
        fi
    fi

    if [ $shouldBanIP -eq 1 ] ; then
        echo "IP=$IP L=$BANTIME" >> $REPORTED_IP_LIST_FILE
        curl --fail 'https://api.abuseipdb.com/api/v2/report' \
            -H 'Accept: application/json' \
            -H "Key: $APIKEY" \
            --data-urlencode "comment=$COMMENT" \
            --data-urlencode "ip=$IP" \
            --data "categories=$CATEGORIES"
    fi
    ```

3. Convertir el archivo en ejecutable

    Escribe `chmod +x abuseipdb-fail2ban-report.sh` en tu terminal

4. Modifica `/etc/fail2ban/action.d/abuseipdb.conf` `actionban` para que sea lo siguiente:

    ```
    actionban = /path/to/file/abuseipdb-fail2ban-report.sh \

    "<abuseipdb_apikey>" "<matches>" "<ip>" "<abuseipdb_category>" "<bantime>"
    ```

5. Cambia la línea `%(action_abuseipdb)s[abuseipdb_apikey="my-api-key", abuseipdb_category="18,22"]` en tu **jail.local** por:

    ```
    %(action_abuseipdb)s[abuseipdb_apikey="my-api-key", abuseipdb_category="18,22", bantime="%(bantime)s"]
    ```

    Para cada jaula a la que desee que se aplique.

6. Reinicie Fail2Ban usando 

    `fail2ban-client reload`

