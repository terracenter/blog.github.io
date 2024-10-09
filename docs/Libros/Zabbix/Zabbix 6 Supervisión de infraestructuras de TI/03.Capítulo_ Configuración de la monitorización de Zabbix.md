# Capítulo 3: Configuración de la monitorización de Zabbix

Zabbix está construido para ser flexible y debe ser capaz de monitorizar casi cualquier cosa que pueda necesitar. En este capítulo, vamos a aprender más acerca de cómo trabajar con Zabbix para construir un montón de diferentes opciones para la monitorización. Vamos a ir sobre ellos receta por receta, por lo que va a terminar con una sólida comprensión de cómo funcionan. Cubriremos las siguiente**s** recetas sobre los diferentes tipos de monitorización:

- Configuración de la monitorización del agente Zabbix
- Trabajando con monitorización SNMP
- Creación de comprobaciones simples de Zabbix y Zabbix trapper
- Trabajo con elementos calculados y dependientes
- Creación de comprobaciones externas
- Configuración de la monitorización JMX
- Configuración de la monitorización de bases de datos
- Configuración de la monitorización del agente HTTP
- Uso del preprocesamiento de Zabbix para modificar los valores de los elementos

## Requisitos técnicos

Necesitaremos un servidor Zabbix capaz de realizar monitorización, con los siguientes requisitos:

- Un servidor con el servidor Zabbix instalado en una distribución Linux de su elección, como CentOS o Ubuntu, pero una distribución como Debian, Rocky Linux, o cualquier otra le vendrá igual de bien
- Un servidor MariaDB para monitorizar

Utilizaré el mismo servidor que usamos en el capítulo anterior, pero cualquier servidor Zabbix puede servir.

## Configuración de la monitorización del agente 2 de Zabbix

A partir del lanzamiento de Zabbix 5, Zabbix también inició oficialmente el soporte para el nuevo agente Zabbix 2. Zabbix agente 2 trae algunas mejoras importantes e incluso está escrito en otro lenguaje de codificación, que es Golang en lugar de C.  En esta receta, vamos a explorar cómo trabajar con Zabbix agente 2 y explorar algunas de las nuevas características introducidas por ella.

### Preparándonos

Para empezar con el agente 2 de Zabbix, todo lo que necesitamos hacer es instalarlo en un host que queramos monitorizar. Asegúrate de que tienes un host vacío basado en Red Hat Enterprise Linux (RHEL) o Ubuntu Linux listo para monitorizar.

### Cómo hacerlo...

Vamos a ver cómo instalar Zabbix agente 2 y luego pasar a trabajar realmente con él.

#### Instalando el agente 2 de Zabbix

Comencemos instalando el agente 2 de Zabbix en el host Linux que queremos monitorizar. Te mostraré cómo hacer los pasos tanto en sistemas RHEL como Ubuntu:

1. Ejecuta el siguiente comando para añadir el repositorio:
   Para sistemas basados en RHEL:

   ```bash
   rpm -Uvh https://repo.zabbix.com/zabbix/6.0/rhel/8/x86_64/zabbix-release-6.0-1.el8.noarch.rpm
   ```

   Para sistemas Ubuntu:

   ```bash
   wget https://repo.zabbix.com/zabbix/6.0/ubuntu/pool/main/z/zabbix-release/zabbix-release_6.0-1+ubuntu20.04_all.deb
   dpkg -i zabbix-release_6.0-1+ubuntu20.04_all.deb
   ```
2. A continuación, ejecute el siguiente comando para instalar el agente 2 de Zabbix:
   Para sistemas basados en RHEL:

   ```bash
   dnf -y install zabbix-agent2
   ```

   Para sistemas Ubuntu:

   ```bash
   apt install zabbix-agent2
   ```

Enhorabuena, el agente 2 de Zabbix ya está instalado y listo para usar.

**Nota importante**
Cuando añadas nuevos repositorios a tu sistema, consulta siempre la página de descargas de Zabbix. Aquí podemos encontrar el repositorio actualizado adecuado para nuestro sistema:
https://www.zabbix.com/download

##### Uso de un agente Zabbix en modo pasivo

Empecemos construyendo un agente Zabbix con comprobaciones pasivas:

1. Después de instalar el agente 2 de Zabbix, vamos a abrir el fichero de configuración del agente Zabbix para editarlo:
   ```bash
   vim /etc/zabbix/zabbix_agent2.conf
   ```

En este archivo, editamos todos los valores de configuración del agente Zabbix que podríamos necesitar desde el lado del servidor.

2. Empecemos por editar los siguientes parámetros

   ```
   Server=127.0.0.1
   Hostname=Zabbix server
   ```
3. Cambie el valor de Server por la IP del servidor Zabbix que monitorizará este agente pasivo. Cambiar el valor de Hostname por el nombre de host del servidor monitorizado. Podemos obtener la dirección IP de nuestro servidor con el siguiente comando:

   ```bash
   ip addr
   ```
4. Ahora reinicie el proceso del agente 2 de Zabbix:

   ```bash
   systemctl enable zabbix-agent2
   systemctl restart zabbix-agent2
   ```
5. A continuación, vaya al frontend de su servidor Zabbix y añada este host para su monitorización.
6. Vaya a **Configuration** | **Hosts** en su frontend Zabbix y haga clic en Crear host en la esquina superior derecha.
7. Para crear este host en nuestro servidor Zabbix, tenemos que rellenar los valores como se ve en la siguiente captura de pantalla
   ![Figura 3.1 - Página de creación de host Zabbix para el host lar-book-agent](https://static.packt-cdn.com/products/9781803246918/graphics/image/B18275_03_001.jpg)
   Figura 3.1 - Página de creación de host Zabbix para el host lar-book-agent

**Es importante añadir lo siguiente**

* **Nombre del Host**: Para identificar este host
* **Grupos**: Para agrupar hosts de forma lógica
* **Interfaces**: Para supervisar este host en una interfaz específica. Sin interfaz no hay comunicación. Es posible tener un host sin interfaz en Zabbix 6, si no lo necesitamos. En el caso de un host monitorizado por un agente Zabbix, necesitamos una interfaz de agente.

8. Asegúrese de añadir la dirección IP correcta a la configuración de la interfaz del agente.
9. También es importante añadir una plantilla a este host, que con Zabbix 6 debe hacerse en la misma pestaña. Como se trata de un servidor Linux monitorizado por un agente Zabbix, vamos a añadir la plantilla out-of-the-box correcta, como se muestra en la siguiente captura de pantalla:
   ![Figura 3.2 - Página de plantilla de host Zabbix para el host lar-book-agent](https://static.packt-cdn.com/products/9781803246918/graphics/image/B18275_03_002.jpg)

   Figura 3.2 - Página de plantilla de host Zabbix para el host lar-book-agent
10. Pulsa el botón azul Añadir y habrás terminado de crear este host agente. Ahora que tienes este host, asegúrate de que el icono ZBX se vuelve verde, indicando que este host está activo y siendo monitorizado por el agente pasivo de Zabbix:
    ![Figura 3.3 - Página de configuración de hosts de Zabbix, host lar-book-agent](https://static.packt-cdn.com/products/9781803246918/graphics/image/B18275_03_003.jpg)
    Figura 3.3 - Página de configuración de hosts de Zabbix, host lar-book-agent
11. Como hemos configurado nuestro host y añadido una plantilla con ítems, ahora podemos ver los valores recibidos en los ítems para este host yendo a **Monitoring** | **Hosts** y marcando el botón **Latest data**. Tenga en cuenta que los valores pueden tardar un poco en aparecer:
    ![Figura 3.4 - Página de Latest data de Zabbix para el host lar-book-agent](https://static.packt-cdn.com/products/9781803246918/graphics/image/B18275_03_004.jpg)
    Figura 3.4 - Página de Latest data de Zabbix para el host lar-book-agent

---

##### Usando un agente Zabbix en modo activo

Ahora vamos a ver cómo configurar el agente Zabbix con comprobaciones activas. Necesitamos cambiar algunos valores en el lado del host del servidor Linux monitorizado:

1. Comience ejecutando el siguiente comando:

   ```bash
   vim /etc/zabbix/zabbix_agent2.conf
   ```
2. Ahora vamos a editar el siguiente valor para cambiar este host a un agente activo:

   ```
   ServerActive=127.0.0.1
   ```
3. Cambia el valor de `serverActive` por la IP del servidor Zabbix que monitorizará este agente pasivo y cambia también el valor de `Hostname` por `lar-book-agent`:

   ```
   Hostname=lar-book-agent
   ```

   **Nota importante**
   Ten en cuenta que si estás trabajando con múltiples servidores Zabbix o proxies Zabbix, por ejemplo cuando estás ejecutando el servidor Zabbix en alta disponibilidad, necesitas rellenar todas las direcciones IP de los servidores Zabbix o proxies en el parámetro **ServerActive**. Los nodos de **Alta Disponibilidad (HA)** están delimitados por un punto y coma (;) y las diferentes IPs del entorno Zabbix por una coma (,).
4. Ahora reinicia el proceso del agente 2 de Zabbix

   ```bash
   systemctl restart zabbix-agent2
   ```
5. A continuación, vaya al frontend de su servidor Zabbix y vamos a añadir otro host con una plantilla para hacer comprobaciones activas en lugar de pasivas.
6. En primer lugar, vamos a cambiar el nombre de nuestro host pasivo. Para ello, vaya a **Configuration** | **Hosts** en su frontend Zabbix y haga clic en el host que acabamos de crear. Cambia el **Host name** como sigue:
   ![Figura 3.5 - Página de configuración del host Zabbix lar-book-agent_passive](https://static.packt-cdn.com/products/9781803246918/graphics/image/B18275_03_005.jpg)
   Figura 3.5 - Página de configuración del host Zabbix lar-book-agent_passive

Hacemos esto porque para un agente Zabbix activo, el nombre de host en el archivo de configuración del agente Zabbix necesita coincidir con la configuración de nuestro host como se ve en el frontend de Zabbix.

7. Haga clic en el botón azul **Update** para guardar los cambios.
8. Vaya a **Configuration** | **Hosts** en su frontend Zabbix y haga clic en **Crear** host en la esquina superior derecha.
9. Ahora vamos a crear el host de la siguiente manera:
   ![Figura 3.6 - Página de configuración de host Zabbix para el host lar-book-agent](https://static.packt-cdn.com/products/9781803246918/graphics/image/B18275_03_006.jpg)
   Figura 3.6 - Página de configuración de host Zabbix para el host lar-book-agent
10. Además, asegúrese de añadir la plantilla correcta, llamada `Linux por el agente de Zabbix activo`:
    ![ Figura 3.7 - Página de plantilla de host Zabbix para el host lar-book-agent ](https://static.packt-cdn.com/products/9781803246918/graphics/image/B18275_03_007.jpg)
    Figura 3.7 - Página de plantilla de host Zabbix para el host lar-book-agent

Tenga en cuenta que el icono ZBX no se pondrá verde para un agente activo. Pero cuando navegamos a **Monitoring** | **Hosts** y comprobamos **Latest data**, podemos ver nuestros datos activos entrando.

**Consejo**
Como se habrá dado cuenta hace un momento, un agente Zabbix puede funcionar en modo pasivo y activo al mismo tiempo. Tenga esto en cuenta cuando cree sus propias plantillas de agente Zabbix, ya que a veces puede querer combinar los tipos de comprobación.

### Cómo funciona...

Ahora que ya hemos configurado nuestros agentes Zabbix y sabemos cómo deben estar configurados, vamos a ver cómo funcionan los diferentes modos.

#### Agente pasivo

El **agente pasivo** funciona recogiendo datos de nuestro host con el agente Zabbix. Cada vez que un elemento de nuestro host alcanza su intervalo de actualización, el servidor Zabbix pregunta al agente Zabbix cuál es el valor actual:
![Figura 3.8 - Diagrama de comunicación entre el servidor y el agente pasivo](https://static.packt-cdn.com/products/9781803246918/graphics/image/B18275_03_008.jpg)
Figura 3.8 - Diagrama de comunicación entre el servidor y el agente pasivo

Los agentes pasivos son geniales cuando se trabaja en entornos en los que se quiere mantener la comunicación iniciada desde el lado del servidor Zabbix o del proxy Zabbix, por ejemplo, cuando hay un cortafuegos que sólo permite el tráfico saliente, visto desde el lado del servidor Zabbix o del proxy.

#### Agente activo

El agente activo funciona enviando datos desde el agente Zabbix a un servidor Zabbix o proxy Zabbix. Cada vez que un elemento de nuestro agente alcance su intervalo de actualización, el agente enviará el valor a nuestro servidor.
![Figura 3.9 - Diagrama de comunicación entre el servidor y el agente activo](https://static.packt-cdn.com/products/9781803246918/graphics/image/B18275_03_009.jpg)
Figura 3.9 - Diagrama de comunicación entre el servidor y el agente activo

El agente activo es genial cuando se trabaja con un entorno en el que hay un cortafuegos que sólo acepta conexiones salientes, como se ve desde el lado del agente de Zabbix. Este es el caso en muchos entornos, y puede mitigar una de las principales preocupaciones de seguridad que se asocia principalmente con la monitorización de hosts.

Por otro lado, el agente Zabbix trabajando en modo activo también puede ser mucho más eficiente. La mayor parte de la carga que viene de obtener datos a su servidor Zabbix está ahora en el lado del agente Zabbix. Debido a que hay más agentes Zabbix que servidores Zabbix o proxies, descargar la carga de esta manera es una gran idea.

Como hemos mencionado, podemos utilizar ambos tipos de comprobaciones al mismo tiempo, dándonos la libertad de configurar cada tipo de comprobación que podamos necesitar. Nuestra configuración se vería así:
![Figura 3.10 - Diagrama de comunicación entre el servidor y los dos tipos de agentes](https://static.packt-cdn.com/products/9781803246918/graphics/image/B18275_03_010.jpg)
Figura 3.10 - Diagrama de comunicación entre el servidor y los dos tipos de agentes

Este podría ser el caso en situaciones en las que queremos monitorizar principalmente de forma pasiva, pero, por ejemplo, la monitorización de ficheros log con el agente Zabbix debe hacerse con un agente Zabbix activo. Entonces podemos combinar nuestros modos y asegurarnos de utilizar la escala completa de nuestras características proporcionadas en el agente Zabbix.

### Ver también

Hay un montón de cosas pasando bajo el capó del agente Zabbix 2; si estás interesado en aprender más sobre el núcleo del agente Zabbix 2, echa un vistazo a esta interesante entrada del blog de Alexey Petrov: https://blog.zabbix.com/magic-of-new-zabbix-agent/8460/

---

## Trabajando con monitorización SNMP

Ahora vamos a hacer algo de lo que más disfruto cuando trabajo con Zabbix: construir monitorización SNMP. Mis raíces profesionales están en la ingeniería de redes, y he trabajado mucho con monitorización SNMP para monitorizar todos estos diferentes dispositivos de red.

### Preparación

Para empezar, necesitamos los dos hosts Linux que utilizamos en las recetas anteriores:

- Nuestro host servidor Zabbix
- El host que usamos en la receta anterior para monitorizar a través del agente activo de Zabbix

### Cómo hacerlo...

Monitorizar mediante SNMP polling es fácil y muy potente. Empezaremos configurando SNMPv3 en nuestro host Linux monitorizado:

1. Empecemos emitiendo los siguientes comandos para instalar SNMP en nuestro host.
   Para sistemas basados en RHEL:

   ```bash
   dnf install net-snmp net-snmp-utils
   ```

   Para sistemas Ubuntu:

   ```bash
   apt install snmp snmpd libsnmp-dev
   ```
2. Ahora, vamos a crear el nuevo usuario SNMPv3 que utilizaremos para monitorizar nuestro host. Tenga en cuenta que vamos a utilizar contraseñas inseguras, pero asegúrese de utilizar contraseñas seguras para sus entornos de producción. Emita el siguiente comando:

   ```bash
   net-snmp-create-v3-user -ro -a my_authpass -x my_privpass -A SHA -X AES snmpv3user
   ```

Esto creará un usuario SNMPv3 con el nombre de usuario `snmpv3user`, la contraseña de autenticación `my_authpass`, y la contraseña de privilegios `my_ privpass`.

3. Asegúrate de editar el archivo de configuración SNMP para permitirnos leer todos los objetos SNMP:

   ```bash
   vim /etc/snmp/snmpd.conf
   ```
4. Añade la siguiente línea en el resto de las líneas de la `view systemview`:

   ```
   view    systemview    included   .1
   ```
5. Ahora inicie y habilite el demonio snmpd para permitirnos empezar a monitorizar este servidor:

   ```bash
   systemctl enable snmpd
   systemctl start snmpd
   ```

Esto es todo lo que necesitamos hacer en el lado del host Linux; ahora podemos ir al frontend de Zabbix para configurar nuestro host. Vaya a **Configuration** | **Hosts** en su frontend Zabbix y haga clic en Crear host en la esquina superior derecha.

6. Ahora rellene la página de configuración del host:
   ![Figura 3.11 - Página de configuración de host Zabbix para el host lar-book-agent_snmp](https://static.packt-cdn.com/products/9781803246918/graphics/image/B18275_03_011.jpg)
   Figura 3.11 - Página de configuración de host Zabbix para el host lar-book-agent_snmp
7. No olvides cambiar la dirección IP de la interfaz SNMP por tu propio valor.
8. Asegúrese de añadir la plantilla out-of-the-box correcta como se muestra en la siguiente captura de pantalla:
   ![Figura 3.12 - Añadir la plantilla Linux SNMP al host](https://static.packt-cdn.com/products/9781803246918/graphics/image/B18275_03_012.jpg)
   Figura 3.12 - Añadir la plantilla Linux SNMP al host

   **Consejo**
   Al actualizar desde una versión anterior de Zabbix a Zabbix 6, no obtendrás todas las nuevas plantillas out-of-the-box. Si crees que te faltan algunas plantillas, puedes descargarlas en el repositorio Git de Zabbix: https://git.zabbix.com/projects/ZBX/repos/zabbix/browse/templates.
9. Estamos utilizando algunas macros en nuestra configuración aquí para el nombre de usuario y contraseña. Podemos utilizar estas macros para añadir un montón de hosts con las mismas credenciales. Esto es muy útil, por ejemplo, si tienes un montón de switches con las mismas credenciales SNMPv3.

   Rellenemos las macros en **Administración** | **General** y utilicemos el desplegable para seleccionar **Macros**. Rellena las macros así:
   ![Figura 3.13 - Página de macros globales de Zabbix con macros SNMP](https://static.packt-cdn.com/products/9781803246918/graphics/image/B18275_03_013.jpg)
   Figura 3.13 - Página de macros globales de Zabbix con macros SNMP<

   **Consejo**
   Una característica interesante en Zabbix 6 es la posibilidad de ocultar macros en el frontend utilizando el tipo de macro Texto secreto. Hay que tener en cuenta que las macros de tipo Secret text siguen sin estar encriptadas en la base de datos de Zabbix, y para macros totalmente encriptadas necesitaríamos algo como HashiCorp Vault. Consulta la documentación para más información: https://www.zabbix.com/documentation/current/en/manual/config/secrets.
10. Usa el desplegable para cambiar `{$SNMPV3_AUTH}` y `{$SNMPV3_PRIV}` a Secret text:
    ![Figura 3.14 - Texto secreto de Zabbix utilizado para ocultar datos sensibles (autenticación)](https://static.packt-cdn.com/products/9781803246918/graphics/image/B18275_03_014.jpg)
    Figura 3.14 - Texto secreto de Zabbix utilizado para ocultar datos sensibles (autenticación)
11. Ahora, después de aplicar estos cambios haciendo clic en Update, deberíamos ser capaces de monitorizar nuestro servidor Linux a través de SNMPv3. Vayamos a **Monitoring** | **Hosts** y comprobemos la página **Latest data** de nuestro nuevo host:
    ![Figura 3.15 - SNMP Lasta data para el host lar-book-agent_snmp](https://static.packt-cdn.com/products/9781803246918/graphics/image/B18275_03_015.jpg)
    Figura 3.15 - SNMP Lasta data para el host lar-book-agent_snmp

    Tenga en cuenta que sus datos pueden tardar algún tiempo en aparecer aquí.

    **Consejo**

    Cuando se trabaja con macros, hay tres niveles en orden de cascada: **Macros de nivel global**, **de plantilla** y **de host**. Cuando trabaje con macros de nivel Global, tenga en cuenta que no se exportan con plantillas o hosts. En la mayoría de los casos, es mejor utilizar el nivel de plantilla y el nivel de host, para mantener sus exportaciones independientes de la configuración global de Zabbix.

## Cómo funciona...

Cuando creamos un host como hicimos en el paso 4, Zabbix sondea el host usando SNMP. El sondeo SNMP funciona con OIDs SNMP. Por ejemplo, cuando sondeamos el elemento llamado Memoria libre, le pedimos al agente SNMP que se ejecuta en nuestro host Linux que nos proporcione el valor de **1.3.6.1.4.1.2021.4.6.0.** Ese valor es devuelto a nosotros en el host. Ese valor es entonces devuelto a nosotros en el servidor Zabbix:
![Figura 3.16 - Diagrama que muestra la comunicación entre el servidor Zabbix y el host SNMP](https://static.packt-cdn.com/products/9781803246918/graphics/image/B18275_03_016.jpg)
Figura 3.16 - Diagrama que muestra la comunicación entre el servidor Zabbix y el host SNMP

SNMPv3 añade autenticación y encriptación a este proceso, asegurando que cuando nuestro servidor Zabbix solicita información, esa solicitud es primero encriptada y los datos son enviados de vuelta encriptados también.

También incluimos la opción de utilizar **solicitudes masivas** al configurar nuestro host. Las peticiones masivas solicitan varios OIDs en el mismo flujo, por lo que este es el método preferido para hacer peticiones SNMP ya que es más eficiente. Sólo desactívalo para hosts que no soporten peticiones masivas.

Por último, echemos un vistazo a los OIDs SNMP, la parte más importante de nuestra petición SNMP. Los OIDs funcionan en una estructura de árbol, lo que significa que cada número detrás del punto puede contener otro valor. Por ejemplo, veamos este OID para nuestro host:

```bash
1.3.6.1.4.1.2021.4 = UCD-SNMP-MIB::memory
```

Si sondeamos ese OID con la herramienta SnmpWalk CLI o con nuestro servidor Zabbix, obtendremos varios OID de vuelta:

```bash
.1.3.6.1.4.1.2021.4.1.0 = INTEGER: 0
.1.3.6.1.4.1.2021.4.2.0 = STRING: swap
.1.3.6.1.4.1.2021.4.3.0 = INTEGER: 1679356 kB
.1.3.6.1.4.1.2021.4.4.0 = INTEGER: 1674464 kB
.1.3.6.1.4.1.2021.4.5.0 = INTEGER: 1872872 kB
.1.3.6.1.4.1.2021.4.6.0 = INTEGER: 184068 kB

```

Eso incluye nuestro OID `1.3.6.1.4.1.2021.4.6.0` con el valor que contiene nuestra memoria libre. Así es como se construye SNMP, como un árbol.

---

## Creación de comprobaciones sencillas de Zabbix y el trap de Zabbix

En esta receta, vamos a repasar dos comprobaciones que pueden ayudarte a construir algunas configuraciones más personalizadas. Las comprobaciones simples de Zabbix te proporcionan una forma sencilla de monitorizar algunos datos específicos. El Zabbix trapper se combina con el Zabbix sender para obtener datos de tus hosts en el servidor, permitiendo algunas opciones de scripting. Vamos a empezar.

### Preparación

Para crear estas comprobaciones, necesitaremos un servidor Zabbix y un host Linux para monitorizar. Podemos utilizar el host con un agente Zabbix y monitorización SNMP de las recetas anteriores.

Ten en cuenta que para estas comprobaciones no necesitamos el agente Zabbix.

### Cómo hacerlo...

Trabajar con cheques simples es bastante sencillo, como su nombre indica, así que empecemos.

#### Creación de comprobaciones simples

Vamos a crear una comprobación simple para monitorizar si un servicio está ejecutándose y aceptando conexiones TCP en un puerto determinado:

1. Para ello, tendremos que crear un nuevo host en el frontend de Zabbix. Vaya a **Configuration** | **Hosts** en su interfaz Zabbix y haga clic en Crear host en la esquina superior derecha.
2. Cree un host con la siguiente configuración:
   ![Figura 3.17 - Página de configuración de host Zabbix para el host lar-book-agent_simple](https://static.packt-cdn.com/products/9781803246918/graphics/image/B18275_03_017.jpg)
   Figura 3.17 - Página de configuración de host Zabbix para el host lar-book-agent_simple
3. Ahora vaya a **Configuration** | **Hosts**, haga clic en el host recién creado, y vaya a **Items**. Queremos crear un nuevo elemento aquí haciendo clic en el botón Crear elemento.
   Vamos a crear un nuevo elemento con los siguientes valores, y después de hacerlo, haremos clic en el botón Añadir en la parte inferior de la página:
   ![Figura 3.18 - Página de configuración de elementos de Zabbix para la comprobación del puerto 22 en el host lar-book-agent_simple](https://static.packt-cdn.com/products/9781803246918/graphics/image/B18275_03_018.jpg)
   Figura 3.18 - Página de configuración de elementos de Zabbix para la comprobación del puerto 22 en el host lar-book-agent_simple
4. Asegúrate de añadir también una etiqueta al ítem, ya que la necesitamos en varios lugares para filtrar y encontrar nuestro ítem cuando trabajemos con Zabbix. Configúralo así:
   ![Figura 3.19 - Puerto SSH de Zabbix, pestaña Etiqueta](https://static.packt-cdn.com/products/9781803246918/graphics/image/B18275_03_019.jpg)
   Figura 3.19 - Puerto SSH de Zabbix, pestaña Etiqueta

   **Nota importante**
   Aquí estamos añadiendo el elemento clave `net.tcp.services[ssh,,22]`. El puerto en este caso es opcional, ya que podemos especificar el servicio SSH con un puerto diferente si queremos.
5. Ahora deberíamos ser capaces de ver si nuestro servidor está aceptando conexiones SSH en el puerto 22 en nuestra pantalla de Latest data. Navegue a **Monitoring** | **Hosts** y compruebe en la pantalla **Latest data** nuestro nuevo valor:
   ![Figura 3.20 - Zabbix Última página de datos para el host lar-book-agent_simple, elemento puerto 22 comprobado](https://static.packt-cdn.com/products/9781803246918/graphics/image/B18275_03_020.jpg)
   Figura 3.20 - Zabbix Última página de datos para el host lar-book-agent_simple, elemento puerto 22 comprobado
6. Hay otra cosa mal aquí. Como puede ver, actualmente no tenemos una configuración de mapeo de valores. El último valor sólo muestra un 1 o un 0. Para cambiar esto, vuelva a **Configuration** | **Hosts** y edite el host `lar-book-agent_simple`.
7. Haga clic en la ficha Asignación de valores y haga clic en el pequeño botón Agregar para agregar una asignación de valores. Cree lo siguiente:
   ![Figura 3.21 - Host lar-book-agent_simple, ventana de asignación de valores](https://static.packt-cdn.com/products/9781803246918/graphics/image/B18275_03_021.jpg)
   Figura 3.21 - Host lar-book-agent_simple, ventana de asignación de valores
8. Pulse el botón azul Añadir y pulse el botón azul Update.
9. A continuación, de vuelta a la lista completa de **Configuration** | **Hosts**, navegue hasta nuestro `host lar-book-agent_simple` y pulse sobre Elementos para este host.
10. Edite el elemento Comprobar si el puerto 22 está disponible y añada la asignación de valores como se indica a continuación:
    ![Figura 3.22 - Host lar-book-agent_simple, ventana de edición de elementos](https://static.packt-cdn.com/products/9781803246918/graphics/image/B18275_03_022.jpg)
    Figura 3.22 - Host lar-book-agent_simple, ventana de edición de elementos

    Eso es todo lo que hay que hacer para crear tus comprobaciones simples en Zabbix. La última página de datos tendrá ahora este aspecto:
    ![Figura 3.23 - Última página de datos de nuestro elemento de comprobación del puerto 22](https://static.packt-cdn.com/products/9781803246918/graphics/image/B18275_03_023.jpg)
    Figura 3.23 - Última página de datos de nuestro elemento de comprobación del puerto 22

Como puede ver, hay un valor legible por humanos que ahora muestra Arriba o Abajo, lo que nos da una entrada legible por humanos, que es más fácil de entender. Ahora pasemos al elemento trap de Zabbix.

#### Creación de un trapper

Podemos hacer algunas cosas interesantes con los elementos trapper de Zabbix una vez que tengamos configuraciones más avanzadas. Pero por ahora, vamos a crear un elemento en nuestro host `lar-book-agent_simple`:

1. Ve a **Configuration** | **Hosts** y haz click en el host, luego ve a **Items**. Queremos crear un nuevo elemento aquí de nuevo haciendo clic en el botón Crear elemento.

   Ahora vamos a crear el siguiente elemento y haga clic en el botón **Create Item**:
   ![Figura 3.24 - Pantalla de configuración del receptor de trampas de elementos de Zabbix para el host lar-book-agent_simple](https://static.packt-cdn.com/products/9781803246918/graphics/image/B18275_03_024.jpg)
   Figura 3.24 - Pantalla de configuración del receptor de trampas de elementos de Zabbix para el host lar-book-agent_simple
2. Asegúrese de navegar también a la pestaña Tags y añadir una etiqueta que utilizaremos más tarde para filtrar:
   ![Figura 3.25 - Pantalla de configuración de etiquetas receptoras de trampas de elementos de Zabbix para el host lar-book-agent_simple](https://static.packt-cdn.com/products/9781803246918/graphics/image/B18275_03_025.jpg)
   Figura 3.25 - Pantalla de configuración de etiquetas receptoras de trampas de elementos de Zabbix para el host lar-book-agent_simple
3. Si vamos a la CLI de nuestro servidor monitorizado, ahora podemos ejecutar lo siguiente para instalar Zabbix sender:
   **RHEL-based systems:**

   ```bash
   dnf -y install zabbix-sender
   ```

   **Ubuntu systems:**

   ```bash
   apt install zabbix-sender
   ```
4. Después de la instalación, podemos utilizar Zabbix sender para enviar alguna información a nuestro servidor:

   ```bash
   zabbix_sender -z 10.16.16.152 -s "lar-book-agent_simple" -k trap -o "Let's test this book trapper"
   ```

   Ahora deberíamos ser capaces de ver si nuestro host monitorizado ha enviado el trap de Zabbix y el servidor de Zabbix ha recibido este trap para su procesamiento.
5. Navegue a **Monitoring** | **Hosts** y compruebe en la pantalla Latest data nuestro nuevo valor:
   ![Figura 3.26 - Última página de datos de Zabbix para el host lar-book-agent_simple, elemento receptor de trampas](https://static.packt-cdn.com/products/9781803246918/graphics/image/B18275_03_026.jpg)
   Figura 3.26 - Última página de datos de Zabbix para el host lar-book-agent_simple, elemento receptor de trampas

   Ahí está, nuestro tramper Zabbix en nuestro frontend Zabbix.

### Cómo funciona...

Ahora que hemos construido nuestros nuevos elementos, vamos a ver cómo funcionan sumergiéndonos en el lado teórico de las comprobaciones simples y las trampas de Zabbix.

#### Comprobaciones simples

Las comprobaciones simples de Zabbix son básicamente una lista de comprobaciones incorporadas hechas para monitorizar ciertos valores. Hay una lista y descripción disponible en la documentación de Zabbix para todas las comprobaciones simples que están disponibles aquí: https://www.zabbix.com/documentation/current/manual/config/items/itemtypes/simple_checks.

Todas estas comprobaciones son realizadas por el servidor Zabbix para recoger datos de un host monitorizado. Por ejemplo, cuando hacemos la comprobación simple de Zabbix para comprobar si un puerto está abierto, nuestro servidor Zabbix pregunta si puede alcanzar ese puerto y lo convierte en un estado que podemos ver en nuestro frontend Zabbix.

Esto significa que si el firewall de su host monitorizado está bloqueando el puerto 22 desde el servidor Zabbix, obtendremos un valor de servicio está caído. Sin embargo, esto no significa necesariamente que SSH no se esté ejecutando en el servidor; simplemente significa que SSH está caído visto desde el lado del servidor Zabbix o proxy.
![Figura 3.27 - Diagrama de comunicación servidor-host de Zabbix](https://static.packt-cdn.com/products/9781803246918/graphics/image/B18275_03_027.jpg)
Figura 3.27 - Diagrama de comunicación servidor-host de Zabbix

**Sugerencia**
Tenga en cuenta que trabajar con comprobaciones simples depende de factores externos como la configuración del cortafuegos en el host monitorizado. Cuando construya sus propias comprobaciones simples, asegúrese de comprobar también estos factores.

**Trappers**
Cuando trabajamos con Zabbix zender, estamos haciendo exactamente lo contrario de la mayoría de los controles. Estamos construyendo un elemento en nuestro servidor Zabbix, que nos permite capturar elementos trapper. Esto nos permite construir algunos chequeos personalizados para enviar datos a nuestro servidor Zabbix desde un host monitorizado:

![Figura 3.28 - Diagrama del receptor de trampas del servidor Zabbix](https://static.packt-cdn.com/products/9781803246918/graphics/image/B18275_03_028.jpg)
Figura 3.28 - Diagrama del receptor de trampas del servidor Zabbix

Digamos, por ejemplo, que usted quiere construir un script de Python personalizado que, al final de la ejecución de los scripts, envía la salida al servidor Zabbix. Podríamos pedirle a Python que envíe estos datos utilizando la utilidad de Zabbix zender, y entonces usted tendría estos datos disponibles para ser procesados en el servidor Zabbix.

Por ejemplo, es utilizado por algunas empresas que escriben su propio software para integrar completamente su software en Zabbix. Como puede ver, podemos ampliar enormemente nuestras opciones con Zabbix trapper y personalizar aún más nuestro servidor Zabbix.

---

## Trabajar con items calculados y dependientes

Los elementos calculados y dependientes se utilizan en Zabbix para producir valores adicionales a partir de valores existentes. A veces, ya hemos recogido un valor y necesitamos hacer algo más con los valores creados por ese elemento. Podemos hacer exactamente eso utilizando elementos calculados y dependientes.

### Preparándonos

Para trabajar con elementos calculados y elementos dependientes, vamos a necesitar el servidor Zabbix y los hosts monitorizados de las recetas anteriores. Añadiremos los ítems en el host `lar-book-agent_passive` y en nuestro host servidor Zabbix (o cualquier servidor MySQL), así que ya tenemos algunos ítems disponibles para calcular y hacer dependientes.

### Cómo hacerlo...

Veamos cómo podemos ampliar nuestros items empezando por los items calculados.

### Trabajando con ítems calculados

1. Naveguemos hasta la configuración de nuestro host yendo a **Configuration** | **Hosts** y pulsando sobre los **ítems** de nuestro host `lar-book-agent_passive`. En el campo de filtro llamado Nombre, introduzca `memory` y obtendrá la siguiente salida:
   ![Figura 3.29 - Página de elementos de Zabbix para el host lar-book-agent_passive](https://static.packt-cdn.com/products/9781803246918/graphics/image/B18275_03_029.jpg)
   Figura 3.29 - Página de elementos de Zabbix para el host lar-book-agent_passive
2. Lo que vamos a hacer ahora es crear un item calculado que nos mostrará la utilización media de memoria durante un periodo de 15 minutos. Podemos usar este valor para determinar cuán ocupado estuvo nuestro host durante ese período, sin tener que mirar los gráficos.
3. Hagamos clic en el botón Create item y comencemos a crear nuestro nuevo item calculado. Queremos que nuestro elemento tenga los siguientes valores:

![Figura 3.30 - Página de configuración de elementos de Zabbix, memoria media utilizada](https://static.packt-cdn.com/products/9781803246918/graphics/image/B18275_03_030.jpg)
Figura 3.30 - Página de configuración de elementos de Zabbix, memoria media utilizada

4. Asegúrese de navegar también a la pestaña Etiquetas y añadir una etiqueta que utilizaremos más tarde para filtrar:
   ![Figura 3.31 - Pestaña Etiquetas de elementos calculados](https://static.packt-cdn.com/products/9781803246918/graphics/image/B18275_03_031.jpg)
   Figura 3.31 - Pestaña Etiquetas de elementos calculados
5. Ahora si vamos a nuestra página **Monitoring** | **Hosts** y seleccionamos **Latest data**, podemos comprobar nuestro valor. Asegúrese de filtrar en el campo Nombre para la **memory**, para que veamos los valores correctos:
   ![Figura 3.32 - Última página de datos de Zabbix para el host lar-book-agent_passive, elementos de memoria](https://static.packt-cdn.com/products/9781803246918/graphics/image/B18275_03_032.jpg)
   Figura 3.32 - Última página de datos de Zabbix para el host lar-book-agent_passive, elementos de memoria

   Ahora podemos ver claramente que estamos calculando la media de 15 minutos de la utilización de memoria en nuestro elemento recién creado.

---

#### Trabajar con elementos dependientes

Es hora de hacer nuestro primer elemento dependiente. Usaré el host `lar-book-agent_centos` o nuestro (como se llama por defecto) servidor Zabbix, pero cualquier servidor de base de datos MySQL debería funcionar. Digamos que queremos solicitar algunas variables de nuestra base de datos MySQL en un gran lote. A continuación, podemos crear elementos dependientes en la parte superior del primer elemento para seguir procesando los datos:

1. Empecemos creando el control principal. Navegue a **Configuration**| **Hosts**, seleccione nuestro host, y luego hagamos clic en el botón **Create item** para empezar a crear nuestro primer elemento nuevo. Queremos un ítem con las siguientes variables:
   ![Figura 3.33 - Página de configuración de elementos de Zabbix, estado de la base de datos](https://static.packt-cdn.com/products/9781803246918/graphics/image/B18275_03_033.jpg)
   Figura 3.33 - Página de configuración de elementos de Zabbix, estado de la base de datos

   Ahora, este elemento es una comprobación SSH que inicia sesión en nuestro servidor Zabbix utilizando SSH y luego ejecuta el código introducido en el campo Script ejecutado. Este código iniciará sesión en nuestra base de datos MariaDB e imprimirá el estado. Asegúrese de introducir sus credenciales correctamente.

   **Sugerencia**

   En lugar de usar credenciales de texto plano en el comando MySQL, lo cual no es recomendable. Utilice macros en su campo Script ejecutado. De esta forma, puede utilizar el tipo de macro Texto secreto para asegurarse de que nadie pueda leer su contraseña desde el frontend.

Antes de guardar este nuevo elemento, asegúrese de añadir también una etiqueta como esta:

2. Antes de guardar este nuevo elemento, asegúrate de añadir también una etiqueta como ésta:
   ![Figura 3.34 - Página de configuración de elementos maestros de Zabbix, pestaña Etiquetas](https://static.packt-cdn.com/products/9781803246918/graphics/image/B18275_03_034.jpg)
   Figura 3.34 - Página de configuración de elementos maestros de Zabbix, pestaña Etiquetas
3. Ahora haga clic en el botón azul **Add **para guardar este nuevo elemento.
4. De vuelta en la lista de ítems, hagamos click en el nombre de host de este host y luego en Macros. Cree una nueva macro `{$USERNAME}` y `{$PASSWORD}` con su nombre de usuario y contraseña SSH como valor.
5. A continuación, vaya a ****Monitoring**** | **Latest data** y compruebe los datos de nuestra nueva comprobación. Debería haber una larga lista de valores MariaDB. Si es así, podemos continuar con nuestro siguiente paso de crear el elemento dependiente.
6. Para crear el elemento dependiente, navegue a **Configuration** | **Hosts**, seleccione nuestro host, y hagamos clic en el botón **Create item**. Queremos que este elemento obtenga las siguientes variables:
   ![Figura 3.35 - Página de configuración de elementos de Zabbix, clientes MariaDB abortados](https://static.packt-cdn.com/products/9781803246918/graphics/image/B18275_03_035.jpg)
   Figura 3.35 - Página de configuración de elementos de Zabbix, clientes MariaDB abortados
7. Asegúrese de añadir la etiqueta, como sigue:
   ![Figura 3.36 - Página de configuración de elementos dependientes de Zabbix, pestaña Etiquetas](https://static.packt-cdn.com/products/9781803246918/graphics/image/B18275_03_036.jpg)
   Figura 3.36 - Página de configuración de elementos dependientes de Zabbix, pestaña Etiquetas
8. Es muy importante añadir preprocesamiento a este elemento también, de lo contrario, simplemente obtendremos los mismos datos que nuestro elemento maestro. Por lo tanto, vamos a añadir lo siguiente:
   ![Figura 3.37 - Página de preprocesamiento de elementos de Zabbix, clientes MariaDB abortados](https://static.packt-cdn.com/products/9781803246918/graphics/image/B18275_03_037.jpg)
   Figura 3.37 - Página de preprocesamiento de elementos de Zabbix, clientes MariaDB abortados

   Con el preprocesamiento añadido, el resultado será el número de clientes abortados para nuestra instancia MariaDB:
   ![Figura 3.38 - Página de últimos datos de Zabbix, clientes MariaDB abortados](https://static.packt-cdn.com/products/9781803246918/graphics/image/B18275_03_038.jpg)
   Figura 3.38 - Página de últimos datos de Zabbix, clientes MariaDB abortados

   Como puede ver, utilizando un elemento dependiente, podemos utilizar información ya disponible de otros elementos de Zabbix y dividirlos en elementos dependientes.

### Cómo funciona...

Los tipos que hemos tratado en esta sección Cómo hacerlo... pueden ser bastante complicados; repasemos cómo funcionan realmente los elementos.

**Elementos calculados**
Trabajar con elementos calculados puede ser una excelente manera de obtener aún más estadísticas de los datos existentes. A veces sólo necesita combinar varios elementos en un valor específico.

Lo que acabamos de hacer funciona tomando varios valores de 1 elemento en un periodo de 15 minutos y calculando la media, como se indica a continuación:
![Figura 3.39 - Diagrama de elementos dependientes de Zabbix](https://static.packt-cdn.com/products/9781803246918/graphics/image/B18275_03_039.jpg)
Figura 3.39 - Diagrama de elementos dependientes de Zabbix

Tomamos esos valores y calculamos la media cada 15 minutos. Nos da una buena indicación de lo que estamos haciendo en un periodo de tiempo determinado.

**Elementos dependientes**
Los elementos dependientes funcionan simplemente tomando los datos de un elemento maestro y procesando esos datos en otros datos. De este modo, podemos estructurar nuestros datos y mantener el mismo intervalo de comprobación para todos estos elementos:
![Figura 3.40 - Diagrama de elementos dependientes](https://static.packt-cdn.com/products/9781803246918/graphics/image/B18275_03_040.jpg)

Figura 3.40 - Diagrama de elementos dependientes

Como se ve aquí, los elementos dependientes funcionan básicamente como duplicadores, con opciones adicionales de preprocesamiento. Tenga en cuenta que el preprocesamiento debe utilizarse para extraer datos del elemento maestro. Sin preprocesamiento, nuestros datos serán exactamente los mismos que los del elemento maestro.

**Sugerencia**

A veces **no necesitamos** que nuestro elemento maestro se guarde en nuestra base de datos; ya tenemos la información en nuestros elementos dependientes. Cuando no queremos que se guarde el elemento maestro, simplemente seleccionamos la opción **Do not keep history** en ese elemento maestro.

---

## Creación de comprobaciones externas

Para ampliar aún más la funcionalidad de Zabbix, podemos utilizar nuestros propios scripts personalizados, que son utilizados por los chequeos externos de Zabbix. No todo lo que queremos monitorizar será siempre estándar en Zabbix, aunque mucho lo es. Siempre hay algo que podría faltar, y las comprobaciones externas son sólo una forma de evitar algunas de ellas.

### Preparación

Para esta receta, vamos a necesitar sólo nuestro servidor Zabbix. Podemos crear un elemento en nuestro `host lar-book-centos`, que es nuestro host monitorizado del servidor Zabbix.

### Cómo hacerlo...

1. En primer lugar, vamos a asegurarnos de que la configuración de nuestro servidor Zabbix está establecida correctamente. Ejecute lo siguiente en la CLI del servidor Zabbix:
   `cat /etc/zabbix/zabbix_server.conf | grep ExternalScripts=`
2. Esto debería mostrarnos la ruta donde colocaremos el script que utiliza el chequeo externo de Zabbix. Por defecto, es `/usr/lib/zabbix/externalscripts/`, y crearemos un nuevo script llamado `test_external` en esta carpeta con el siguiente comando:

```bash
vim /usr/lib/zabbix/externalscripts/test_external
```

Añade el siguiente código a este archivo y guárdalo:

```bash
#!/bin/bash
echo $1
```

3. Asegúrese de que nuestro servidor Zabbix puede ejecutar el script añadiendo los permisos adecuados al archivo. El usuario zabbix en su servidor Linux debe ser capaz de acceder y ejecutar el archivo:
   chmod +x /usr/lib/zabbix/externalscripts/test_external

```bash
chown zabbix:zabbix /usr/lib/zabbix/externalscripts/test_external
```

4. Ahora estamos listos para ir a nuestro host para crear un nuevo elemento. Vaya a **Configuration** | **Hosts**, seleccione nuestro host, **lar-book-centos**, y haga clic en el botón **Create item**. Queremos que este item obtenga las siguientes variables:
   ![Figura 3.41 - Página de configuración de elementos de Zabbix](https://static.packt-cdn.com/products/9781803246918/graphics/image/B18275_03_041.jpg)
   Figura 3.41 - Página de configuración de elementos de Zabbix
5. Después de añadir este nuevo elemento, naveguemos a **Monitoring** | **Hosts** y comprobemos la página **Latest data** para nuestro host. Deberíamos obtener nuestra variable Test devuelta por nuestro script como Último valor en Zabbix, como se muestra en la siguiente captura de pantalla:

![Figura 3.42 - Página de últimos datos de Zabbix](https://static.packt-cdn.com/products/9781803246918/graphics/image/B18275_03_042.jpg)
Figura 3.42 - Página de últimos datos de Zabbix

**Sugerencia**
Utilice las macros del frontend como variables para enviar datos desde su frontend a sus scripts. Puede automatizar aún más sus comprobaciones con esto para mejorar sus comprobaciones externas.

### Cómo funciona...

Las comprobaciones externas parecen tener una curva de aprendizaje empinada, pero en realidad son bastante simples desde el lado de Zabbix. Todo lo que hacemos es enviar un comando a un script externo y esperar un resultado:
![Figura 3.43 - Diagrama de comunicación del script externo del servidor Zabbix](https://static.packt-cdn.com/products/9781803246918/graphics/image/B18275_03_043.jpg)
Figura 3.43 - Diagrama de comunicación del script externo del servidor Zabbix

Como en nuestro ejemplo, enviamos el valor Test a nuestro script, el cual, a su vez, nos lo devuelve como $1.

Cuando se tiene un buen conocimiento de un lenguaje de programación como Python, se puede utilizar esta función para construir muchas más expansiones sobre el actual conjunto de características existentes de Zabbix - una herramienta simple pero poderosa para trabajar.

---

## Configuración de la monitorización JMX

Zabbix incorpora la monitorización JMX para que podamos monitorizar nuestras aplicaciones Java. En esta receta, vamos a ver cómo supervisar Apache Tomcat con Zabbix JMX para que podamos tener una idea de lo que esta opción de monitoreo se trata.

### Preparándonos

Para prepararnos para esta receta, vamos a necesitar nuestro servidor Zabbix para monitorizar nuestra aplicación JMX.

He utilizado una máquina CentOS 7 para el ejemplo de esta receta, con Tomcat instalado. Puede ser bastante complicado utilizar Tomcat en versiones posteriores de CentOS debido a las dependencias de paquetes, por lo que recomiendo seguir con CentOS 7 para el ejemplo. Puedes añadir lo siguiente a tu configuración de Tomcat después de la instalación para que funcione en nuestra receta:

```bash
JAVA_OPTS="-Djava.rmi.server.hostname=10.16.16.155 
dcom.sun.management.jmxremote 
dcom.sun.management.jmxremote.port=12345
dcom.sun.management.jmxremote.authenticate=false 
dcom.sun.management.jmxremote.ssl=false"
```

Si desea configurar la monitorización JMX en su entorno de producción, puede utilizar la configuración que probablemente ya haya configurado allí. Basta con cambiar el puerto y la dirección IP en consecuencia.

### Cómo hacerlo...

Para configurar la monitorización JMX, vamos a añadir un host a nuestro servidor Zabbix que monitorizará nuestra instalación Apache Tomcat. Pero primero, tendremos que añadir algunos ajustes a nuestro archivo` /etc/zabbix/zabbix_server.conf`:

1. Vamos a editar el archivo `zabbix_server.conf` iniciando sesión en nuestro servidor Zabbix y ejecutando el siguiente comando:

   ```bash
   vim /etc/zabbix/zabbix_server.conf
   ```
2. A continuación, tendremos que añadir las siguientes líneas a este archivo:

   ```bash
   JavaGateway=127.0.0.1
   StartJavaPollers=5
   ```

   **Consejo**

   Es posible instalar su pasarela Java en un host separado de su Zabbix. De esta manera podemos repartir la carga y escalar más fácilmente. Simplemente instálelo en un host separado y añada la dirección IP de ese host al parámetro JavaGateway. No haremos esto en el ejemplo, sino que mantendremos la configuración de la pasarela Java en el propio host del servidor Zabbix.
3. También tendremos que instalar la aplicación zabbix-java-gateway en nuestro servidor Zabbix con el siguiente comando.

Sistemas basados en RHEL:

```bash
dnf install zabbix-java-gateway
systemctl enable zabbix-java-gateway
systemctl start zabbix-java-gateway
systemctl restart zabbix-server
```

Sistemas Ubuntu:

```bash
apt install zabbix-java-gateway
systemctl enable zabbix-java-gateway
systemctl start zabbix-java-gateway
systemctl restart zabbix-server
```

Eso es todo lo que tenemos que hacer en el lado del servidor de las cosas para obtener JMX monitoreo para trabajar. Zabbix no incluye esta configuración por defecto, por lo que necesitamos añadir el texto a nuestro archivo e instalar la aplicación.

4. Para empezar con la monitorización de nuestro host JMX, vaya a Configuración | Hosts en su frontend Zabbix y haga clic en Crear host en la esquina superior derecha.

A continuación, vamos a añadir un host con la siguiente configuración:

![Figura 3.44 - Página de configuración de elementos de Zabbix](https://static.packt-cdn.com/products/9781803246918/graphics/image/B18275_03_044.jpg)
Figura 3.44 - Página de configuración de elementos de Zabbix

5. Después de esto, nuestro icono JMX debería volverse verde; comprobémoslo en **Monitoring** | **Hosts**. Debería verse así:
   ![Figura 3.45 - Página Zabbix Monitoring | Hosts](https://static.packt-cdn.com/products/9781803246918/graphics/image/B18275_03_045.jpg)
   Figura 3.45 - Página Zabbix Monitoring | Hosts
6. Si pulsamos sobre **Latest data** para nuestro nuevo host monitorizado por **JMX**, deberíamos ver también nuestros datos entrantes. Compruébalo; debería devolver estadísticas como estas:

![Figura 3.46 - Página de últimos datos de Zabbix](https://static.packt-cdn.com/products/9781803246918/graphics/image/B18275_03_046.jpg)
Figura 3.46 - Página de últimos datos de Zabbix

### Cómo funciona...

Zabbix utiliza una pasarela Java alojada en el propio servidor Zabbix o en otro servidor (proxy) para monitorizar aplicaciones JMX:

![Figura 3.47 - Diagrama de comunicación entre el servidor Zabbix y Java](https://static.packt-cdn.com/products/9781803246918/graphics/image/B18275_03_047.jpg)
Figura 3.47 - Diagrama de comunicación entre el servidor Zabbix y Java

Zabbix sondea la pasarela Java y ésta, a su vez, se comunica con nuestra aplicación JMX, como lo hace con Tomcat en nuestro ejemplo. Los datos a su vez son devueltos a través de la misma ruta y podemos ver nuestros datos en nuestro servidor Zabbix.

### Ver también

Hay un montón de aplicaciones que pueden ser monitorizadas a través de Zabbix JMX. Echa un vistazo a la página de monitorización e integraciones de Zabbix para más usos de la monitorización JMX de Zabbix: https://www.zabbix.com/integrations/jmx.

---

## Configuración de la supervisión de bases de datos Configuración de la supervisión de bases de datos

Las bases de datos son un agujero negro para muchos ingenieros: se escriben datos en ellas y se hace algo con ellos. ¿Pero qué pasa si quieres saber más sobre la salud de tu base de datos? Ahí es donde entra en juego la monitorización de bases de datos de Zabbix: podemos utilizarla para monitorizar la salud de nuestra base de datos.

### Preparándonos

Vamos a monitorizar nuestra base de datos Zabbix, por comodidad. Esto significa que todo lo que vamos a necesitar es nuestro servidor Zabbix instalado con nuestra base de datos en él. Vamos a utilizar MariaDB en este ejemplo, así que si usted tiene una configuración PostgreSQL, asegúrese de instalar una instancia MariaDB en un host Linux.

### Cómo hacerlo...

Antes de empezar con la configuración del elemento, tendremos que hacer algunas cosas en el lado CLI del servidor:

1. Empecemos por instalar los módulos necesarios en nuestro servidor.

   Sistemas basados en **RHEL**:

   ```bash
   dnf install unixODBC mariadb-connector-odbc
   ```

   Sistemas **Ubuntu**:

   ```bash
   apt install odbc-mariadb unixodbc unixodbc-dev odbcinst
   ```
2. Ahora vamos a comprobar si existen nuestros archivos de configuración de Open Database Connectivity (ODBC):

   ```bash
   odbcinst -j   
   ```

   El resultado debería ser el siguiente:

   ```
   unixODBC 2.3.7
   DRIVERS............: /etc/odbcinst.ini
   SYSTEM DATA SOURCES: /etc/odbc.ini
   FILE DATA SOURCES..: /etc/ODBCDataSources
   USER DATA SOURCES..: /root/.odbc.ini
   SQLULEN Size.......: 8
   SQLLEN Size........: 8
   SQLSETPOSIROW Size.: 8
   ```
3. Si la salida es correcta, podemos ir a la CLI de Linux y continuar editando odbc.ini para conectarnos a nuestra base de datos:

   ```bash
   vim /etc/odbc.ini
   ```

Ahora rellene la información de su base de datos Zabbix. Se verá así:

```
[book]
Description = MySQL book test database
Driver      = mariadb
Server      = 127.0.0.1
Port        = 3306
Database    = zabbix
```

4. Ahora vamos a probar si nuestra conexión funciona como esperábamos ejecutando esto:
   ```bash
   isql -v book
   ```

Deberías recibir un mensaje diciendo Conectado; si no es así, comprueba tus archivos de configuración e inténtalo de nuevo.

5. Ahora vamos al frontend de Zabbix para configurar nuestra primera comprobación de base de datos. Vaya a **Configuration** | **Hosts** y haga clic en el host llamado `lar-book-centos`, o todavía podría ser llamado servidor Zabbix. Ahora vaya a **Items**; queremos crear un nuevo elemento aquí haciendo clic en el botón **Create item**.

   **Consejo**
   Si aún no lo has hecho, una gran manera de mantener tu Zabbix estructurado es mantener todos los nombres de host en Zabbix iguales al nombre real del servidor. Cambie el nombre de su servidor Zabbix por defecto en el frontend a lo que realmente ha llamado a su servidor.

Queremos añadir un elemento con los siguientes parámetros:

![Figura 3.48 - Página de configuración de elementos Zabbix, elementos en la base de datos Zabbix](https://static.packt-cdn.com/products/9781803246918/graphics/image/B18275_03_048.jpg)
Figura 3.48 - Página de configuración de elementos Zabbix, elementos en la base de datos Zabbix

6. Asegúrese también de añadir una etiqueta al elemento:
   ![Figura 3.49 - Página de configuración de elementos Zabbix, elementos en la base de datos Zabbix, pestaña Etiquetas](https://static.packt-cdn.com/products/9781803246918/graphics/image/B18275_03_049.jpg)
   Figura 3.49 - Página de configuración de elementos Zabbix, elementos en la base de datos Zabbix, pestaña Etiquetas
7. Ahora haga clic en el botón **Add** y haga clic en el nombre del host para añadir las macros, como se indica a continuación:
   ![Figura 3.50 - Página de configuración de macros de host Zabbix](https://static.packt-cdn.com/products/9781803246918/graphics/image/B18275_03_050.jpg)
   Figura 3.50 - Página de configuración de macros de host Zabbix
8. Ahora si vamos a **Monitoring** | **Hosts** y pulsamos en **Latest data** para nuestro host, veremos esto:
   ![Figura 3.51 - Página de últimos datos de Zabbix para el host lar-book-centos, elementos en la base de datos de Zabbix](https://static.packt-cdn.com/products/9781803246918/graphics/image/B18275_03_051.jpg)
   Figura 3.51 - Página de últimos datos de Zabbix para el host lar-book-centos, elementos en la base de datos de Zabbix

Ahora podemos ver directamente desde la base de datos cuántos elementos se han escrito en ella.

### Cómo funciona...

La monitorización de bases de datos de Zabbix funciona conectándose a tu base de datos con el middleware API ODBC. Cualquier base de datos soportada por ODBC puede ser consultada con la monitorización de bases de datos de Zabbix:

![Figura 3.52 - Diagrama de comunicación entre el servidor Zabbix y ODBC](https://static.packt-cdn.com/products/9781803246918/graphics/image/B18275_03_052.jpg)
Figura 3.52 - Diagrama de comunicación entre el servidor Zabbix y ODBC

Básicamente, su servidor Zabbix envía un comando con, por ejemplo, su consulta MySQL al conector ODBC. Su conector ODBC envía esta consulta a la base de datos a través de la API ODBC, que a su vez devuelve un valor a ODBC. ODBC reenvía el valor al servidor Zabbix y listo: tenemos un valor en nuestro elemento.

### Y aún hay más...

Puedes hacer un montón de consultas a tus bases de datos con la monitorización de bases de datos de Zabbix, pero ten en cuenta que estás trabajando con consultas reales. Consultar una base de datos requiere tiempo y capacidad de procesamiento, así que mantenga su monitorización de bases de datos estructurada y defina los tiempos de ejecución adecuados.

Alternativamente, podemos utilizar el agente 2 de Zabbix para monitorizar la mayoría de las bases de datos de forma nativa. Esto puede mejorar la seguridad y el rendimiento y mantener la complejidad más baja.

---

## Configuración de la monitorización del agente HTTP

Con el agente HTTP de Zabbix, podemos monitorizar una página web o API recuperando datos de ella. Por ejemplo, si hay un contador en una página web y queremos mantener un ojo en ese valor del contador, podemos hacerlo con el monitor HTTP de Zabbix.

### Preparándonos

Vamos a necesitar una página web para monitorizar y necesitaremos nuestro servidor Zabbix. Para su comodidad, hemos añadido una página a nuestro propio sitio web para recuperar un valor de. Aquí está la página: https://oicts.com/book-page/.

Tenga en cuenta también que su servidor Zabbix necesitará una conexión activa a Internet para esta receta.

### Cómo hacerlo...

Vamos a sondear la página web especial que hemos creado para usted mostrando un recuento de visitantes que actualmente está configurado en ella. Se trata de un contador real del número de veces que los visitantes (únicos) han abierto la URL:

1. Navegue a su frontend Zabbix y vaya a **Configuration** | **Hosts**, a continuación, haga clic en el host llamado `lar-book-agent_simple`.
2. Ahora vaya a **item**; queremos crear un nuevo elemento aquí haciendo clic en el botón **Create item**. Vamos a necesitar crear un elemento de agente HTTP como se muestra en la siguiente captura de pantalla:

![Figura 3.53 - Página de configuración de Zabbix Item, recuento de visitantes en la página oicts.com](https://static.packt-cdn.com/products/9781803246918/graphics/image/B18275_03_053.jpg)
Figura 3.53 - Página de configuración de Zabbix Item, recuento de visitantes en la página oicts.com

3. No olvide editar Tipo de información.

![Figura 3.54 - Página de configuración de Zabbix Item, recuento de visitantes en la página oicts.com, parte inferior de la página](https://static.packt-cdn.com/products/9781803246918/graphics/image/B18275_03_054.jpg)
Figura 3.54 - Página de configuración de Zabbix Item, recuento de visitantes en la página oicts.com, parte inferior de la página

4. También necesitaremos añadir una etiqueta a este elemento.

![Figura 3.55 - Página de configuración de elementos de Zabbix, recuento de visitantes en la página oicts.com, pestaña Etiquetas](https://static.packt-cdn.com/products/9781803246918/graphics/image/B18275_03_055.jpg)
Figura 3.55 - Página de configuración de elementos de Zabbix, recuento de visitantes en la página oicts.com, pestaña Etiquetas

5. Utilice los siguientes pasos de preprocesamiento:
   ![Figura 3.56 - Página de configuración de elementos de Zabbix, recuento de visitantes en la página oicts.com, pestaña Preprocesamiento](https://static.packt-cdn.com/products/9781803246918/graphics/image/B18275_03_056.jpg)
   Figura 3.56 - Página de configuración de elementos de Zabbix, recuento de visitantes en la página oicts.com, pestaña Preprocesamiento
6. Ahora navegue a **Monitoring** | **Hosts** y abra la página Últimos datos para nuestro host `lar-book-agent_simple`. Si todo está funcionando como debería, ahora deberíamos estar solicitando el recuento de visitantes de esta página cada 15 minutos de la siguiente manera:
   ![Figura 3.57 - Página de últimos datos de Zabbix](https://static.packt-cdn.com/products/9781803246918/graphics/image/B18275_03_057.jpg)
   Figura 3.57 - Página de últimos datos de Zabbix

### Cómo funciona...

Lo que hacemos aquí es solicitar la página web completa a Zabbix navegando a la página con el agente HTTP y descargándola. Cuando tenemos el contenido completo de la página, en este caso, una página HTML/PHP, podemos procesar los datos:
![Figura 3.58 - Diagrama que muestra la comunicación del agente HTTP de Zabbix](https://static.packt-cdn.com/products/9781803246918/graphics/image/B18275_03_058.jpg)
Figura 3.58 - Diagrama que muestra la comunicación del agente HTTP de Zabbix

Le pedimos a nuestro preprocesador mediante una regex que recorra el código solicitado y sólo muestre el número que hay detrás de donde pone Total Page Visits:.

Todo lo que queda es el número, listo para que lo utilicemos en gráficos y otros tipos de visualización de datos.

---

## Uso del preprocesamiento de Zabbix para modificar los valores de los elementos

El preprocesamiento de valores de ítems es una funcionalidad importante en Zabbix; podemos usarla para crear todo tipo de comprobaciones. Ya hemos hecho algo de preprocesamiento en este capítulo, pero vamos a profundizar un poco más en él y en lo que hace.

### Comenzando

Vamos a necesitar un servidor Zabbix para crear nuestra comprobación. También necesitaremos un agente Zabbix pasivo en un host Linux para obtener nuestros valores y preprocesarlos. Podemos utilizar el agente que se está ejecutando en nuestro servidor Zabbix para esto; en mi caso, este es `lar-book-centos`.

### Cómo hacerlo...

1. Vamos a empezar por iniciar sesión en nuestro frontend Zabbix e ir a **Configuration** | **Hosts**.
2. Haz clic en el host de tu servidor Zabbix; en mi caso, se llama `lar-book-centos`.
3. Ahora ve a Items y haz clic en el botón azul **Create item** en la esquina superior derecha. Vamos a crear un nuevo elemento con la siguiente información:
   ![Figura 3.59 - Pantalla de creación de nuevo elemento, Obtener estadísticas de tráfico desde CLI](https://static.packt-cdn.com/products/9781803246918/graphics/image/B18275_03_059.jpg)
   Figura 3.59 - Pantalla de creación de nuevo elemento, Obtener estadísticas de tráfico desde CLI</p>
4. No olvide añadir su etiqueta:
   ![Figura 3.60 - Pantalla de creación de nuevo elemento, Obtener estadísticas de tráfico de CLI, pestaña Etiquetas](https://static.packt-cdn.com/products/9781803246918/graphics/image/B18275_03_060.jpg)
   Figura 3.60 - Pantalla de creación de nuevo elemento, Obtener estadísticas de tráfico de CLI, pestaña Etiquetas
5. Asegúrese de cambiar `ens192` a su propia interfaz de red primaria. Puede encontrar su interfaz de red primaria iniciando sesión en la CLI de Linux y ejecutando lo siguiente:

   ```bash
    Ifconfig 
   ```
6. De vuelta en la pantalla de creación de **item**, haga clic en el botón azul **Add**. Este item utilizará el agente de Zabbix para ejecutar un comando remoto en la CLI de Linux.
7. Cuando navegamos a este nuevo item, podemos ver que el item se convierte en no soportado. Esto se debe a que cuando usamos la clave **system.run**, necesitamos permitirla en la configuración del agente de Zabbix:

   ![Figura 3.61 - Información de item system.run sistema métrico no admitida](https://static.packt-cdn.com/products/9781803246918/graphics/image/B18275_03_061.jpg)
   Figura 3.61 - Información de item `system.run` sistema métrico no admitida
8. Acceda a la CLI Linux del host monitorizado y edite la configuración del agente Zabbix con esto:

   ```bash
   vim /etc/zabbix/zabbix_agent2.conf
   ```
9. Vaya a la línea Option: **AllowKey** y añada **AllowKey=system.run[*]** como se muestra aquí:
   ![Figura 3.62 - Fichero de configuración del agente Zabbix, AllowKey=system.run[*]](https://static.packt-cdn.com/products/9781803246918/graphics/image/B18275_03_062.jpg)
   Figura 3.62 - Fichero de configuración del agente Zabbix, AllowKey=system.run[*]
10. Guarde el archivo y reinicie el agente Zabbix con esto

```bash
systemctl restart zabbix-agent2
```

11. De vuelta en el frontend de Zabbix, el error que notamos en el paso 7 debería desaparecer después de unos minutos.
12. Navegue a **Monitoring** | **Latest data** y filtre en el host **lar-book-centos** de su servidor Zabbix y el nombre del nuevo item Obtener estadísticas de tráfico desde CLI.
13. Ahora el valor debería ser extraído del host. Si hacemos clic en **History**, podemos ver el valor completo; debería tener el siguiente aspecto:
    ![Figura 3.63 - Comando system.run del agente Zabbix ejecutando los resultados de ifconfig ens192](https://static.packt-cdn.com/products/9781803246918/graphics/image/B18275_03_063.jpg)
    Figura 3.63 - Comando system.run del agente Zabbix ejecutando los resultados de ifconfig ens192
14. La información que se ve en la imagen es demasiada para un solo item. Necesitamos dividirla. Usemos el preprocesado para obtener el número de bytes RX de la información.
15. Vuelva a **Configuration** | **Hosts** y haga clic en el host de su servidor Zabbix. Vaya a **Items** de este host.
16. Haga clic en el Item **Obtener estadísticas de tráfico de CLI** para editarlo. Cambie el nombre a `Total RX traffic in bytes for ens192` y añada `B` a Units, donde B significa bytes. Quedará así:
    ![Figura 3.64 - Elemento system.run del agente Zabbix](https://static.packt-cdn.com/products/9781803246918/graphics/image/B18275_03_064.jpg)
    Figura 3.64 - Elemento system.run del agente Zabbix
17. Añada su tag:
    ![Figura 3.65 - Pantalla de creación de nuevo item, Obtener estadísticas de tráfico de CLI, pestaña Etiquetas](https://static.packt-cdn.com/products/9781803246918/graphics/image/B18275_03_065.jpg)
    Figura 3.65 - Pantalla de creación de nuevo item, Obtener estadísticas de tráfico de CLI, pestaña Etiquetas
18. Ahora haga clic en **Preprocessing** y pulse el botón subrayado **Add**.
19. Se añadirá un campo de expresión regular (regex), que vamos a rellenar para que coincida con el número total de bytes de su interfaz. Rellene lo siguiente:
    ![Figura 3.66 - Preprocesamiento del elemento system.run del agente Zabbix](https://static.packt-cdn.com/products/9781803246918/graphics/image/B18275_03_066.jpg)
    Figura 3.66 - Preprocesamiento del elemento system.run del agente Zabbix
20. Asegúrese de seleccionar también la casilla **Custom on fail**.
21. Hagamos clic de nuevo en el botón subrayado **Add** y utilicemos el menú desplegable de este nuevo paso para seleccionar **Discard unchanged**. El resultado final será el siguiente:
    ![Figura 3.67 - Preprocesamiento del elemento system.run del agente Zabbix](https://static.packt-cdn.com/products/9781803246918/graphics/image/B18275_03_067.jpg)
    Figura 3.67 - Preprocesamiento del elemento system.run del agente Zabbix
22. Ahora podemos pulsar el botón azul Update para terminar de editar este elemento.
23. Navegue de nuevo a **Monitoring** | **Latest data** y filtre sobre su host y el nuevo nombre del ítem, **Tráfico RX total en bytes para ens192**. Asegúrese de utilizar su propio nombre de interfaz.
24. Ahora podemos ver nuestro valor entrando, y tenemos un elemento mostrando nuestro tráfico RX total para nuestra interfaz principal:
    ![Figura 3.68 - Últimos datos del elemento de tráfico Zabbix Total RX](https://static.packt-cdn.com/products/9781803246918/graphics/image/B18275_03_068.jpg)
    Figura 3.68 - Últimos datos del elemento de tráfico Zabbix Total RX

### Cómo funciona...

Ya hemos realizado algún preprocesamiento en la receta Trabajar con elementos calculados y dependientes para obtener datos de un elemento maestro. También hemos utilizado el preprocesamiento en la receta Configuración de la monitorización del agente HTTP para obtener un valor específico de una página web. Sin embargo, no hemos repasado los conceptos de preprocesamiento utilizados en esas recetas, así que vamos a repasarlos.

Cuando se trabaja con preprocesamiento, es importante conocer la configuración básica. Echemos un vistazo a los datos entrantes antes de utilizar el preprocesamiento:
![Figura 3.69 - Comando system.run del agente Zabbix ejecutando los resultados de ifconfig ens192](https://static.packt-cdn.com/products/9781803246918/graphics/image/B18275_03_063.jpg)
Figura 3.69 - Comando system.run del agente Zabbix ejecutando los resultados de ifconfig ens192

Esto es mucha información. Cuando nos fijamos en cómo se utilizan los elementos de Zabbix, tratamos de poner la información comprensible en un solo elemento. Por suerte, podemos preprocesar este elemento antes de almacenar el valor en Zabbix. En la siguiente figura, podemos ver los pasos de preprocesamiento que añadimos a nuestro elemento:
![Figura 3.70 - Preprocesamiento del elemento system.run del agente Zabbix con dos pasos](https://static.packt-cdn.com/products/9781803246918/graphics/image/B18275_03_070.jpg)
Figura 3.70 - Preprocesamiento del elemento system.run del agente Zabbix con dos pasos

Nuestro primer paso es una regex. Este paso se asegurará de utilizar sólo los números que necesitamos. Hacemos coincidir la palabra **RX**, luego la palabra **bytes**, y una secuencia de números después de ellas. De esta manera, terminamos con el número total de bytes RX en el grupo de captura 2. Por eso rellenamos `\2` en el campo de salida. También especificamos **Custom on fail**, que descartará cualquier valor si la regex no coincide.

Nuestro segundo paso es descartar cualquier valor que sea igual al valor recibido anteriormente. En lugar de almacenar valores duplicados, simplemente los descartamos y ahorramos algo de espacio en nuestra base de datos Zabbix.

**Consejo**

Es mucho más fácil construir una regex cuando se utiliza una herramienta en línea como `https://regex101.com/`. Puedes ver qué número obtendrán tus grupos de captura, y también hay mucha información valiosa en las herramientas.

Es importante señalar aquí que los pasos se ejecutan en la secuencia en que se definen en el frontend. Si el primer paso falla, el elemento se convierte en no soportado a menos que Custom on fail esté configurado para hacer otra cosa.

Al añadir preprocesamiento a Zabbix, abrimos todo un abanico de opciones para nuestros elementos, y podemos alterar nuestros datos de casi cualquier forma que sea necesaria. Estos dos pasos son sólo el comienzo de las opciones que se abren al sumergirse en el mundo del preprocesamiento de Zabbix.

### Ver también

El preprocesamiento en Zabbix es un tema importante, y es imposible cubrir todos sus aspectos en una sola receta. Los dos pasos de preprocesamiento en el ejemplo de esta receta son sólo dos de las muchas opciones que podemos utilizar. Echa un vistazo a la documentación oficial de Zabbix para ver las otras opciones que podemos utilizar:

https://www.zabbix.com/documentation/current/en/manual/config/items/preprocessing

---
