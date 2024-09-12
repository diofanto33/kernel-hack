## Para probar modulos en Ram

Crear un directorio para el proyecto, nos referiremos a este como ***raiz***

```mkdir project```

En la raiz, descargar el kernel en su version deseada (con o sin meterle mano) compilarlo hasta optener el **bzImage**
 
```bash
wget https://cdn.kernel.org/pub/linux/kernel/vx.y/linux-x.yz.tar.xz
tar -xvf linux-x.yz.tar.xz
cd linux-x.yz
```

Configurar por defecto 

```bash
make defconfig
```

Compilar usando todos los núcleos del procesador
```bash
make -j$(nproc)
```

Esto generará el archivo del kernel **bzImage** en ***arch/x86_64/boot/bzImage***.

Entonces creas un **initramfs**.
  
  
  Para armar un **initramfs** vamos a usar **BusyBox** que es un conjunto de utilidades que se pueden usar en un sistema linux.

En la raiz del proyecto descargar y Compilar **BusyBox**
```bash 
wget https://busybox.net/downloads/busybox-x.y.z.tar.bz2
tar -xvf busybox-x.y.z.tar.bz2
cd busybox-x.y.z
```

Configura **BusyBox** para que se compile en modo estático:
```bash
make defconfig
make menuconfig
```
En el menú de configuración, fijarse que esté activada la opción "Build BusyBox as a static binary (no shared libs)" en Settings -> Build options.

Ojo **BussyBox** levanta una gui al hacer ```make menuconfig``` en arch crudo puede tener problemas con la Lib de C *ncurses*. [La Solución en este enlace](https://bbs.archlinux.org/viewtopic.php?id=295859)

Compila **BusyBox** usando todos los núcleos del procesador, luego instalar 
```bash
make -j$(nproc)
```
Luego instalar
```bash 
make install
```
Esto instalará **BusyBox** en el directorio ***_install***.

Ahora estamos para crear el **initramfs**;

En raiz creamos un directorio llamado **initramfs**
```bash 
workspace/
├── busybox-x.y.y
├── hello-world
├── initramfs
└── linux-x.y.z
4 directories
```
Creamos una estructura para el **initramfs** y copiamos **BusyBox** 

```bash 
mkdir -p initramfs/{bin,sbin,etc,proc,sys,usr/{bin,sbin},lib/modules}
cp -a busybox-x.y.z/_install/* initramfs/
```
Se supone que ya tenemos nuestro modulo de driver en binario **joe_doe.ko** entonces lo copiamos a ***lib/modules***

```bash
cp /joe/doe/module/joe_doe.ko initramfs/lib/modules/
```

Crea un <u>script</u> llamado init en el directorio raíz del **initramfs** que será ejecutado al arrancar el sistema
```bash
echo '#!/bin/sh
mount -t proc none /proc
mount -t sysfs none /sys
insmod /lib/modules/joe_doe.ko
dmesg
exec /bin/sh
' > initramfs/init
```

Dar permisos de ejecución al script init
```bash
chmod +x initramfs/init
```
Crear el Archivo Initramfs
```bash
cd initramfs
find . | cpio -H newc -o | gzip > ../initramfs.igz
cd ..
```


Ahora podemos probar el kernel con el initramfs que creamos, para eso usamos qemu, 
  en la terminal

En la raiz del proyecto lanzamos el comando
```bash 
qemu-system-x86_64 -kernel linux-5.15/arch/x86_64/boot/bzImage \
                   -initrd initramfs.igz \
                   -append "console=ttyS0" \
                   -nographic
```

Esto levantará una máquina virtual con el kernel que se compiló y el initramfs que se creó.
Si todo está bien, debería ver el mensaje de dmesg que se puso en el script init.
