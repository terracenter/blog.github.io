# Instalación de FreeRADIUS 3.0 en Debian Debian 12 “Bookworm” y PostgreSQL 16


* Actualización del servidor
    ```bash
    sudo apt update && sudo apt upgrade
    ```
    ```bash
     reboot
    ```
* Paquetes y herramientas
    ```bash
    sudo apt install -y curl gnupg2 vim-nox
    ```

## PostgreSQL
* Cree la configuración del repositorio de archivos:
    ```bash
    sudo sh -c 'echo "deb https://apt.postgresql.org/pub/repos/apt $(lsb_release -cs)-pgdg main" > /etc/apt/sources.list.d/pgdg.list'
    ```

* Importar la clave de firma del repositorio:
    ```bash
    wget --quiet -O - https://www.postgresql.org/media/keys/ACCC4CF8.asc | sudo apt-key add -
    ```
* Actualice las listas de paquetes:
    ```bash
    sudo apt update
    ```
    ```bash
    sudo mv /etc/apt/trusted.gpg /etc/apt/trusted.gpg.d/
    ```

* Instale PostgreSQL 16
    ```bash
    sudo apt-get -y install postgresql-16
    ```

 * Usuario administrador de PostgreSQL `$USER`
  ```bash
    sudo su -c "createuser -sP $USER" postgres
    ```

## FreeRADIUS
* Añada la clave pública PGP de NetworkRADIUS:
    ```bash
    curl -s 'https://packages.networkradius.com/pgp/packages%40networkradius.com' sudo tee /etc/apt/keyrings/packages.networkradius.com.asc > /dev/null
    ```

* Añada un archivo de preferencias APT para garantizar que todos los paquetes freeradius se instalan desde el repositorio RADIUS de red:
     ```bash
    printf 'Package: /freeradius/\nPin: origin "packages.networkradius.com"\nPin-Priority: 999\n' sudo tee /etc/apt/preferences.d/networkradius > /dev/null
     ``

echo "deb [arch=amd64 signed-by=/etc/apt/keyrings/packages.networkradius.com.asc] http://packages.networkradius.com/freeradius-3.2/debian/bookworm bookworm main" sudo tee /etc/apt/sources.list.d/networkradius.list > /dev/null



* Por último, actualice la base de datos APT
    ```bash
    sudo apt-get update
    ```
* Instale los paquetes:
    ```bash
    sudo apt install -y freeradius freeradius-utils freeradius-postgresql
    ```

* Configuración de Freeradius
    ```bash 
    sudo ln -s  /etc/freeradius/3.0/mods-available/sql /etc/freeradius/3.0/mods-enabled/sql
    ```

* Buscar dentro del archivo `default`, -`sql` por `sql`
    ```bash 
    sudo vim  /etc/freeradius/3.0/sites-available/default
    ```
    **Donde**:
    * See "Authorization Queries" in mods-available/sql 
    * Log traffic to an SQL database.
    * See "Simultaneous Use Checking Queries" in mods-available/sql
    * See "Authentication Logging Queries" in mods-available/sql
    * Post-Auth-Type REJECT {

    
* Buscar dentro del archivo `inner-tunnel` todos los `-sql` y `#sql` cambiarlos por `sql`  
    ```bash 
    sudo vim /etc/freeradius/3.0/sites-available/inner-tunnel
    ```
    **Donde**:
    * See "Authorization Queries" in `mods-config/sql/main/$driver/queries.conf/`
    * See "Simultaneous Use Checking Queries" in `mods-config/sql/main/$driver/queries.conf`
    * See "Authentication Logging Queries" in `mods-config/sql/main/$driver/queries.conf`
    * log failed authentications in SQL, too



* Buscar en el archivo `sql`
    ```bash
    sudo vim  /etc/freeradius/3.0/mods-available/sql
    ```
    **Cambios**:
    * `dialect = "postgresql"`
    * `driver = "rlm_sql_${dialect}"`
    * ` server = "localhost"`
    * `port = 3306`
    * `login = "radius"`
    * `password = "radpass$2020"`
    * `read_clients = yes`
    * `read_clients = yes`


    ``bash
    sudo chown -R freerad:freerad /etc/freeradius/3.0/mods-enabled/sql
    ``
  
 * Usuario de FreeRADIUS en PostgreSQL 
    ```bash
    sudo su -c "createuser -P radius" postgres 
    ```
    ```
    clave: radpass$2020
    ```

*  * Creación de la Base de datos de FreeRADIUS  y asignada al usuario `radius`
    ```bash
    createdb --owner=radius radius
    ```

* Importar el esquema de base de datos
    ```bash
    sudo cat /etc/freeradius/3.0/mods-config/sql/main/postgresql/schema.sql |  psql -h 127.0.0.1 -U radius -W  -d radius
    ```
    ```
    clave: radpass$2020
    ```
* Importar el esquema de base de datos de usurios
    ```bash
    sudo cat /etc/freeradius/3.0/mods-config/sql/cui/postgresql/schema.sql |  psql -h 127.0.0.1 -U radius -W  -d radius
    ```
    ```
    clave: radpass$2020
    ```


* Reiniciar el servicio
    sudo systemctl restart freeradius





* Prueba de funcionamiento usuario y claves simples.
    ```bash
    psql postgres
    ``
    ```bash
    \c radius
    ```

    ```
    INSERT INTO radcheck VALUES (1, 'demo', 'Cleartext-Password', ':=', '12345');
    ```
    ```
    INSERT INTO radcheck VALUES (2, 'test', 'Cleartext-Password', ':=', '54321');
    ```
    ```
    \q
    ```
* Prueba uno
    ```bash
    radtest demo 12345 localhost 10 testing123
    ```
    ```
    Sent Access-Request Id 248 from 0.0.0.0:40049 to 127.0.0.1:1812 length 74
            User-Name = "demo"
            User-Password = "12345"
            NAS-IP-Address = 192.168.122.20
            NAS-Port = 10
            Message-Authenticator = 0x00
            Cleartext-Password = "12345"
    Received Access-Accept Id 248 from 127.0.0.1:1812 to 127.0.0.1:40049 length 20
    ```

* Prueba dos
    ```bash
    radtest test 54321 localhost 10 testing123
    ```
     ```
    Sent Access-Request Id 86 from 0.0.0.0:55341 to 127.0.0.1:1812 length 74
            User-Name = "test"
            User-Password = "54321"
            NAS-IP-Address = 192.168.122.20
            NAS-Port = 10
            Message-Authenticator = 0x00
            Cleartext-Password = "54321"
    Received Access-Accept Id 86 from 127.0.0.1:1812 to 127.0.0.1:55341 length 20
     ```

* Pruebas con usuarios con md5

* Hacer la GUI para entregar

* Fuentes:
    https://www.youtube.com/watch?v=SMDqr9rhs_g


## Rellenar SQL
Ahora debería crear algunos datos ficticios en la base de datos para probarlos. Es algo así
* En usergroup, pon entradas que coincidan con un nombre de cuenta de usuario con un nombre de grupo.
* En `radcheck`, ponga una entrada para cada nombre de cuenta de usuario con un atributo `'Cleartext-Password'` con un valor de su contraseña.
* En `radreply`, cree entradas para cada atributo de respuesta radius específico del usuario con su nombre de usuario.
* En `radgroupreply`, crear atributos para ser devueltos a todos los miembros del grupo.

A continuación se muestra un volcado de algunas tablas `'radius'` de ejemplo de una base de datos MySQL (con PostgreSQL el formato será ligeramente diferente, pero utiliza exactamente el mismo contenido).

Este ejemplo incluye tres usuarios, uno con una IP asignada dinámicamente por el NAS (fredf), otro con una IP estática (barney) y otro que representa una conexión enrutada de acceso telefónico (`dialrouter`):

