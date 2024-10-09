## Prefacio

Bienvenido a Zabbix 6 IT Infrastructure Monitoring Cookbook. La infraestructura de TI abarca desde Windows y Linux hasta redes y desarrollo, y básicamente cualquier cosa que se ejecute en hardware informático. En este libro, vamos a repasar varios temas útiles para cualquier persona en TI que quiera utilizar Zabbix para monitorizar su infraestructura de TI.

## A quién va dirigido este libro

Los sistemas de monitorización a menudo se pasan por alto dentro de las organizaciones de TI, pero pueden proporcionar una visión general que le ahorrará tiempo, dinero y dolores de cabeza. Este libro es para los ingenieros de TI que quieren aprender acerca de Zabbix 6 y cómo utilizarlo para llevar sus entornos de TI al siguiente nivel.

## Qué cubre este libro

[Capítulo 1](https://), Instalando Zabbix y Empezando a Usar el Frontend, cubre cómo configurar Zabbix, opcionalmente con HA. También trabajaremos en el frontend de Zabbix.

[Capítulo 2](https://), Preparando las cosas con la gestión de usuarios de Zabbix, cubre cómo configurar tus primeros usuarios, grupos de usuarios y roles de usuario.

[Capítulo 3](https://), Configuración de la monitorización de Zabbix, cubre cómo configurar casi cualquier tipo de monitorización dentro de Zabbix.

[Capítulo 4](https://), Trabajando con disparadores y alertas, cubre cómo configurar disparadores y obtener alertas de ellos.

[Capítulo 5](https://), Construyendo sus propias plantillas estructuradas, cubre cómo construir plantillas estructuradas, que harán maravillas para mantener su configuración de Zabbix organizada.

[Capítulo 6](https://), Visualización de Datos, Inventario e Informes, cubre cómo visualizar datos en gráficos, mapas y cuadros de mando. También cubre cómo utilizar el inventario de Zabbix, la presentación de informes, y la funcionalidad de monitorización de servicios de negocio.

[Capítulo 7](https://), Uso del descubrimiento para la creación automática, cubre cómo utilizar el descubrimiento de Zabbix para la creación automática de host, así como elementos, disparadores, y más con agentes, SNMP, WMI, y JMX.

[Capítulo 8](https://), Configuración de proxies Zabbix, le enseña cómo configurar proxies Zabbix correctamente para su uso en un entorno de producción.

[Capítulo 9](https://), Integrando Zabbix con Servicios Externos, le enseña cómo integrar Zabbix con servicios externos para alertas.

[Capítulo 10](https://), Extendiendo la funcionalidad de Zabbix con scripts personalizados y la API de Zabbix, cubre cómo extender la funcionalidad de Zabbix usando scripts personalizados y la API de Zabbix.

[Capítulo 11](https://), Manteniendo tu configuración Zabbix, cubre cómo mantener una configuración Zabbix y mantener su rendimiento a lo largo del tiempo.

[Capítulo 12](https://), Gestión avanzada de bases de datos Zabbix, le enseña cómo gestionar bases de datos Zabbix para una configuración avanzada.

[Capítulo 13](https://), Llevando Zabbix a la Nube con la Integración de Zabbix en la Nube, cubre cómo usar Zabbix en la nube con servicios como AWS, Azure, y Docker.

## Para sacar el máximo partido a este libro

Debe tener una buena base en TI para entender la terminología utilizada en este libro. Este libro es mejor para personas con al menos un conocimiento inicial de sistemas de monitorización, Linux e ingeniería de redes.

![](https://static.packt-cdn.com/products/9781803246918/graphics/image/Preface_Table.jpg)

Asegúrate de tener un entorno de virtualización listo para crear máquinas virtuales para usar con las recetas. VirtualBox, VMware, o cualquier tipo de cliente/hipervisor servirá.

A lo largo del libro, haremos uso de VIM para editar archivos, así que asegúrate de instalarlo. Si no te sientes cómodo usando VIM, puedes sustituir el comando por NANO o cualquier otro que prefieras.

**Si estás utilizando la versión digital de este libro, te aconsejamos que escribas el código tú mismo o accedas al código desde el repositorio GitHub del libro (encontrarás un enlace en la siguiente sección). De este modo evitará posibles errores relacionados con el copiado y pegado de código.**

## Descarga de los archivos de código de ejemplo

Puede descargar los archivos de código de ejemplo para este libro desde GitHub en: https://github.com/PacktPublishing/Zabbix-6-IT-Infrastructure-Monitoring-Cookbook

Si hay una actualización del código, se actualizará en el repositorio de GitHub.

También tenemos otros paquetes de código de nuestro rico catálogo de libros y vídeos disponibles en:

https://github.com/PacktPublishing/

¡Échales un vistazo!

### Convenciones utilizadas

A lo largo de este libro se utilizan una serie de convenciones textuales.

`Código en el texto`: Indica palabras clave en texto, nombres de tablas de bases de datos, nombres de carpetas, nombres de archivos, extensiones de archivos, nombres de rutas, URL ficticias, entradas de usuario y handles de Twitter. He aquí un ejemplo: "Es importante hacer una copia de seguridad de todos nuestros datos de configuración de Zabbix, que se encuentran en la carpeta `/etc/zabbix/`".

Un bloque de código se establece de la siguiente manera:

```bash
# MariaDB Server
# To use a different major version of the server, or to pin to a specific minor version, change URI below.
deb [arch=amd64] http://downloads.mariadb.com/MariaDB/ mariadb-10.5/repo/ubuntu xenial main
```

Cualquier entrada o salida de la línea de comandos se escribe de la siguiente manera:

```bash
systemctl start mariadb
```

**Negrita**: Indica un término nuevo, una palabra importante o palabras que se ven en pantalla. Por ejemplo, las palabras de los menús o cuadros de diálogo aparecen en **negrita**. He aquí un ejemplo: "Luego navegamos a **Monitoring**| **Hosts **y hacemos clic en **Latest data** para el host del servidor Zabbix".

Consejos o notas importantes

Aparece así.

## Contacto

Los comentarios de nuestros lectores son siempre bienvenidos.

**Comentarios generales**: Si tiene preguntas sobre cualquier aspecto de este libro, envíenos un correo electrónico a customercare@packtpub.com y mencione el título del libro en el asunto del mensaje.

**Erratas**: Aunque hemos puesto todo nuestro empeño en garantizar la exactitud de nuestros contenidos, es posible que se produzcan errores. Si ha encontrado un error en este libro, le agradeceríamos que nos lo comunicara. Visite www.packtpub.com/support/errata y rellene el formulario.

**Piratería**: si encuentra copias ilegales de nuestras obras en Internet, le agradeceríamos que nos facilitara la dirección o el nombre del sitio web. Póngase en contacto con nosotros en copyright@packt.com con un enlace al material.

**Si está interesado en convertirse en autor**: Si tiene conocimientos sobre algún tema y está interesado en escribir o colaborar en un libro, visite authors.packtpub.com.

### Comparta sus opiniones

Cuando haya leído Zabbix 6 IT Infrastructure Monitoring Cookbook - Second Edition, ¡nos encantaría conocer su opinión! Por favor, [haga clic aquí para ir directamente a la página de revisión de Amazon](https://packt.link/r/180324691Xhttps:/) para este libro y compartir sus comentarios.

Su opinión es importante para nosotros y la comunidad tecnológica y nos ayudará a asegurarnos de que estamos ofreciendo un contenido de excelente calidad.
