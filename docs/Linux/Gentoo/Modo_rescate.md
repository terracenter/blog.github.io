#### Montaje 
```bash
swapon /dev/vda2                && \
mkdir -p /mnt/gentoo            && \
cd /mnt/gentoo/                 && \

mount /dev/vda3 /mnt/gentoo     && \
#mkdir -p /mnt/gentoo/{home,opt,tmp,usr,var} && \
mount /dev/vg/home /mnt/gentoo/home && \
mount /dev/vg/opt /mnt/gentoo/opt && \
mount /dev/vg/tmp /mnt/gentoo/tmp && \
mount /dev/vg/usr /mnt/gentoo/usr && \
mount /dev/vg//var /mnt/gentoo/var && \
#mkdir -p /mnt/gentoo/var/{cache/distfiles,tmp} && \
mount /dev/vg/distfiles /mnt/gentoo/var/cache/distfiles && \
mount /dev/vg/var_tmp /mnt/gentoo/var/tmp
```

* Permisos de /tmp y /var/tmp
    ```bash
    chmod 1777 /mnt/gentoo/tmp && \
    chmod 1777 /mnt/gentoo/var/tmp
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

    mount /dev/vda1 /efi