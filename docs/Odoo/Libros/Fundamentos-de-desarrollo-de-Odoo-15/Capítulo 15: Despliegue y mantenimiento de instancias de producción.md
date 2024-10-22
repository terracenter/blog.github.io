# Capítulo 15: Despliegue y mantenimiento de instancias de producción

En este capítulo, aprenderás los conceptos básicos de la preparación de un servidor Odoo para su uso en un entorno de producción.

La configuración y el mantenimiento de servidores es un tema no trivial en sí mismo y debe ser realizado por especialistas. La información que aquí se ofrece no es suficiente para garantizar que un usuario medio pueda crear un entorno resistente y seguro que aloje datos y servicios sensibles.

El objetivo de este capítulo es introducir los aspectos de configuración más importantes y las mejores prácticas específicas para los despliegues de Odoo. Esto ayudará a los administradores de sistemas a preparar sus hosts de servidores Odoo.

Empezarás configurando el sistema anfitrión, y luego instalarás los prer-requisitos de Odoo y el propio Odoo. Ubuntu es una elección popular para servidores en la nube, y será usado aquí. Luego, el archivo de configuración de Odoo necesita ser preparado. Hasta este punto, la configuración es similar a la utilizada para el entorno de desarrollo.

A continuación, Odoo necesita ser configurado como un servicio del sistema para que se inicie automáticamente cuando se inicie el servidor.

Para servidores alojados en una nube pública, Odoo debe ser servido a través de HTTPS. Para esto, aprenderás como instalar y configurar un proxy inverso Nginx usando un certificado autofirmado.

En la última sección se explica cómo realizar actualizaciones de servidores y preparar un entorno de ensayo que nos permita realizar pruebas de funcionamiento antes de aplicar las actualizaciones reales.

Los temas tratados en este capítulo son los siguientes:

* Preparación del sistema anfitrión
* Instalar Odoo desde el código fuente
* Configurando Odoo
* Configurar Odoo como un servicio del sistema
* Configurando un proxy inverso Nginx
* Configurando y reforzando HTTPS
* Manteniendo el servicio y los módulos de Odoo

Al final de este capítulo, serás capaz de configurar un servidor Odoo razonablemente seguro que sea lo suficientemente bueno para un uso de producción de bajo perfil. Sin embargo, las recetas dadas en este capítulo no son la única forma válida de desplegar Odoo - otros enfoques también son posibles.

---

## Requisitos técnicos

Para seguir este capítulo, necesitarás un servidor Ubuntu 20.04 limpio - por ejemplo, un servidor virtual privado (VPS) alojado en la nube.

El código y los scripts utilizados en este capítulo se encuentran en el directorio `ch15`/ del repositorio de GitHub en [https://github.com/PacktPublishing/Odoo-15-Development-Essentials](https://github.com/PacktPublishing/Odoo-15-Development-Essentials).

---

## Preparación del sistema anfitrión

Odoo se despliega normalmente en sistemas Linux basados en Debian. Ubuntu es una opción popular, y la última versión de soporte a largo plazo (LTS) es 20.04 Focal Fossa.

También pueden utilizarse otras distribuciones de Linux. El sistema CentOS/Red Hat Enterprise Linux (RHEL) también es popular en los círculos corporativos.

El proceso de instalación requiere acceso elevado, utilizando el super usuario ``root`` o el comando sudo. Cuando se utiliza una distribución Debian, el inicio de sesión predeterminado es `root`, que tiene acceso de administración, y el símbolo del sistema muestra `#`. En los sistemas Ubuntu, la cuenta `root` está deshabilitada. En su lugar, el usuario inicial se configura durante el proceso de instalación y es un `sudoer`, lo que significa que el usuario puede utilizar el comando sudo para elevar el acceso y ejecutar comandos con los privilegios de `root`.

Antes de iniciar la instalación de Odoo, las dependencias del sistema anfitrión deben ser instaladas, y un usuario específico debe ser creado para ejecutar el servicio de Odoo.

La siguiente sección explica las dependencias del sistema necesarias en un sistema Debian.

### Instalación de las dependencias del sistema

Cuando se ejecuta Odoo desde el código fuente, algunas dependencias necesitan ser instaladas en el sistema.

Antes de empezar, es una buena práctica actualizar el índice de paquetes y, a continuación, realizar una actualización para asegurarse de que todos los programas instalados están al día, como se indica a continuación:

```bash
sudo apt update && sudo apt upgrade -y
```

A continuación, se puede instalar la base de datos PostgreSQL. Nuestro usuario debe ser hecho super usuario de la base de datos para que tenga acceso de administración a la base de datos. Estos son los comandos para esto:

```bash
sudo apt install postgresql -y
sudo su -c "createuser -s $USER" postgres
```

**Nota**

Odoo puede usar una base de datos PostgreSQL existente, la cual esta instalada en su propio servidor. Si este es el caso, el servicio PostgreSQL no necesita ser instalado en el servidor Odoo y los detalles de conexión correspondientes deben ser establecidos en el archivo de configuración de Odoo.

Estas son las dependencias de Debian necesarias para ejecutar Odoo:

```bash
sudo apt install git python3-dev python3-pip \
python3-wheel python3-venv -y
```

```bash
sudo apt install build-essential libpq-dev libxslt-dev \
libzip-dev libldap2-dev libsasl2-dev libssl-dev
```

Para tener capacidades de impresión de informes, `wkhtmltox` debe estar instalado. La versión recomendada para Odoo 10 y posteriores es 0.12.5-1. Los enlaces de descarga se encuentran en https://github.com/wkhtmltopdf/wkhtmltopdf/releases/tag/0.12.5. Los nombres en clave de Ubuntu son bionic para la versión 18.04 y focal para la versión 20.04.

```bash
 wget "https://github.com/wkhtmltopdf/wkhtmltopdf\
/releases""/download/0.12.5/\
wkhtmltox_0.12.5-1.focal_amd64.deb" \
-O /tmp/wkhtml.deb
```

```bash
sudo dpkg -i /tmp/wkhtml.deb
```

```bash
sudo apt-get -fy install  # Fix dependency errors
```

La instalación del paquete puede reportar errores de dependencias faltantes. En este caso, el último comando forzará la instalación de esas dependencias y completará correctamente la instalación.

A continuación, se creará un usuario del sistema que se utilizará para los procesos de Odoo.

### Preparar un usuario dedicado al sistema

Una buena práctica de seguridad es ejecutar Odoo usando un usuario dedicado, que no tenga privilegios especiales en el sistema anfitrión.

Una opción popular para este nombre de usuario es `odoo`. Este es el comando para crearlo:

```bash
sudo adduser --home=/opt/odoo --disabled-password --gecos "Odoo" odoo
```

Los usuarios del sistema Linux pueden tener un directorio `home`. Para el usuario de Odoo, este es un lugar conveniente para almacenar los archivos de Odoo. Una opción popular para esto es `/opt/odoo`. La opción `--home` usada automáticamente crea este directorio y lo establece como el home del usuario odoo.

Este usuario no tiene acceso a la base de datos PostgreSQL todavía. Los siguientes comandos añaden ese acceso y crean la base de datos para que inicialice el entorno de producción de Odoo:

```bash
sudo su -c "createuser odoo" postgres
```

```bash
createdb --owner=odoo odoo-prod
```

Aquí, `odoo` es el nombre de usuario y `odoo-prod` es el nombre de la base de datos que soporta nuestra instancia de Odoo. El usuario `odoo` fue hecho dueño de la base de datos `odoo-prod`. Esto significa que tiene privilegios de creación y eliminación sobre esa base de datos, incluyendo la habilidad de eliminarla.

**Consejo**

Para funcionar, ***Odoo no requiere*** privilegios elevados a la base de datos que se está utilizando. Estos solo pueden ser necesarios para algunas operaciones de mantenimiento, como instalar o actualizar módulos. Entonces, para mejorar la seguridad, el usuario del sistema Odoo puede ser un usuario no-propietario de la base de datos. Tenga en cuenta que en este caso, el mantenimiento debe hacerse ejecutando Odoo con un usuario diferente al propietario de la base de datos.

Para iniciar una sesión con el usuario del sistema Odoo, utilice el siguiente comando:

```bash
sudo su - odoo
```

```bash
exit
```

Esto será usado para ejecutar los pasos de instalación como el usuario de Odoo. Cuando termine, el comando `exit` termina esa sesión y regresa al usuario original.

En la siguiente sección, continuaremos con la instalación del código de Odoo y las dependencias de Python en el directorio `/opt/odoo`.

---

## Instalar Odoo desde el código fuente
Aunque Odoo proporciona paquetes de sistema Debian/Ubuntu y CentOS/RHEL, la instalación desde el código fuente es una opción popular debido a la flexibilidad y control que proporciona.

El uso del código fuente proporciona un mejor control sobre lo que se despliega y facilita la gestión de cambios y correcciones una vez en producción. Por ejemplo, nos permite vincular el proceso de despliegue a un flujo de trabajo Git.

En este punto, las dependencias del sistema Odoo ya están instaladas, y la base de datos está lista para usarse. Ahora, el código fuente de Odoo puede ser descargado e instalado, junto con las dependencias requeridas de Python.

Veamos cómo descargar el código fuente de Odoo.

### Descargar el código fuente de Odoo
Tarde o temprano, su servidor necesitará actualizaciones y parches. Un repositorio de control de versiones puede ser de gran ayuda cuando llegue ese momento. Usamos `git` para obtener nuestro código de un repositorio, igual que hicimos al instalar el entorno de desarrollo.

A continuación, nos haremos pasar por el usuario `odoo` y descargaremos el código en su directorio home, de la siguiente manera:
```bash
sudo su - odoo
```
```bash
git clone https://github.com/odoo/odoo.git /opt/odoo/odoo15 -b 15.0 --depth=1
```

La opción `-b` asegura que obtenemos la rama correcta, y la opción `--depth=1` recupera sólo la última revisión del código, ignorando el (largo) historial de cambios y haciendo la descarga más pequeña y rápida.

**Consejo**

Git es una herramienta importante para gestionar las versiones de código de tus despliegues de Odoo. Si no estás familiarizado con Git, vale la pena aprender más sobre él. Un buen punto de partida es [http://git-scm.com/doc](http://git-scm.com/doc).

Los módulos personalizados normalmente también se gestionarán con Git y también deberían clonarse en el servidor de producción. Por ejemplo, el siguiente código añadirá los módulos personalizados de la librería al directorio `/opt/odoo/odoo15/library`:
```bash
git clone https://github.com/PacktPublishing/Odoo-15-Development-Essentials/opt/odoo/library
```

El código fuente de Odoo está disponible en el servidor, pero no se puede ejecutar todavía porque las dependencias requeridas de Python no están instaladas todavía. Vamos a instalarlas en la siguiente sección.

### Instalación de las dependencias de Python
Una vez descargado el código fuente de Odoo, los paquetes de Python requeridos por Odoo deben ser instalados.

Muchos de ellos también tienen paquetes de sistema Debian o Ubuntu. El paquete oficial de instalación de Odoo para Debian los utiliza, y los nombres de los paquetes de dependencia se pueden encontrar en el código fuente de Odoo en el archivo `debian/control`: [https://github.com/odoo/odoo/blob/15.0/debian/control](https://github.com/odoo/odoo/blob/15.0/debian/control).


Estas dependencias de Python también pueden instalarse directamente desde el **Python Package Index (PyPI)**. Hacer esto utilizando un **entorno virtual** Python proporciona una mejor protección contra los cambios que se realizan en el sistema anfitrión.

Los siguientes comandos crean un entorno virtual, lo activan, y luego instalan Odoo desde el código fuente, junto con todas las dependencias de Python necesarias:
```bash
$ python3 -m venv /opt/odoo/env15
```
```bash
$ source /opt/odoo/env15/bin/activate
```
```bash
(env15) $ pip install -r /opt/odoo/odoo15/requirements.txt
```
```bash
(env15) $ pip install -e /opt/odoo/odoo15
```

Y Odoo debería estar listo ahora. Cualquiera de los siguientes comandos se puede utilizar para confirmar esto:
```
(env15) $ odoo --version
Odoo Server 15.0
(env15) $ /opt/odoo/odoo15/odoo-bin --version
Odoo Server 15.0
$ /opt/odoo/env15/bin/python3 /opt/odoo/odoo15/odoo-bin --version
Odoo Server 15.0
$ /opt/odoo/env15/bin/odoo --version
Odoo Server 15.0
```

Entendamos estos comandos uno por uno:
* El primer comando se basa en el comando `odoo` puesto a disposición por `pip install -e /opt/odoo/odoo15`.
* El segundo comando no depende del comando `odoo`, y llama directamente al script de inicio de Odoo, `/opt/odoo/odoo15/odoo-bin`.
* El tercer comando no necesita que se active previamente el entorno virtual, ya que utiliza directamente el ejecutable Python correspondiente, que tiene el mismo efecto.
* El último comando hace lo mismo de una forma más compacta. Utiliza directamente el comando odoo disponible en ese entorno virtual. Esto puede ser útil para algunos scripts.

Odoo esta ahora listo para correr. El siguiente paso es ocuparse del archivo de configuración a utilizar, que explicaremos en la siguiente sección.

---

## Configuración de Odoo
Una vez instalado Odoo, es necesario preparar el archivo de configuración que utilizará el servicio de producción.

En la siguiente sub-sección se ofrece orientación sobre cómo hacerlo.

### Creación del archivo de configuración
Se espera que los archivos de configuración estén en el directorio del sistema `/etc`. Así, el archivo de configuración de producción de Odoo será almacenado en `/etc/odoo/odoo.conf`.

Si aún no lo ha hecho, cree una sesión para el usuario `odoo` y active el entorno virtual:
```bash
$ sudo su - odoo
$ python3 -m venv /opt/odoo/env15
```

Ahora, el siguiente comando se puede utilizar para crear un archivo de configuración por defecto:
```bash
(env15) $ odoo -c /opt/odoo/odoo.conf --save --stop-after-init
```