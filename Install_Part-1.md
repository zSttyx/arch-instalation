## Prep


Descargar la iso de [Arch Linux](https://archlinux.org/download/) y bootearla en un pendrive. Arrancar la maquina desde el pendrive, y despues de un tiempito de carga, deberia aparecer una terminal.

## Primeros pasos
- Verificar que estemos en UEFI: `ls /sys/firmware/efi/efivars` no deberia estar vacio
- Sincronizamos el reloj: `timedatectl set-ntp true`.
- Actualizamos la lista de paquetes `pacman -Syyy`.

#### Creamos las particiones
Vamos a crear 2 particiones, una particion EFI y la otra que contendra el sistema de archivos /. Si quisieramos hacer otra particion para que contenga otro directorio, ejemplo, hacer una particion en otro disco para que ese tenga el /home, se puede tambien. 

Si tenemos Windows instalado, la particion EFI ya existe y no hace falta crearla. Solo habra que montarla cuando corresponda.

Ejecutamos `cfdisk /dev/sdX` donde X es el disco donde vamos a instalar Arch.

Para crear la particion EFI nos vamos a `new` e ingresamos el tamaño 512M y aceptamos. Luego nos vamos a `type` y seleccionamos EFI System.

Ahora nos posicionamos sobre el espacio libre, seleccionamos `new` y le damos todo el resto de espacio libre. Luego vamos a `type` y seleccionamos Linux Filesystem.

Finalmente seleccionamos `Write`, confirmamos los cambios ingresando `yes` y luego `quit`.

Si quisieramos hacer una particion en otro disco para que contenga el `/home`, hay que proceder igual, y al seleccionar `type`, elegimos `Linux`.


Comprobamos que las particiones se hayan creado ejecutando `lsblk`.

#### Formateamos las particiones
Para formatear la particion EFI ejecutamos `mkfs.fat -F32 /dev/sdXN`. Para la particion principal y las demas ejecutamos `mkfs.ext4 /dev/sdXM`.

#### Montamos las particiones
Montamos primero la particion principal con `mount /dev/sdXM /mnt`.

Creamos una carpeta con `mkdir /mnt/boot` y montamos la particion EFI: `mount /dev/sdXN /mnt/boot`.

Si tuvieramos otra particion para contener otra ruta como `/home` hacemos algo similar que con efi, primero creamos la carpeta `mkdir /mnt/home` y luego montamos `mount /dev/sdXR /mnt/home`

Si se quisiera montar la particion de Windows en el sistema, el procedimiento es el mismo que con `/home`

Comprobamos con `lsblk`.


#### Instalacion Base

Ejecutamos `pacstrap /mnt base linux linux-firmware vim amd-ucode`. 

Se puede cambiar `vim` por `nano`, cualquiera de los dos editores de texto estan bien.

El ultimo paquete en el comando puede cambiar. Si la maquina donde se esta instalando tiene un procesador amd, se instala `amd-ucode`, pero si es intel, el paquete que debe instalarse es `intel-ucode`. En una maquina virtual, este paquete se omite.

#### Creamos la tabla de particiones

Una vez instalados los paquetes base, creamos la tabla de particiones con `genfstab -U /mnt >> /mnt/etc/fstab`.

#### Creamos el archivo SWAP

Nos movemos a la instalacion base con `arch-chroot /mnt`. 

Creamos un archivo con `dd if=/dev/zero of=/swapfile bs=512K count=4000 status=progress`. Esto dara como resultado un archivo de 2 GB que la maquina usara para mover los procesos inactivos.

Seguido de esto, le damos permisos `chmod 600 /swapfile`, cambiamos el tipo de archivo `mkswap /swapfile`, habilitamos el archivo `swapon /swapfile`.

Finalmente, agregamos el archivo a la tabla de particiones. Editamos (con vim o nano) el archvo fstab y agregamos la linea `/swapfile none swap defaults 0 0`.

#### Configuramos idioma, localización y zona horaria

Para configurar la zona horaria ejecutamos `ln -sf /usr/share/zoneinfo/America/Buenos_Aires /etc/localtime`.

Sincronizamos el reloj del sistema con el del hardware `hwclock --systohc`.

Para configurar nuestra localizacion: editamos el archivo `/etc/locale.gen` y descomentamos las zonas que correspondan, Ej.: `es_AR.UTF-8 UTF-8`. Luego ejecutamos `locale-gen`.

Especificamos el lenguaje del sistema: `echo LANG=es_AR.UTF-8 >> /etc/locale.conf`.

Finalmente especificamos la distribucion de teclado: `echo KEYMAP=es >> /etc/vconsole.conf`.

#### Configuramos la red
Creamos el archivo hostname (el nombre de la maquina): `echo NOMBRE_QUE_QUIERAS > /etc/hostname`.

Editamos el archivo hosts y agregamos:
``` 
127.0.0.1  localhost
::1        localhost
127.0.0.1   NOMBRE_QUE_PUSISTE_EN_HOSTNAME.localdomain NOMBRE_QUE_PUSISTE_EN_HOSTNAME
```

#### Damos una contraseña a root e instalamos los paquetes que faltan

Creamos la password para root, ejecutando `passwd`.

Instalamos los paquetes finales: `pacman -Syu grub efibootmgr os-prober ntfs-3g networkmanager network-manager-applet wireless_tools wpa_supplicant dialog mtools dosfstools base-devel linux-headers git bluez bluez-utils xdg-utils`.

De los paquetes que se instalan, `grub`, `efibootmanager` y `os-prober` son necesarios para poder configurar el grub y el dual-boot si se tuviera. `ntfs-3g` es para poder leer y escribir en el sistema de archivos NTFS. `networkmanager`, `network-manager-applet`, `wireless_tools` y `wpa_supplicant` son para manejar el WiFi principalmente. `dialog` sirve para mostrar cuadros de dialogo en shell scripts, sirve para las herramientas que puedan necesitarlo, como networkmanager. `mtools` y `dosfstools` son similares a `ntfs-3g` pero para sistemas de archivos DOS. `base-devel` contiene herramientas y paquetes que son importantes para compatibilidad, al igual que `linux-headers`. instalarlos ahora ahorra tiempo despues. `git` es fundamental. `bluez` y `bluez-utils` son necesarios para manejar las conecciones bluetooth. `xdg-utils` es util para manejar desde consola los archivos multimedia.

#### Ahora instalamos grub

Para instalar grub, ejecutamos `grub-install --target=x86_64-efi --efi-directory=/boot --bootloader-id=GRUB`.

Se puede editar el archivo /etc/defaults/grub y modificar el tiempo de espera en la pantalla del grub a 0 (creo que era GRUB_TIMEOUT el parametro).

Tambien, puede que grub no detecte el loader de Windows, y cuando se instale todo y se reinicie, solo muestre Arch Linux. En casos como este, hay que ir al archivo /etc/defaults/grub y descomentar la linea que dice `GRUB_DISABLE_OS_PROBER=false`. Esto lo hago cuando me pasa que GRUB no detecta a Widnows, pero quizas se ahorra tiempo si se hace ahora.

Generamos el archivo de configuracion con: `grub-mkconfig -o /boot/grub/grub.cfg`.

Esto tiene que hacerse cada vez que se modifique la configuracion del grub, para que se actualice.

#### Ultimando detalles

Habilitamos algunos servicios:
- `systemctl enable NetworkManager`.
- `systemctl enable bluetooth`.

Creamos el usuario:
- `useradd -mG wheel USUARIO`.
- `passwd USUARIO`.

Configuramos el grupo wheel para que pueda ejecutar comandos con sudo. Para esto ejecutamos `EDITOR=vim visudo` (si instalaste `nano` reemplazalo por `vim` en el comando). Descomentamos la opcion debajo de la linea que dice `%wheel ALL=(ALL:ALL) ALL`.

Finalizamos la instalación ejecutando:
```
exit
umount -a
reboot
```

Al iniciar, deberia mostrarse el GRUB.
