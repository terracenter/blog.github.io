---
title: Asterisk en Debian 12
---

!!! Fuente
[https://hotkey404.com/installing-asterisk-20-from-source-on-debian-12/](https://hotkey404.com/installing-asterisk-20-from-source-on-debian-12/)


[//]: # (fin-Fuente)


Actualización del servidor
--------------------------
```bash
sudo apt update && sudo apt upgrade
```
### Herramientas y Utilidades Requeridas
```bash
sudo apt install -y vim-nox wget tar curl subversion
```

Proceso de Instalación
----------------------

!!! nota
		  
Siempre es bueno usar la ultima LTS

[//]: # (fin-nota)		

### Las fuentes
* Descarga
```
wget http://downloads.asterisk.org/pub/telephony/asterisk/asterisk-20-current.tar.gz
```
* Descomprimir 
```bash
tar zxvf asterisk-20-current.tar.gz
```
* Eliminar paquete (No se utilizará mas)
```bash
rm -rf asterisk-20-current.tar.gz
```
```bash
cd asterisk-20*
```

Configuración de archivos de instalación de Asterisk
----------------------------------------------------
### Soporte para MP3
```bash
contrib/scripts/get_mp3_source.sh  
```

### Pre-Requisitos
```bash
sudo contrib/scripts/install_prereq install
```

#### Limpiar cualquier configuración previa
```bash
make distclean
```

### Configuración archivos de instalación
```bash
./configure  CFLAGS=-"O2 -pipe -march=native" LDFLAGS="-O2 -pipe -march=native" --with-pjproject-bundled --with-jansson-bundled 
```

### Seleccionar los paquetes 
```bash 
make menuselect
```
### Compilación
```bash
make -j4 # -j4 es opcional 
```
Instalación del Asterisk 
------------------------
```bash
sudo make install
```
### Instalar una configuración de ejemplo
```bash
sudo make samples
``` 
```bash
sudo mkdir /etc/asterisk/samples
```
```bash
sudo mv /etc/asterisk/*.*  /etc/asterisk/samples/
```

### Configuración básica de una pbx
```bash
sudo make basic-pbx
```

### Archivos de inicio del servicio Asterisk.
```bash
sudo make config
```

### Creación del usuario Asterisk
```bash
sudo useradd -c "Asterisk User" -M -r -s /usr/sbin/nologin -U asterisk
```

#### Actualizaremos la propiedad de los siguientes directorio y archivos:
```bash
sudo chown -R asterisk:asterisk /var/run/asterisk 
```
```bash
sudo chown -R asterisk:asterisk /etc/asterisk 
```
```bash
sudo chown -R asterisk:asterisk /var/{lib,log,spool}/asterisk 
```

### Habilitar el usuario de Asterisk
```bash
sudo vim /etc/default/asterisk
```
```
AST_USER="asterisk"
AST_GROUP="asterisk"
```

### activación del logs
```bash
sudo vim /etc/asterisk/logger.conf
```
```
full = verbose,notice,warning,error,debug
security = security
```


Arrancar el sistema Asterisk
----------------------------
### Al iniciar el sistema operativo
```bash
sudo systemctl enable asterisk.service
```

### Iniciar el servicio
```bash
sudo systemctl start asterisk.service
```

### Validar el servicio
```bash
sudo systemctl status asterisk.service
```

Codecs
------
### Activar el codec opus
- Descarga
```bash
wget https://downloads.digium.com/pub/telephony/codec_opus/asterisk-20.0/x86-64/codec_opus-20.0_current-x86_64.tar.gz
```
- Desempaquetar
```bash
tar -xzvf codec_opus-20.0_current-x86_64.tar.gz 
```   
```bash
cd codec_opus-20*
```

- Copiar a la carpeta de módulos
```bash 
sudo cp codec_opus.so /usr/lib/asterisk/modules
```
```bash
sudo cp format_ogg_opus.so /usr/lib/asterisk/modules
```

- Copiar a la carpeta de documentación
```bash
sudo cp codec_opus_config-en_US.xml /var/lib/asterisk/documentation/
```

- Configurar que el modulo cargue al iniciar o reiniciar el servicio asterisk
```bash
sudo vim /etc/asterisk/modules.conf 
```
```
load = codec_opus.so
```
```
load = format_ogg_opus.so
```
- Reiniciar Asterisk
```bash
sudo systemctl restart asterisk.service 
```
- Validar que el codec opus este activo
```bash
sudo asterisk -rx "module show like opus"
```
```bash
sudo asterisk -rx "core show translation paths opus"
```

### Activar el codec g729 (Validar)
* Descargar el paquete
```bash
wget https://downloads.digium.com/pub/telephony/codec_g729/asterisk-20.0/x86-64/codec_g729a-20.0_current-x86_64.tar.gz
```

* Desempacar
```bash
tar -xvzf codec_g729a-20.0_current-x86_64.tar.gz 
```

* Copiar el modulo 
```bash
sudo cp codec_g729a-20.0_3.1.10-x86_64/codec_g729a.so /usr/lib/asterisk/modules/
```

* Activar
```bash
sudo vim /etc/asterisk/modules.conf
```
```
load = codec_g729a.so
```

* Reiniciar Asterisk
```bash
sudo systemctl restart asterisk.service 
```

* Validar que el codec opus este activo
```bash
sudo asterisk -rx "module show like g729a"
```
```bash
sudo asterisk -rx "core show translation paths g729a"
```