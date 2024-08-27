Descargar el Kernel 

```bash
wget https://cdn.kernel.org/pub/linux/kernel/v5.x/linux-5.15.tar.xz
tar -xvf linux-5.15.tar.xz
cd linux-5.15
```

Configurar por defecto 

```bash
make defconfig
```

Compilar usando todos los núcleos del procesador
```bash
make -j$(nproc)
```

Esto generará el archivo del kernel (bzImage) en arch/x86_64/boot/bzImage.


