## 17 Cifrado

### Visión general

Zabbix soporta comunicaciones encriptadas entre componentes Zabbix usando el protocolo Transport Layer Security (TLS) v.1.2 y 1.3 (dependiendo de la librería crypto). Se admite el cifrado basado en certificados y en claves precompartidas.

Se puede configurar el cifrado de las conexiones:

* Entre servidor Zabbix, proxy Zabbix, agente Zabbix, utilidades zabbix_sender y zabbix_get
* A la base de datos Zabbix desde el [frontend Zabbix y el servidor/proxy](https://www.zabbix.com/documentation/devel/en/manual/appendix/install/db_encrypt)

El cifrado es opcional y configurable para componentes individuales

* Algunos proxies y agentes pueden configurarse para utilizar el cifrado basado en certificados con el servidor, mientras que otros pueden utilizar el cifrado basado en claves precompartidas, y otros continúan con las comunicaciones sin cifrar (como antes).
* El servidor (proxy) puede utilizar diferentes configuraciones de cifrado para diferentes hosts

Los programas daemon de Zabbix utilizan un puerto de escucha para las conexiones entrantes encriptadas y no encriptadas. Añadir un cifrado no requiere abrir nuevos puertos en los cortafuegos.

### Limitaciones

* Las claves privadas se almacenan en texto plano en archivos legibles por los componentes de Zabbix durante el arranque.
* Las claves precompartidas se introducen en el frontend de Zabbix y se almacenan en la base de datos de Zabbix en texto plano.
* El cifrado incorporado no protege las comunicaciones:
  - Entre el servidor web que ejecuta el frontend Zabbix y el navegador web del usuario
  - Entre el frontend Zabbix y el servidor Zabbix
* Actualmente cada conexión encriptada se abre con un handshake TLS completo, no hay caché de sesión ni tickets implementados.
* Al añadir el cifrado aumenta el tiempo de las comprobaciones y acciones de los elementos, en función de la latencia de la red.
  - Por ejemplo, si el retardo del paquete es de 100 ms, abrir una conexión TCP y enviar una solicitud sin cifrar tarda unos 200 ms. Con el cifrado se añaden unos 1000 ms para establecer la conexión TLS. Con cifrado se añaden unos 1000 ms para establecer la conexión TLS;
  - Puede que sea necesario aumentar los tiempos de espera, de lo contrario algunos elementos y acciones que ejecutan scripts remotos en los agentes pueden funcionar con conexiones sin cifrar, pero fallar con tiempo de espera con cifrado.
* El cifrado no está soportado por el [descubrimiento de red](https://www.zabbix.com/documentation/devel/en/manual/discovery/network_discovery). Las comprobaciones del agente Zabbix realizadas por el descubrimiento de red no estarán cifradas y si el agente Zabbix está configurado para rechazar conexiones no cifradas, dichas comprobaciones no tendrán éxito.

### Compilación de Zabbix con soporte de encriptación

* Para soportar encriptación Zabbix debe ser compilado y enlazado con una de las librerías crypto soportadas:
  - GnuTLS - a partir de la versión 3.1.18
  - OpenSSL - versiones 1.0.1, 1.0.2, 1.1.0, 1.1.1, 3.0.x
  - LibreSSL - probado con las versiones 2.7.4, 2.8.2:
    * LibreSSL 2.6.x no es compatible
    * LibreSSL es soportado como un reemplazo compatible de OpenSSL; las nuevas funciones tls_*() API específicas de LibreSSL no son utilizadas. Los componentes de Zabbix compilados con LibreSSL no podrán usar PSK, sólo se pueden usar certificados.

**NOTA**:

> Puede encontrar más información sobre la configuración de SSL para Zabbix frontend consultando estas mejores prácticas.

La biblioteca se selecciona especificando la opción correspondiente en el script "configure":

* --con-gnutls[=DIR]
* --with-openssl[=DIR] (también se utiliza para LibreSSL)

Por ejemplo, para configurar las fuentes para el servidor y el agente con OpenSSL puede usar algo como:

```bash
./configure --enable-server --enable-agent --with-mysql --enable-ipv6 --with-net-snmp --with-libcurl --with-libxml2 --with-openssl
```

Diferentes componentes de Zabbix pueden ser compilados con diferentes librerías criptográficas (por ejemplo, un servidor con OpenSSL, un agente con GnuTLS).

**ATENCIÓN**:

> Si planea usar claves pre-compartidas (PSK), considere usar las librerías GnuTLS u OpenSSL 1.1.0 (o más recientes) en componentes Zabbix que usen PSKs. Las librerías GnuTLS y OpenSSL 1.1.0 soportan cifrados PSK con Perfect Forward Secrecy. Las versiones anteriores de la biblioteca OpenSSL (1.0.1, 1.0.2c) también admiten PSK, pero los cifrados PSK disponibles no proporcionan Perfect Forward Secrecy.

### Gestión del cifrado de la conexión

Conexiones en Zabbix puede utilizar:

* Sin cifrado (por defecto)
* [Cifrado basado en certificado RSA](https://www.zabbix.com/documentation/devel/en/manual/encryption/using_certificates)
* [Cifrado basado en PSK](https://www.zabbix.com/documentation/devel/en/manual/encryption/using_pre_shared_keys)

Hay dos parámetros importantes utilizados para especificar el cifrado entre los componentes de Zabbix:

* TLSConnect - especifica qué encriptación utilizar para las conexiones salientes (sin encriptar, PSK o certificado).
* TLSAccept - especifica qué tipos de conexiones se permiten para las conexiones entrantes (sin cifrar, PSK o certificado). Se pueden especificar uno o más valores.

`TLSConnect` se utiliza en los ficheros de configuración del proxy Zabbix (en modo activo, especifica sólo conexiones al servidor) y del agente Zabbix (para comprobaciones activas). En el frontend de Zabbix el equivalente a TLSConnect es el campo Conexiones a host en **Data collection → Hosts → <mismo host> → pestaña Encryption** y el campo Conexiones a proxy en **Administration → Proxies → <mismo proxy> → pestaña Encryption**. Si el tipo de cifrado configurado para la conexión falla, no se intentará ningún otro tipo de cifrado.

`TLSAccept` se utiliza en los ficheros de configuración del proxy Zabbix (en modo pasivo, especifica sólo conexiones desde el servidor) y del agente Zabbix (para comprobaciones pasivas). En el frontend de Zabbix el equivalente a TLSAccept es el campo Conexiones desde host en **Data collection → Hosts → <mismo host> → pestaña Encryption** y el campo Conexiones desde proxy en **Administration → Proxies → <mismo proxy> → pestaña Encryption**.

Normalmente sólo se configura un tipo de cifrado para los cifrados entrantes. Pero es posible que desee cambiar el tipo de cifrado, por ejemplo, de no cifrado a basado en certificado con el mínimo tiempo de inactividad y posibilidad de reversión. Para ello:

* Establece `TLSAccept=unencrypted,cert` en el fichero de configuración del agente y reinicia el agente Zabbix
* Prueba la conexión con zabbix_get al agente usando certificado. Si funciona, puede reconfigurar el cifrado para ese agente en el frontend de Zabbix en la pestaña **Data collection → Hosts → <mismo host> → Encryption** colocando  `Connections to host` en `Certificate`.
* Cuando la caché de configuración del servidor se actualiza (y la configuración del proxy se actualiza si el host es monitorizado por el proxy) entonces las conexiones a ese agente serán encriptadas.
* Si todo funciona como se espera puedes establecer `TLSAccept=cert` en el fichero de configuración del agente y reiniciar el agente Zabbix. Ahora el agente sólo aceptará conexiones cifradas basadas en certificados. Las conexiones sin cifrar y basadas en PSK serán rechazadas.

De forma similar funciona en servidor y proxy. Si en el frontend de Zabbix en la configuración del host Conexiones desde el host se establece en "Certificado" entonces sólo se aceptarán conexiones encriptadas basadas en certificados desde el agente (comprobaciones activas) y zabbix_sender (elementos trapper).

Lo más probable es que configures las conexiones entrantes y salientes para que utilicen el mismo tipo de cifrado o ningún tipo de cifrado. Pero técnicamente es posible configurarlo de forma asimétrica, por ejemplo, cifrado basado en certificados para las conexiones entrantes y basado en PSK para las salientes.

La configuración de cifrado para cada host se muestra en el frontend de Zabbix, en ***Data collection → Hosts*** en la columna ***Agent encryption**. Por ejemplo:


| Ejemplo                                      | Connections to host             | Allowed connections from host                   | Rejected connections from host               |
| ---------------------------------------------- | --------------------------------- | ------------------------------------------------- | ---------------------------------------------- |
| ![](assets/20240208_110826_none_none.png)    | Sin cifrar                      | Sin cifrar                                      | Cifrado, certificado y cifrado basado en PSK |
| ![](assets/20240208_110916_cert_cert.png)    | Cifrado, basado en certificados | Cifrado basado en certificados                  | Sin cifrar y cifrado basado en PSK           |
| ![](assets/20240208_111001_psk_psk.png)      | Cifrado, basado en PSK          | Cifrado, basado en PSK                          | Sin cifrar y cifrado basado en certificados  |
| ![](assets/20240208_111154_psk_none_psk.png) | Cifrado, basado en PSK          | Sin cifrar y cifrado basado en PSK              | Cifrado basado en certificados               |
| ![](assets/20240208_111240_cert_all.png)     | Cifrado, basado en certificados | Sin cifrar, PSK o cifrado basado en certificado |                                              |

**ATENCIÓN:**

> Las conexiones no están cifradas por defecto. El cifrado debe configurarse individualmente para cada host y proxy.

### zabbix_get y zabbix_sender con cifrado

Consulte las páginas de manual [zabbix_get](https://www.zabbix.com/documentation/devel/en/manpages/zabbix_get) y [zabbix_sender](https://www.zabbix.com/documentation/devel/en/manual/encryption#:~:text=See%20zabbix_get%20and-,zabbix_sender,-manpages%20for%20using) para utilizarlas con cifrado.

### Ciphersuites

Los cifrados por defecto se configuran internamente durante el arranque de Zabbix.

También se admiten cifrados configurados por el usuario para GnuTLS y OpenSSL. Los usuarios pueden configurar los cifradores de acuerdo con sus políticas de seguridad. El uso de esta función es opcional (los cifrados por defecto siguen funcionando).

Para las bibliotecas criptográficas compiladas con la configuración predeterminada, las reglas incorporadas de Zabbix suelen dar como resultado los siguientes cifrados (en orden de mayor a menor prioridad):


| BibliotecaLibrar | Cifrados de certificado                                                                                                                                                                                                                                       | Cifrados PSK                                                                                                                                                                                                                           |
| ------------------ | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| *GnuTLS 3.1.18*  | TLS_ECDHE_RSA_AES_128_GCM_SHA256<br/>TLS_ECDHE_RSA_AES_128_CBC_SHA256<br/>TLS_ECDHE_RSA_AES_128_CBC_SHA1<br/>TLS_RSA_AES_128_GCM_SHA256<br/>TLS_RSA_AES_128_CBC_SHA256<br/>TLS_RSA_AES_128_CBC_SHA1                                                           | TLS_ECDHE_PSK_AES_128_CBC_SHA256<br/>TLS_ECDHE_PSK_AES_128_CBC_SHA1<br/>TLS_PSK_AES_128_GCM_SHA256<br/>TLS_PSK_AES_128_CBC_SHA256<br/>TLS_PSK_AES_128_CBC_SHA1                                                                         |
| *OpenSSL 1.0.2c* | ECDHE-RSA-AES128-GCM-SHA256<br/>ECDHE-RSA-AES128-SHA256<br/>ECDHE-RSA-AES128-SHA<br/>AES128-GCM-SHA256<br/>AES128-SHA256<br/>AES128-SHA                                                                                                                       | PSK-AES128-CBC-SHA                                                                                                                                                                                                                     |
| *OpenSSL 1.1.0*  | ECDHE-RSA-AES128-GCM-SHA256<br/>ECDHE-RSA-AES128-SHA256<br/>ECDHE-RSA-AES128-SHA<br/>AES128-GCM-SHA256<br/>AES128-CCM8<br/>AES128-CCM<br/>AES128-SHA256<br/>AES128-SHA<br/>                                                                                   | ECDHE-PSK-AES128-CBC-SHA256<br/>ECDHE-PSK-AES128-CBC-SHA<br/>PSK-AES128-GCM-SHA256<br/>PSK-AES128-CCM8<br/>PSK-AES128-CCM<br/>PSK-AES128-CBC-SHA256<br/>PSK-AES128-CBC-SHA                                                             |
| *OpenSSL 1.1.1d* | TLS_AES_256_GCM_SHA384<br/>TLS_CHACHA20_POLY1305_SHA256<br/>TLS_AES_128_GCM_SHA256<br/>ECDHE-RSA-AES128-GCM-SHA256<br/>ECDHE-RSA-AES128-SHA256<br/>ECDHE-RSA-AES128-SHA<br/>AES128-GCM-SHA256<br/>AES128-CCM8<br/>AES128-CCM<br/>AES128-SHA256<br/>AES128-SHA | TLS_CHACHA20_POLY1305_SHA256<br/>TLS_AES_128_GCM_SHA256<br/>ECDHE-PSK-AES128-CBC-SHA256<br/>ECDHE-PSK-AES128-CBC-SHA<br/>PSK-AES128-GCM-SHA256<br/>PSK-AES128-CCM8<br/>PSK-AES128-CCM<br/>PSK-AES128-CBC-SHA256<br/>PSK-AES128-CBC-SHA |

### Cifrados configurados por el usuario

Los criterios de selección de cifrado incorporados pueden anularse con cifrados configurados por el usuario.

**ATENCIÓN**:

> Los cifrados configurados por el usuario son una función destinada a usuarios avanzados que entienden los cifrados TLS, su seguridad y las consecuencias de los errores, y que se sienten cómodos con la resolución de problemas TLS.

Los criterios de selección de la cifrado incorporada pueden anularse utilizando los siguientes parámetros:


| Anular ámbito                                     | Parámetro      | Valor                                                                                                                                                                                                                                                                                                             | Descripción                                                                                            |
| ---------------------------------------------------- | ----------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------- |
| Selección de cifrado para certificados            | TLSCipherCert13 | Valid OpenSSL 1.1.1[cipher strings](https://www.openssl.org/docs/man1.1.1/man1/ciphers.html) for TLS 1.3 protocol (their values are passed to the OpenSSL function `SSL_CTX_set_ciphersuites()`).                                                                                                                 | Criterios de selección de cifrados basados en certificados para TLS 1.3                                |
|                                                    | TLSCipherCert   | Valid OpenSSL[cipher strings](https://www.openssl.org/docs/man1.1.1/man1/ciphers.html) for TLS 1.2 or valid GnuTLS [priority strings](https://gnutls.org/manual/html_node/Priority-Strings.html). Their values are passed to the `SSL_CTX_set_cipher_list()` or `gnutls_priority_init()` functions, respectively. | Criterios de selección de cifrado basados en certificados para TLS 1.2/1.3 (GnuTLS), TLS 1.2 (OpenSSL) |
| Selección de cifrado para PSK                     | TLSCipherPSK13  | Valid OpenSSL 1.1.1[cipher strings](https://www.openssl.org/docs/man1.1.1/man1/ciphers.html) for TLS 1.3 protocol (their values are passed to the OpenSSL function `SSL_CTX_set_ciphersuites()`).                                                                                                                 | Criterios de selección de cifrados basados en PSK para TLS 1.3  Solo OpenSSL 1.1.1 o posterior.        |
|                                                    | TLSCipherPSK    | Valid OpenSSL[cipher strings](https://www.openssl.org/docs/man1.1.1/man1/ciphers.html) for TLS 1.2 or valid GnuTLS [priority strings](https://gnutls.org/manual/html_node/Priority-Strings.html). Their values are passed to the `SSL_CTX_set_cipher_list()` or `gnutls_priority_init()` functions, respectively. | PSK-based ciphersuite selection criteria for TLS 1.2/1.3 (GnuTLS), TLS 1.2 (OpenSSL)                    |
| Lista combinada de cifrados para certificado y PSK | TLSCipherAll13  | Valid OpenSSL 1.1.1[cipher strings](https://www.openssl.org/docs/man1.1.1/man1/ciphers.html) for TLS 1.3 protocol (their values are passed to the OpenSSL function `SSL_CTX_set_ciphersuites()`).                                                                                                                 | Ciphersuite selection criteria for TLS 1.3<br/><br/>Only OpenSSL 1.1.1 or newer.                        |
|                                                    | TLSCipherAll    | Valid OpenSSL[cipher strings](https://www.openssl.org/docs/man1.1.1/man1/ciphers.html) for TLS 1.2 or valid GnuTLS [priority strings](https://gnutls.org/manual/html_node/Priority-Strings.html). Their values are passed to the `SSL_CTX_set_cipher_list()` or `gnutls_priority_init()` functions, respectively. | Ciphersuite selection criteria for TLS 1.2/1.3 (GnuTLS), TLS 1.2 (OpenSSL)                              |

Para anular la selección de ciphersuite en las utilidades zabbix_get y zabbix_sender - utilice los parámetros de línea de comandos:

* --tls-cipher13
* --tls-cipher

Los nuevos parámetros son opcionales. Si no se especifica un parámetro, se utiliza el valor interno por defecto. Si se define un parámetro, no puede estar vacío.

Si la configuración de un valor TLSCipher* en la biblioteca crypto falla, el servidor, proxy o agente no se iniciará y se registrará un error.

Es importante comprender cuándo es aplicable cada parámetro.

#### Conexiones salientes

El caso más sencillo es el de las conexiones salientes:

* Para conexiones salientes con certificado - utilice TLSCipherCert13 o TLSCipherCert
* Para conexiones salientes con PSK - utilice TLSCipherPSK13 o TLSCipherPSK
* En el caso de las utilidades zabbix_get y zabbix_sender pueden utilizarse los parámetros de línea de comandos --tls-cipher13 o --tls-cipher (el cifrado se especifica inequívocamente con un parámetro --tls-connect)

#### Conexiones entrantes

Es un poco más complicado con las conexiones entrantes porque las reglas son específicas para los componentes y la configuración.

Para el agente **Zabbix**:


| Configuración de la conexión del agente | Configuración del cifrado     |
| ------------------------------------------- | -------------------------------- |
| TLSConnect=cert                           | TLSCipherCert, TLSCipherCert13 |
| TLSConnect=psk                            | TLSCipherPSK, TLSCipherPSK13   |
| TLSAccept=cert                            | TLSCipherCert, TLSCipherCert13 |
| TLSAccept=psk                             | TLSCipherPSK, TLSCipherPSK13   |
| TLSAccept=cert,psk                        | TLSCipherAll, TLSCipherAll13   |

Para el **Servidor** y el **Proxy**:


| Configuración de la conexión                              | Configuración del cifrado   |
| ------------------------------------------------------------- | ------------------------------ |
| Outgoing connections using PSK                              | TLSCipherPSK, TLSCipherPSK13 |
| Incoming connections using certificates                     | TLSCipherAll, TLSCipherAll13 |
| Incoming connections using PSK if server has no certificate | TLSCipherPSK, TLSCipherPSK13 |
| Incoming connections using PSK if server has certificate    | TLSCipherAll, TLSCipherAll13 |

En las dos tablas anteriores se puede observar un cierto patrón:

* TLSCipherAll y TLSCipherAll13 sólo pueden especificarse si se utiliza una lista combinada de cifrados basados en certificados **y** PSK. Hay dos casos en los que tiene lugar: servidor (proxy) con un certificado configurado (los cifrados PSK siempre se configuran en el servidor, proxy si la biblioteca criptográfica admite PSK), agente configurado para aceptar conexiones entrantes basadas tanto en certificados como en PSK.
* En otros casos TLSCipherCert* y/o TLSCipherPSK* son suficientes

Las siguientes tablas muestran los valores por defecto incorporados en TLSCipher*. Podrían ser un buen punto de partida para sus propios valores personalizados.


| Parámetro    | GnuTLS 3.6.12                                                                                                                                |
| --------------- | ---------------------------------------------------------------------------------------------------------------------------------------------- |
| TLSCipherCert | NONE:+VERS-TLS1.2:+ECDHE-RSA:+RSA:+AES-128-GCM:+AES-128-CBC:+AEAD:+SHA256:+SHA1:+CURVE-ALL:+COMP-NULL:+SIGN-ALL:+CTYPE-X.509                 |
|               |                                                                                                                                              |
| TLSCipherPSK  | NONE:+VERS-TLS1.2:+ECDHE-PSK:+PSK:+AES-128-GCM:+AES-128-CBC:+AEAD:+SHA256:+SHA1:+CURVE-ALL:+COMP-NULL:+SIGN-ALL                              |
| TLSCipherAll  | NONE:+VERS-TLS1.2:+ECDHE-RSA:+RSA:+ECDHE-PSK:+PSK:+AES-128-GCM:+AES-128-CBC:+AEAD:+SHA256:+SHA1:+CURVE-ALL:+COMP-NULL:+SIGN-ALL:+CTYPE-X.509 |


| Parámetro      | OpenSSL 1.1.1d ^**1**^                                         |
| ----------------- | ---------------------------------------------------------------- |
| TLSCipherCert13 |                                                                |
| TLSCipherCert   | EECDH+aRSA+AES128:RSA+aRSA+AES128                              |
| TLSCipherPSK13  | TLS_CHACHA20_POLY1305_SHA256:TLS_AES_128_GCM_SHA256            |
| TLSCipherPSK    | kECDHEPSK+AES128:kPSK+AES128                                   |
| TLSCipherAll13  |                                                                |
| TLSCipherAll    | EECDH+aRSA+AES128:RSA+aRSA+AES128:kECDHEPSK+AES128:kPSK+AES128 |

^1^ Los valores por defecto son diferentes para versiones antiguas de OpenSSL (1.0.1, 1.0.2, 1.1.0), para LibreSSL y si OpenSSL se compila sin soporte PSK.

##### Ejemplos de cifrados configurados por el usuario

Vea a continuación los siguientes ejemplos de cifrados configurados por el usuario:

* [Probar cadenas de cifrado y permitir sólo cifrados PFS](https://www.zabbix.com/documentation/devel/en/manual/encryption#testing-cipher-strings-and-allowing-only-pfs-ciphersuites)
* [Cambio de AES128 a AES256](https://www.zabbix.com/documentation/devel/en/manual/encryption#switching-from-aes128-to-aes256)

### Probar cadenas de cifrado y permitir sólo cifrados PFS

Para ver qué cifrados se han seleccionado es necesario establecer `DebugLevel=4` en el archivo de configuración, o utilizar la opción `-vv` para `zabbix_sender`.

Puede que sea necesario experimentar con los parámetros de TLSCipher* antes de obtener los cifrados deseados. Es inconveniente reiniciar el servidor Zabbix, proxy o agente varias veces sólo para ajustar los parámetros TLSCipher*. Opciones más convenientes son usar **zabbix_sender** o el comando **openssl**. Vamos a mostrar ambos.

1. Usando zabbix_sender.
  Hagamos un archivo de configuración de prueba, por ejemplo /home/zabbix/test.conf, con la sintaxis de un archivo zabbix_agentd.conf:
    ```
    Hostname=nonexisting
    ServerActive=nonexisting
    
    TLSConnect=cert
    TLSCAFile=/home/zabbix/ca.crt
    TLSCertFile=/home/zabbix/agent.crt
    TLSKeyFile=/home/zabbix/agent.key
    TLSPSKIdentity=nonexisting
    TLSPSKFile=/home/zabbix/agent.psk
    ```

    Para este ejemplo necesita certificados CA y de agente y PSK válidos. Adapte las rutas y los nombres de los archivos de certificados y PSK a su entorno.

    Si no utiliza certificados, sino sólo PSK, puede crear un archivo de prueba más sencillo:
    ```
    Hostname=nonexisting
    ServerActive=nonexisting
  
    TLSConnect=psk
    TLSPSKIdentity=nonexisting
    TLSPSKFile=/home/zabbix/agentd.psk
    ```

    Los cifrados seleccionados pueden verse ejecutando zabbix_sender (ejemplo compilado con OpenSSL 1.1.d):
    ```bash
    zabbix_sender -vv -c /home/zabbix/test.conf -k nonexisting_item -o 1 2>&1 | grep cifrado
    ```
    ```
    zabbix_sender [41271]: DEBUG: zbx_tls_init_child() certificate ciphersuites: TLS_AES_256_GCM_SHA384 TLS_CHACHA20_POLY1305_SHA256 TLS_AES_128_GCM_SHA256 ECDHE-RSA-AES128-GCM-SHA256 ECDHE-RSA-AES128-SHA256 ECDHE-RSA-AES128-SHA AES128-GCM-SHA256 AES128-CCM8 AES128-CCM AES128-SHA256 AES128-SHA
    zabbix_sender [41271]: DEBUG: zbx_tls_init_child() PSK ciphersuites: TLS_CHACHA20_POLY1305_SHA256 TLS_AES_128_GCM_SHA256 ECDHE-PSK-AES128-CBC-SHA256 ECDHE-PSK-AES128-CBC-SHA PSK-AES128-GCM-SHA256 PSK-AES128-CCM8 PSK-AES128-CCM PSK-AES128-CBC-SHA256 PSK-AES128-CBC-SHA
    zabbix_sender [41271]: DEBUG: zbx_tls_init_child() certificate and PSK ciphersuites: TLS_AES_256_GCM_SHA384 TLS_CHACHA20_POLY1305_SHA256 TLS_AES_128_GCM_SHA256 ECDHE-RSA-AES128-GCM-SHA256 ECDHE-RSA-AES128-SHA256 ECDHE-RSA-AES128-SHA AES128-GCM-SHA256 AES128-CCM8 AES128-CCM AES128-SHA256 AES128-SHA ECDHE-PSK-AES128-CBC-SHA256 ECDHE-PSK-AES128-CBC-SHA PSK-AES128-GCM-SHA256 PSK-AES128-CCM8 PSK-AES128-CCM PSK-AES128-CBC-SHA256 PSK-AES128-CBC-SHA
    ```

    Aquí puede ver los cifrados seleccionados por defecto. Estos valores por defecto se eligen para garantizar la interoperabilidad con los agentes Zabbix que se ejecutan en sistemas con versiones anteriores de OpenSSL (a partir de 1.0.1).

    Con los nuevos sistemas puedes elegir reforzar la seguridad permitiendo sólo unos pocos cifrados, por ejemplo, sólo cifrados con PFS (Perfect Forward Secrecy). Intentemos permitir sólo cifrados con PFS usando los parámetros TLSCipher*.

    **ATENCIÓN**:

    > El resultado no será interoperable con sistemas que utilicen OpenSSL 1.0.1 y 1.0.2, si se utiliza PSK. El cifrado basado en certificados debería funcionar

    Añada dos líneas al archivo de configuración test.conf:
    ```
    TLSCipherCert=EECDH+aRSA+AES128
    TLSCipherPSK=kECDHEPSK+AES128
    ```

    y prueba de nuevo:
    ```bash
    zabbix_sender -vv -c /home/zabbix/test.conf -k nonexisting_item -o 1 2>&1 | grep ciphersuites            
    ```
    ```
    zabbix_sender [42892]: DEBUG: zbx_tls_init_child() certificate ciphersuites: TLS_AES_256_GCM_SHA384 TLS_CHACHA20_POLY1305_SHA256 TLS_AES_128_GCM_SHA256 ECDHE-RSA-AES128-GCM-SHA256 ECDHE-RSA-AES128-SHA256 ECDHE-RSA-AES128-SHA        
    zabbix_sender [42892]: DEBUG: zbx_tls_init_child() PSK ciphersuites: TLS_CHACHA20_POLY1305_SHA256 TLS_AES_128_GCM_SHA256 ECDHE-PSK-AES128-CBC-SHA256 ECDHE-PSK-AES128-CBC-SHA        
    zabbix_sender [42892]: DEBUG: zbx_tls_init_child() certificate and PSK ciphersuites: TLS_AES_256_GCM_SHA384 TLS_CHACHA20_POLY1305_SHA256 TLS_AES_128_GCM_SHA256 ECDHE-RSA-AES128-GCM-SHA256 ECDHE-RSA-AES128-SHA256 ECDHE-RSA-AES128-SHA AES128-GCM-SHA256 AES128-CCM8 AES128-CCM AES128-SHA256 AES128-SHA ECDHE-PSK-AES128-CBC-SHA256 ECDHE-PSK-AES128-CBC-SHA PSK-AES128-GCM-SHA256 PSK-AES128-CCM8 PSK-AES128-CCM PSK-AES128-CBC-SHA256 PSK-AES128-CBC-SHA
    ```

    Las listas "certificados cifrados" y "PSK cifrados" han cambiado: son más cortas que antes y sólo contienen cifrados TLS 1.3 y cifrados TLS 1.2 ECDHE-*, como era de esperar.

2. TLSCipherAll y TLSCipherAll13 no se pueden probar con zabbix_sender; no afectan al valor de "certificate and PSK cifrados" mostrado en el ejemplo anterior. Para ajustar TLSCipherAll y TLSCipherAll13 necesita experimentar con el agente, proxy o servidor.

    Por lo tanto, para permitir sólo cifrados PFS, puede que tenga que añadir hasta tres parámetros
    ```
    TLSCipherCert=EECDH+aRSA+AES128
    TLSCipherPSK=kECDHEPSK+AES128
    TLSCipherAll=EECDH+aRSA+AES128:kECDHEPSK+AES128
    ```

    A zabbix_agentd.conf, zabbix_proxy.conf y zabbix_server_conf si cada uno de ellos tiene un certificado configurado y el agente también tiene PSK.

    Si su entorno Zabbix utiliza sólo el cifrado basado en PSK y sin certificados, entonces sólo uno:
    ``` 
    TLSCipherPSK=kECDHEPSK+AES128
    ```

    Ahora que entiendes cómo funciona puedes probar la selección de la cifrado incluso fuera de Zabbix, con el comando openssl. Vamos a probar los tres valores del parámetro TLSCipher*:
    ```bash
    openssl ciphers EECDH+aRSA+AES128 | sed 's/:/ /g'
    ```
    ```
    TLS_AES_256_GCM_SHA384 TLS_CHACHA20_POLY1305_SHA256 TLS_AES_128_GCM_SHA256 ECDHE-RSA-AES128-GCM-SHA256 ECDHE-RSA-AES128-SHA256 ECDHE-RSA-AES128-SHA
    ```
    ```bash
    openssl ciphers kECDHEPSK+AES128 | sed 's/:/ /g'
    ```
    ```
    TLS_AES_256_GCM_SHA384 TLS_CHACHA20_POLY1305_SHA256 TLS_AES_128_GCM_SHA256 ECDHE-PSK-AES128-CBC-SHA256 ECDHE-PSK-AES128-CBC-SHA
    ```
    ```bash
    openssl ciphers EECDH+aRSA+AES128:kECDHEPSK+AES128 | sed 's/:/ /g'
    ```
    ```
    TLS_AES_256_GCM_SHA384 TLS_CHACHA20_POLY1305_SHA256 TLS_AES_128_GCM_SHA256 ECDHE-RSA-AES128-GCM-SHA256 ECDHE-RSA-AES128-SHA256 ECDHE-RSA-AES128-SHA ECDHE-PSK-AES128-CBC-SHA256 ECDHE-PSK-AES128-CBC-SHA
    ```
    Puede que prefiera openssl ciphers con la opción -V para una salida más verbosa:

    ```bash
    openssl ciphers -V EECDH+aRSA+AES128:kECDHEPSK+AES128
    ```
    ```
            0x13,0x02 - TLS_AES_256_GCM_SHA384  TLSv1.3 Kx=any      Au=any  Enc=AESGCM(256) Mac=AEAD
            0x13,0x03 - TLS_CHACHA20_POLY1305_SHA256 TLSv1.3 Kx=any      Au=any  Enc=CHACHA20/POLY1305(256) Mac=AEAD
            0x13,0x01 - TLS_AES_128_GCM_SHA256  TLSv1.3 Kx=any      Au=any  Enc=AESGCM(128) Mac=AEAD
            0xC0,0x2F - ECDHE-RSA-AES128-GCM-SHA256 TLSv1.2 Kx=ECDH     Au=RSA  Enc=AESGCM(128) Mac=AEAD
            0xC0,0x27 - ECDHE-RSA-AES128-SHA256 TLSv1.2 Kx=ECDH     Au=RSA  Enc=AES(128)  Mac=SHA256
            0xC0,0x13 - ECDHE-RSA-AES128-SHA    TLSv1 Kx=ECDH     Au=RSA  Enc=AES(128)  Mac=SHA1
            0xC0,0x37 - ECDHE-PSK-AES128-CBC-SHA256 TLSv1 Kx=ECDHEPSK Au=PSK  Enc=AES(128)  Mac=SHA256
            0xC0,0x35 - ECDHE-PSK-AES128-CBC-SHA TLSv1 Kx=ECDHEPSK Au=PSK  Enc=AES(128)  Mac=SHA1
    ```

    Del mismo modo, puede probar las cadenas de prioridad para GnuTLS:
    ```bash
    gnutls-cli -l --priority=NONE:+VERS-TLS1.2:+ECDHE-RSA:+AES-128-GCM:+AES-128-CBC:+AEAD:+SHA256:+CURVE-ALL:+COMP-NULL:+SIGN-ALL:+CTYPE-X.509
    ```
    ```
    Cipher suites for NONE:+VERS-TLS1.2:+ECDHE-RSA:+AES-128-GCM:+AES-128-CBC:+AEAD:+SHA256:+CURVE-ALL:+COMP-NULL:+SIGN-ALL:+CTYPE-X.509
    TLS_ECDHE_RSA_AES_128_GCM_SHA256                        0xc0, 0x2f      TLS1.2
    TLS_ECDHE_RSA_AES_128_CBC_SHA256                        0xc0, 0x27      TLS1.2
         
    Protocols: VERS-TLS1.2
    Ciphers: AES-128-GCM, AES-128-CBC
    MACs: AEAD, SHA256
    Key Exchange Algorithms: ECDHE-RSA
    Groups: GROUP-SECP256R1, GROUP-SECP384R1, GROUP-SECP521R1, GROUP-X25519, GROUP-X448, GROUP-FFDHE2048, GROUP-FFDHE3072, GROUP-FFDHE4096, GROUP-FFDHE6144, GROUP-FFDHE8192
    PK-signatures: SIGN-RSA-SHA256, SIGN-RSA-PSS-SHA256, SIGN-RSA-PSS-RSAE-SHA256, SIGN-ECDSA-SHA256, SIGN-ECDSA-SECP256R1-SHA256, SIGN-EdDSA-Ed25519, SIGN-RSA-SHA384, SIGN-RSA-PSS-SHA384, SIGN-RSA-PSS-RSAE-SHA384, SIGN-ECDSA-SHA384, SIGN-ECDSA-SECP384R1-SHA384, SIGN-EdDSA-Ed448, SIGN-RSA-SHA512, SIGN-RSA-PSS-SHA512, SIGN-RSA-PSS-RSAE-SHA512, SIGN-ECDSA-SHA512, SIGN-ECDSA-SECP521R1-SHA512, SIGN-RSA-SHA1, SIGN-ECDSA-SHA1
    ```

#### Cambio de AES128 a AES256
Zabbix utiliza AES128 por defecto para los datos. Supongamos que está utilizando certificados y desea cambiar a AES256, en OpenSSL 1.1.1.

Esto se puede conseguir añadiendo los parámetros respectivos en `zabbix_server.conf`:
```
TLSCAFile=/home/zabbix/ca.crt
TLSCertFile=/home/zabbix/server.crt
TLSKeyFile=/home/zabbix/server.key
TLSCipherCert13=TLS_AES_256_GCM_SHA384
TLSCipherCert=EECDH+aRSA+AES256:-SHA1:-SHA384
TLSCipherPSK13=TLS_CHACHA20_POLY1305_SHA256
TLSCipherPSK=kECDHEPSK+AES256:-SHA1
TLSCipherAll13=TLS_AES_256_GCM_SHA384
TLSCipherAll=EECDH+aRSA+AES256:-SHA1:-SHA384
```

**ATENCIÓN**:

> Aunque sólo se utilizarán los cifrados relacionados con certificados, también se definen los parámetros TLSCipherPSK* para evitar sus valores por defecto, que incluyen cifrados menos seguros para una mayor interoperabilidad. Los cifrados PSK no pueden deshabilitarse completamente en el servidor/proxy.

Y en zabbix_agentd.conf:
```
TLSConnect=cert
TLSAccept=cert
TLSCAFile=/home/zabbix/ca.crt
TLSCertFile=/home/zabbix/agent.crt
TLSKeyFile=/home/zabbix/agent.key
TLSCipherCert13=TLS_AES_256_GCM_SHA384
TLSCipherCert=EECDH+aRSA+AES256:-SHA1:-SHA384
```



         









  






