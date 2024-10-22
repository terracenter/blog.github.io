# Instalación Base de Gentoo SystemD 2024
* Preparar los discos
* Instalar los archivos de instalación de Gentoo
* Instalar el sistema base de Gentoo
* Configurar el núcleo Linux
* Instalar las herramientas del sistema
* Configurar el cargador de arranque
* Terminar la instalación

## Preparar los discos
### Crear la particiones Primarias
* Etiqueta de Disco
    ```bash
    parted -s -a optimal  /dev/vg/vda -- mklabel gpt
    ```

* Partición EFI
    ```bash
    parted -s /dev/vg/vda -- mkpart EFI fat32  1MiB 1025MiB 
    ```
    ```bash
    parted -s /dev/vg/vda set 1 esp on
    ```

* Partición SWAP (Si tienes )
    ```bash
    parted -s /dev/vg/vda -- mkpart SWAP sw  1026MiB 5121Mib
    ```
    ```bash
    parted -s /dev/vg/vda set 2 swap on
    ```

* Partición ROOT
    ```
    parted -s /dev/vg/vda -- mkpart ROOT ext4 7169Mib
    ```

* Partición LVM (Recomendado)
    ```
    parted -s /dev/vg/vda -- mkpart LVM  7170Mib -1Mib
    ```
    ```bash
    parted -s /dev/vg/vda set 4 lvm on
    ```
### Creando Particiones LVM
* 
    ```bash
    pvcreate /dev/vg/vda4
    ```
*     
    ```bash
    vgcreate vg /dev/vg/vda4
    ```
* Root
    ```bash
     lvcreate -L+6G -n usr vg
    ```
* Var    
    ```bash
    lvcreate -L+2G -n var vg
    ```
* Home
    ```bash
    lvcreate -L+2G -n home vg
    ```
* Distfiles    
    ```bash
    lvcreate -L+1G -n distfiles vg
    ```
* /var/tmp
    ```bash
    lvcreate -L+4G -n var_tmp vg
    ```
* /tmp
    ```bash
    lvcreate -L+2G -n tmp vg
    ```
* /opt
    ```bash
    lvcreate -L+2G -n opt vg
    ```
    
### Formateo y Montaje de Particiones   
* /Efi
* /Swap
* /root
* /home
* /opt
* /tmp
* /usr
* /var
* /var/tmp
* /var/cache/distfiles

#### Formateo
```bash
mkfs.vfat -F32 /dev/vg/vda1 && \
mkswap /dev/vg/vda2 && \
mkfs.xfs -f /dev/vg/vda3 && \
mkfs.xfs -f /dev/vg/vg/distfiles && \
mkfs.xfs -f /dev/vg/vg/home && \
mkfs.xfs -f /dev/vg/vg/opt && \
mkfs.xfs -f /dev/vg/vg/tmp && \
mkfs.xfs -f /dev/vg/vg/usr && \
mkfs.xfs -f /dev/vg/vg/var && \
mkfs.xfs -f /dev/vg/vg/var_tmp
```

#### Montaje 
```bash
swapon /dev/vda2                && \
mkdir -p /mnt/gentoo            && \
mkdir -p /mnt/gentoo/efi        && \
mount /dev/vda3 /mnt/gentoo     && \
mkdir -p /mnt/gentoo/{home,opt,tmp,usr,var} && \
mount /dev/vg/home /mnt/gentoo/home && \
mount /dev/vg/opt /mnt/gentoo/opt && \
mount /dev/vg/tmp /mnt/gentoo/tmp && \
mount /dev/vg/usr /mnt/gentoo/usr && \
mount /dev/vg//var /mnt/gentoo/var && \
mkdir -p /mnt/gentoo/var/{cache/distfiles,tmp} && \
mount /dev/vg/distfiles /mnt/gentoo/var/cache/distfiles && \
mount /dev/vg/var_tmp /mnt/gentoo/var/tmp
```


* Permisos de /tmp y /var/tmp
    ```bash
    chmod 1777 /mnt/gentoo/tmp && \
    chmod 1777 /mnt/gentoo/var/tmp
    ```

## Instalar los archivos de instalación de Gentoo

No multilibrería (64 bits puros)

---
**NOTA**

Los lectores que acaban de empezar con Gentoo no deben elegir un empaquetado sin multilib a menos que sea absolutamente necesario. Migrar de un sistema no-multilib a uno multilib requiere un conocimiento extremadamente bueno de Gentoo y de la cadena de herramientas de nivel inferior (incluso puede hacer que nuestros desarrolladores de la cadedena de herramientas se estremezcan un poco). No es para miedosos y está más allá del alcance de esta guía.

---

* Ajustar la Fecha/Hora correcta
    ```bash
    chronyd -q
    ```
* Instalar un archivo de stage no-multilib Systemd
    ```bash
    cd /mnt/gentoo/
    ```
    ```bash
    wget -c https://gentoo.osuosl.org/releases/amd64/autobuilds/20241013T160327Z/stage3-amd64-nomultilib-systemd-20241013T160327Z.tar.xz
    ```
    ```bash
    tar xpvf stage3-*.tar.xz --xattrs-include='*.*' --numeric-owner
    ```

* Configurar las opciones de compilación
    ```
    nano /mnt/gentoo/etc/portage/make.conf
    ```
    ```
    COMMON_CFLAGS="-march=native -O2 -pipe"
    ```
* MAKEOPTS

## Instalar el sistema base de Gentoo
* Copiar la información DNS
    ```bash
    cp --dereference /etc/resolv.conf /mnt/gentoo/etc/
    ```
* Montar los sistemas de archivos necesarios
    ```bash
    mount --types proc /proc /mnt/gentoo/proc   && \
    mount --rbind /sys /mnt/gentoo/sys          && \
    mount --make-rslave /mnt/gentoo/sys         && \
    mount --rbind /dev /mnt/gentoo/dev          && \
    mount --make-rslave /mnt/gentoo/dev         && \
    mount --bind /run /mnt/gentoo/run           && \
    mount --make-slave /mnt/gentoo/run          
    ```

    ```bash
    test -L /dev/vg/shm && rm /dev/vg/shm && mkdir /dev/vg/shm && \
    mount --types tmpfs --options nosuid,nodev,noexec shm /dev/vg/shm
    ```

* Entrar en el nuevo entorno
    ```bash
    chroot /mnt/gentoo /bin/bash
    source /etc/profile
    export PS1="(chroot) ${PS1}"
    ```
* Sistemas UEFI
    ```bash
    mkdir /efi  && \
    mount /dev/vg/vda1 /efi
    ```
* Configurar Portage
    ```bash
    emerge-webrsync -v
    ```

* Seleccionar los servidores réplica
    ```bash
    emerge -v --oneshot app-portage/mirrorselect
    
    ```bash
    mirrorselect -s3 -b10 -D
    ```
*  Actualizar el repositorio de ebuilds de Gentoo
    ```bash
    emerge --sync -v
    ```
* Leer los elementos de noticias (Recomendado)
    ```bash
    eselect news list
    ```
    ```bash
    eselect news read
    ```

* Elegir el perfil adecuado
    ```bash
    eselect profile list | grep stable | grep no-multilib
    ```
    ```
    [30]  default/linux/amd64/23.0/no-multilib/systemd (stable) *
    ```

* CPU_FLAGS_*
    ```bssh
    emerge  -q --oneshot app-portage/cpuid2cpuflags
    ```

    ```bash
    echo "*/* $(cpuid2cpuflags)" >> /etc/portage/package.use/00cpu-flags
    ```

* VIDEO_CARDS
    ```bash
    echo 'VIDEO_CARDS="intel"'  >> /etc/portage/make.conf
    ```

* ACCEPT_LICENSE
    ```bash
    echo 'ACCEPT_LICENSE="*"' >> /etc/portage/make.conf
    ```
* Actualizar el conjunto @world
    ```bash
    emerge -avq --update --deep --newuse @world
    ```

    ```
    * Some software with pre-loaded PAM libraries might experience
    * warnings or failures related to missing symbols and/or versions
    * after any update. While unfortunate this is a limit of the
    * implementation of PAM and the software, and it requires you to
    * restart the software manually after the update.
    * 
    * You can get a list of such software running a command like
    *   lsof / | grep -E -i 'del.*libpam\.so'
    * 
    * Alternatively, simply reboot your system.
    ```
    ```bash
    emerge -aqv lsof
    ```
    ```bash
    lsof / | grep -E -i 'del.*libpam\.so'
    ```

* Zona horaria
    ```bash
    ln -sf ../usr/share/zoneinfo/Europe/Brussels /etc/localtime
    ```

* Configurar localizaciones
    ```
    echo 'es_VE.UTF-8 UTF-8' >> /etc/locale.gen && \
    echo 'es_VE ISO-8859-1' >> /etc/locale.gen
    ```
    ```bash
    locale-gen
    ```

* Selección de localización
    ```bash
    eselect locale list
    ```
    ```
    [1]   C
    [2]   C.utf8
    [3]   POSIX
    [4]   es_VE
    [5]   es_VE.iso88591
    [6]   es_VE.utf8
    [7]   C.UTF8 *
    [ ]   (free form)
    ```
    ```bash
    eselect locale set 6
    ```
    ```bash
    echo 'LC_COLLATE="C.UTF-8"' >> /etc/env.d/02locale
    ```
    ```bash
    env-update && source /etc/profile && export PS1="(chroot) ${PS1}"
    ```


## Configurar el núcleo Linux
* Firmware para Linux 
    ```bash
    emerge -q sys-kernel/linux-firmware
    ```
* SOF Firmware
    ```bash
    emerge -q sys-firmware/sof-firmware
    ```
* Microcódigo
    ```bash
    echo "sys-firmware/intel-microcode initramfs" > /etc/portage/package.use/intel-microcode
    ```
    ```bash
    emerge -q sys-firmware/intel-microcode
    ```
* Instalando un núcleo de distribución
    ```bash
    echo 'sys-kernel/installkernel dracut' >> /etc/portage/package.use/installkernel
    ```
    
    USE="dist-kernel"


    ```bash
    emerge -aqv sys-kernel/gentoo-kernel-bin
    ```
    ```
    * Install additional packages for optional runtime features:
    *   net-misc/networkmanager for Networking support
    *   sys-fs/btrfs-progs for Scan for Btrfs on block devices
    *   net-fs/cifs-utils for Support CIFS
    *   sys-fs/cryptsetup[-static-libs] for Decrypt devices encrypted with cryptsetup/LUKS
    *   app-shells/dash for Allows use of dash instead of default bash (on your own risk)
    *   sys-apps/busybox for Allows use of busybox instead of default bash (on your own risk)
    *   sys-block/open-iscsi for Support iSCSI
    *   sys-fs/lvm2[lvm] for Support Logical Volume Manager
    *   sys-fs/mdadm for Support MD devices, also known as software RAID devices
    *   sys-fs/dmraid for Support MD devices, also known as software RAID devices
    *   sys-fs/multipath-tools for Support Device Mapper multipathing
    *   >=sys-boot/plymouth-0.8.5-r5 for Plymouth boot splash
    *   sys-block/nbd for Support network block devices
    *   net-fs/nfs-utils for Support NFS
    *   net-nds/rpcbind for Support NFS
    *   app-admin/rsyslog for Enable logging with rsyslog
    *   sys-fs/squashfs-tools for Support Squashfs
    *   app-crypt/tpm2-tools for Support TPM 2.0 TSS
    *   net-wireless/bluez for Support Bluetooth (experimental)
    *   sys-apps/biosdevname for Support BIOS-given device names
    *   sys-apps/nvme-cli for Support network NVMe
    *   app-misc/jq for Support network NVMe
    *   sys-apps/rng-tools for Enable rngd service to help generating entropy early during boot
    *   sys-apps/systemd[boot] for building Unified Kernel Images with dracut (--uefi)
    *   sys-apps/systemd-utils[boot] for building Unified Kernel Images with dracut (--uefi)
    *   sys-kernel/installkernel[dracut] for automatically generating an initramfs on each kernel installation
    *   sys-kernel/installkernel[dracut,uki] for automatically generating an UKI on each kernel installation
    ```
    ```bash
    mkdir /etc/dracut.conf.d/
    ```
    ```bash
    echo 'add_dracutmodules+=" usrmount "' > /etc/dracut.conf.d/usrmount.conf
    ```

* LVM
    ```bash
    echo 'sys-fs/lvm2 lvm ' >> /etc/portage/package.use/lvm2
    ```
    ```bash
    emerge -q sys-fs/lvm2
    ```
    ```bash
    systemctl enable lvm2-monitor.service
    ```
    ```bash
    nano /etc/default/grub
    ```
    ```
    GRUB_CMDLINE_LINUX="net.ifnames=0 dolvm init=/usr/lib/systemd/systemd"
    ```

    ```bash
    emerge -q --config sys-kernel/gentoo-kernel
    ```
* Reconstrucción del initramfs

    ```
    emerge --config sys-kernel/gentoo-kernel-bin
    ```

## Configurar el sistema
* Crear el archivo fstab
    ```bash
    nano /etc/fstab 
    ```
    ```
    /dev/vg/vda1       /efi                    vfat    umask=0077          0 2
    /dev/vg/vda2       none                    swap    sw                  0 0
    /dev/vg/vda3       /                       xfs     defaults,noatime    0 1
    /dev/vg/usr        /usr                    xfs     defaults,noatime    0 1
    /dev/vg/home       /home                   xfs     defaults,noatime    0 1
    /dev/vg/opt        /opt                    xfs     defaults,noatime    0 1
    /dev/vg/tmp        /tmp                    xfs     defaults,noatime    0 1
    /dev/vg/var        /var                    xfs     defaults,noatime    0 1
    /dev/vg/var_tmp    /var/tmp                xfs     defaults,noatime    0 1
    /dev/vg/distfile   /var/cache/distfiles    xfs     defaults,noatime    0 1
    ```

* Información de la red
    ```bash
    hostnamectl hostname le
    ```
* Red
    ```bash
    echo 'net-wireless/wpa_supplicant dbus' >> /etc/portage/package.use/wpa_supplicant
    ```
    ```bash
    emerge -q networkmanager
    ```
    ```bash
    systemctl enable NetworkManager
    ```
* Información del sistema
    ```bash
    passwd
    ```

## Instalar las herramientas del sistema
*  Indexar Archivos
    ```bash
     emerge -q sys-apps/mlocate
    ```

* Shell de acceso remoto
    ```bash
    systemctl enable sshd
    ```

* Completado de shell
    ```bash
    emerge -q app-shells/bash-completion
    ```

* Sincronización temporal
    ```bash
    emerge -q net-misc/chrony
    ```
    ```bash
    systemctl enable chronyd.service
    ```

* Herramientas del Sistema de Archivos
    ```bash
    emerge -q sys-fs/xfsprogs sys-fs/e2fsprogs sys-fs/dosfstools
    ```

* Para el correcto comportamiento del planificador dispositivos nvme
    ```
    emerge  -q sys-block/io-scheduler-udev-rules
    ```


## Configurar el cargador de arranque
* Seleccionar un gestor de arranque
    ```bash
    echo '>=sys-boot/grub-2.12-r5 mount' >> /etc/portage/package.use/grub
    ```
    ```bash
    emerge -q sys-boot/os-prober
    ```
    ```bash
    echo 'GRUB_PLATFORMS="efi-64"' >> /etc/portage/make.conf
    ```
    ```bash
    emerge -q sys-boot/grub
    ```
* Instalación
    ```bash
    grub-install --efi-directory=/efi
    ```
* Configuración
    ```bash
    grub-mkconfig -o /boot/grub/grub.cfg
    ```

## Modo rescate


