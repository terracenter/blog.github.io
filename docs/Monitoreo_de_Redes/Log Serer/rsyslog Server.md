# Log Server Rsyslog Debian 12 
 
 1. Configurar Rsyslog en el servidor
    * Instalación de paquetes
        ```bash
        sudo apt install -y rsyslog
        ```
        ```
    * Tras la instalación, puede comprobar su estado de funcionamiento de la siguiente manera:
        sudo systemctl status rsyslog
        ```
    * Configuraremos `rsyslog` para que se ejecute en modo servidor. El archivo de configuración es el archivo `/etc/rsyslog.conf`. 
        ```bash
        sudo vim /etc/rsyslog.conf
         ```

     * Proceda y descomente las siguientes líneas que permiten la recepción de syslog UDP y TCP desde clientes remotos.
         ```
        # provides UDP syslog reception
        module(load="imudp")
        input(type="imudp" port="514")

        # provides TCP syslog reception
        module(load="imtcp")
        input(type="imtcp" port="514")

        $template remote-incoming-logs,"/var/log/%HOSTNAME%/%PROGRAMNAME%.log"
        *.* ?remote-incoming-logs

        ```
        * La plantilla que el demonio `Rsyslog` utilizará para almacenar los registros entrantes de los sistemas cliente.
        * Los archivos de registro utilizarán la siguiente convención de nomenclatura:
            * `/%HOSTNAME%/` - Es el nombre de host del sistema cliente.
            * `/%PROGRAMNAME%` - Identifica el programa cliente que creó el archivo de registro.

    * Para aplicar los cambios, reinicie el demonio rsyslog.
        ```bash
        sudo systemctl restart rsyslog
        ```

    * Por defecto, rsyslog escucha en el puerto 514. Puede confirmar que este es el puerto en el que escucha el demonio rsyslog ejecutando el comando ss.
        ```bash
        sudo ss -tunlp | grep 514
        ```

2. Configurar las reglas del cortafuegos para rsyslog
    * 514/tcp
    * 514/udp