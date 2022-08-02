# GUÍA: Instalación de Arch Linux paso a paso.

Para instalar arch linux vamos a necesitar de bootear un USB, o realizarlo por medio de un sistema virtual (Ej: gnome boxes, virtualbox, etc). Podemos descargar la ISO por medio de su centro de descargas: [archlinux/descargas](https://archlinux.org/download/).

## Configurar teclado
La configuración más esencial es la configuración del teclado, dado que a partir de este momento todo va ser instalado por medio de la terminal de comandos. Por defecto el instalador usa la versión americana, de teclado `us`. Si quieres ver el listado de teclados disponibles es el siguiente comando:
```bash
ls /usr/share/kbd/keymaps/**/*.map.gz
```
Es así que tenemos el directorio `usr/` viene del nombre *User System Resources/Recursos del sistema de usuario*, y aquí se encuentran la mayor parte de comandos y archivos ejecutables, además de documentación. Como dato curiosos en el pasado, no existía `usr/home/tu_usuario`, si no era `/usr/`. Además este directorio debe ser sólo de lectura, pero en subdirectorios como `/usr/share/applications` con permisos de root se puede modificar, dado que no afecta críticamente al sistema. 

Por otro lado una subcarpeta importante es `usr/bin`, donde se encuentran los comandos de UNIX, no son considerados esenciales para su funcionamiento, por ello se encuentran en el `usr` (conclusión usr relativos a programas).

Con el comando `localectl status` podemos ver nuestra plantilla actual de configuración de la distribución del teclado. Todo esto se puede cambiar manualmente en el directoriol `etc/locale.conf`. En `usr/share` están datos independiente de qué estructura de sistema se esté usando (por ejemplo la configuración del teclado). 

El paquete `kbd` contiene ficheros de mapas de teclado y utilidades para este. El teclado por defecto es el US americano. En el caso de la instrucción para listar los mapas de teclado usamos el `/**/*.map.gz` para buscar `**/` dentro de directorios/subdirectorios coincidencias con ficheros de extensión `*.map.gz`. 

Ahora para determinar nuestro teclado usamos el comando `loadkeys` y seguido del nombre del del `.map.gz`, sin añadir la arquitectura ej (*i386-la-latin1.map.gz*), sería:
```bash
loadkeys la-latin1
```
Ojo que no es necesario colocar la ruta del archivo, ni la extensión (aunque si se hace igual funciona el comando). Ojo que ademas podemos cambiar la fuente de nuestra consola, el listado de disponibles vienen con la descarga de el paquete `kbd`, y se encuentran en `/usr/share/kbd/consolefonts/` y tienen la extensión `psf.gz` ó `psfu.gz`, donde son de formato bitmap, es decir está formado de píxeles el glifo (mapa de bits), que han sido comprimidos con `gz`. Usamos el comando `setfont`, si no se pasa ninguna fuente se coloca la default.
```bash
setfont /usr/share/kbd/consolefonts/ter-i14n.psf.gz
```

## Configuración de red
Para continuar con la instalación será necesario conectarse a internet, es por ello que lo primero es verificar que dispositivos están conectados a través de PCI, usando la utilidad de `lspci`. Con la opción `lspci -k` podremos mostrar en pantalla los dispositivos que estén conectados por PCI y a la vez interactúen con alguna parte del kernel de nuestro linux, como lo es el driver de internet (*Network*). En este caso me aparece el módulo (*Ethernet controller*).

Ojo que para continuar necesitamos actualizar nuestra fecha y hora para que a la hora de descargar paquetes no exista ningún problema. Usamos el comando:
```bash
timedatectl set-ntp true
```
El `set-ntp` recibe un booleano, sincroniza la hora con los repositorios de internet. Como dato curioso con este comando te das cuenta que estas sobre el INIT system (primer proceso que se inicia durante el arranque del kernel) basado en `systemd`, dado que termina en *ctl*, lo cual hace referencia a `Control Temporal Logic`. En caso quieras saber que INIT tienes, puedes hacerlo con el comando:
```bash
/sbin/init --version
```
En caso de tener wifi acudir a esta [guía](https://wiki.archlinux.org/title/Iwd#iwctl)

## Configurar servidores de paquetes
Ahora con el script `reflector` vamos a solicitar usar los mejores servidores de réplica, y este script va a sobreescribir lo existente en `etc/pacman.d/mirrorList`. Recordar que los servidores espejo (mirror) son servidores que contienen una réplica exacta de un sitio web, que facilita poder descargarlo de la locación más cercana.
```
# Instalamos reflector
sudo pacman -Sy reflector
```
Ahora configuramos para obtener los mejores 30 paquetes, reduciendo el tiempo de búsqueda y descarga. Ademas le pedimos que retornemos como mínimo 20 mirrors, además de que se ordene por puntaje y por último guardar en `/etc/pacman.d/mirrorlist` 
``` 
reflector --protocol http --latest 30 --number 20 --sort rate --save /etc/pacman.d/mirrorlist
```
Despues de un momento podemos ver los paquetes entrando a mirrorlist. Puede tomar su tiempo el agregarlos.

## Configurar zona horaria
Ahora para configurar nuestra hora, usamos el comando `timedatectl`, el cual nos mostrará por defecto nuestra configuración. Podemos ver la lista de horarios disponibles con `timedatectl list-timezones`, donde deberemos buscar la que querramos, y con el comando `timedatectl set-timezone "America/Lima"`, colocar entre comillas la que desee. Ojo que tiene que reemplazar con la elegida en la lista explicada anterior, con la mayúscula, la diagonal y entre comillas.
```
timedatectl set-timezone "America/Lima"
```

## Particionado del disco
Para particionar el disco, es decir dividir el espacio en diferentes secciones como (SWAP, EFI-FAT32, HOME, ROOT). En el caso que usemos una partición **MBR** usaremos el comando `fdisk -l` para listar las particiones, mientras que con una GPT sería `gdisk -l`. Ojo que los dispositivos `dev/loop*` son dispositivos virtuales (no fisicos) que sirven para montar archivos de imagenes, los snaps crean un dispositivo virtual `dev/loop`.

Al listar nuestros discos, ya sea con una u otra opción veremos que se desplegan los discos, sumado a su nombre y tamaño. En este caso usaremos el espacio `/dev/sda`. Para entrar dentro usamos el comando:
```bash
fdisk /dev/sda
```
Esto nos permitirá trabajar dentro del disco duro seleccionado: `/dev/sda`. Por otro lado tenemos distintas opciones que podemos ver ingresando el comando `m`. Creamos una nueva partición primaria usando el comando `n` y posteriormente `p`, seguido la seleccionamos como partición 1. 

Ahora la primera partición que crearemos será de nuestra carpeta raíz, es decir el `./`, mientras que la otra será el `home`, donde están todos nuestros datos personales. Mientras que la tercera será una partición extendida del home, y será el `swap`, es decir el área de intercambio. 

Para cambiar el tipo de partición se hace con `t` y recibe el código de tipo de almacenamiento, colocando posteriormente `L` se puede ver un listado de los disponibles. Una vez finalizado, se debe de designar la partición de arranque, se puede colocando `p` y después el número de la partición, es este caso 1.

Para salir del instalador pulsamos la tecla `w`, seguido a ello configuramos el swap con el comando `mkswap /dev/sda6`, donde `/dev/sda6` se cambia por el numero de la partición swap que tengas. Ahora para activar el swap lo hacemos con el comando:
```bash
swapon
```
Ahora tenemos que formatear las otras particiones (root y home) para que tengan el formato ext4, donde el `*` se reemplaza por el numero de partición de tu home y root. 
```bash
mkfs.ext4 /dev/sda*
```

Ahora todas las particiones están listas, solo falta montarlas, en este primer caso la partición raíz no podremos montarla, dado que está siendo utilizada por el instalador de Arch, es por ello que procederemos a montarla provisionalmente en `/mnt`, sumado a ello en el caso de la partición home al no existir una carpeta tendremos que crearla utilizando el comando `mkdir /mnt/home`.

## Instalar Arch Linux
Para proceder con la instalación de arch linux vamos a utilizar el comando `pacstrap`, que sirve para crear una instalación desde cero con los paquetes que le indiquemos, ademas debemos incluir el directorio raíz, en este caso `/mnt`, además de la flag `base` que hace referencia al sistema base, además el kernel (linux) y los drivers `linux-firmware`, podemos añadir mas software que se desee.
```bash
pacstrap /mnt base linux linux-firmware grub networkmanager dhcpcd nano wpa_supplicant dialog man-pages
```
Ojo! En caso de que tire un error a la hora de llamar el comando es importante recargar las llaves de arch con los comandos: `pacman -Syy` y `pacman -S archlinux-keyring`

Ahora ya está instalado el sistema, lo que toca es 
```
genfstab /mnt >> /mnt/etc/fstab
```
Posteriormente nos movemos a la distribución arch como tal usando `arch-chroot /mnt`, de esta forma todos los comandos se harán sobre nuestra nueva distro.

## Configurar nuestro Arch Linux
1. Zona horaria:
   ```
   ln -sf /usr/share/zoneinfo/America/Lima /etc/localtime
   ```
2. Idioma:
   ```
   locale-gen
   echo "LANG=es_PE.UTF-8" >> /etc/locale.conf
   ```
   Debemos de editar el archivo /etc/locale.gen y quitar el # del idioma que hemos seleccionado anteriormente. Para concluir ejecutamos nuevamente `locale-gen`.
3. Distribución de teclado:
    ```
    echo "KEYMAP=la-latin1" > /etc/vconsole.conf
    ```
4. Usuario de la computadora:
    ```
    echo "mezora" > /etc/hostname
    ```
5. Red local de host:
    ```
    echo "127.0.0.1 localhost" > /etc/host
    ```

## Crear usuarios y contraseñas:
Debemos crear la contraseña para root base, para ello vamos a colocar `passwd` y seguido la contraseña. Luego para crear un usuario usamos `useradd` y agregamos la flag `-m` para crear todos los ficheros. Ojo que el nombre no debe tener espacios. Para concluir le asignamos una contraseña con `passwd`, seguido del usuario: `passwd mezora`.

## Configurar GRUB/Arranque:
Para poder arrancar el sistema tendremos que realizar la configuración del grub, de otro modo no arrancará. Usamos el comando `grub-install` y seleccionamos el disco, ya que este no se actualiza en ninguna partición. Si en caso no tenemos exito con el comando podemos usar `pacman -Sy grub`.

Por ultimo creamos el archivo de arranque para configurar el grub, con el comando `grub-mkconfig –o /boot/grub/grub.cfg`. Por ultimo actualizamos la imagen de arranque con `mkinitcpio -P`. 

## Primeros pasos Arch Linux
Una vez ya tenemos instalada nuestro sistema, toca activar los servicios, empezamos por el de internet haciendo uso de `systemctl start`, ojo que tenemos que ser root, usando `su`.
```
systemctl start NetworkManager.service
```
