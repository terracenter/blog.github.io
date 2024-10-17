# Cambiar el nombre de un Proxmox
>> NOTA No es ideal cambiar el nombre de un Proxmox VE después que esta Producción.

## Introducción

Proxmox VE utiliza el nombre de host como nombre de nodo, por lo que cambiarlo funciona de forma similar a cambiar el nombre de host. Esto debe hacerse en un nodo vacío.


## Detener servicios

```bash
sudo systemctl stop pve-cluster.service 
```
```bash
sudo systemctl stop pveproxy.service 
```


## Cambiar nombre de host
Para cambiar el nombre de un host PVE independiente, debes editar los siguientes archivos:

* /etc/hosts
    ```bash
    sudo nano /etc/hosts 
    ```
    ```
    IP nuevo_nombre.dominio.algo nuevo_nombre
    ```

* /etc/hostname
    ```bash
    sudo hostnamectl hostname nuevo_nombre
    ```
Hay otros archivos que es posible que desee editar, no son importantes para las funciones de Proxmox VE en sí.

* /etc/mailname (En 8.x no veo que exista)
* /etc/postfix/main.cf
    ```bash
    sudo vim /etc/postfix/main.cf  
    ```
    ```
    myhostname=nuevo_nombre.domimio.algo
    ```
> **NOTA**:  Ahora mueva los archivos de configuración, como el pmxcfs tiene algunas restricciones para garantizar la coherencia no se puede cambiar el nombre de las carpetas no vacías. Por lo tanto, si usted tiene VMs o contenedores en el nodo, que no se recomienda cuando se cambia el nombre de un nodo, usted tiene que volver a crear la estructura de carpetas y copiar los archivos por nivel de carpeta.

Copie también el contenido de /var/lib/rrdcached/db/pve2-{node,storage}/old-hostname a /var/lib/rrdcached/db/pve2-{node,storage}/new-hostname y elimine el directorio antiguo.

```bash
sudo cp -r /etc/pve/nodes/nombre_viejo/* /etc/pve/nodes/nombre_nuevo 
```
```bash
sudo rm -rf /etc/pve/nodes/nombre_viejo 
```
```bash
sudo mv /var/lib/rrdcached/db/pve2-node/nombre_viejo /var/lib/rrdcached/db/pve2-node/nombre_nuevo
```
```bash
sudo mv /var/lib/rrdcached/db/pve2-storage/nombre_viejo /var/lib/rrdcached/db/pve2-storage/nombre_nuevo 
```

```
sudo reboot
```


* General de nuevo los SSL (Se debe hacer para pode ingresar a la web)
    ```
    sudo pvecm updatecerts -f
    ```
