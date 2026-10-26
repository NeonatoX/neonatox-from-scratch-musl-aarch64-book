# Capítulo 10: BusyBox 1.37.0 (estático, musl aarch64)

## 10.1 — Objetivo

Instalar el primer conjunto de utilidades del sistema final: un BusyBox
**estático** enlazado contra musl aarch64, con 415 applets (shell, init,
coreutils, red, módulos del kernel, etc.).

Hasta ahora `$LFS` tiene compilador y libc, pero **ningún programa
ejecutable**: no hay `/bin/sh`, no hay `ls`, no hay `make`. BusyBox es lo
que convierte ese árbol de ficheros en un sistema con el que se puede
trabajar, y es la puerta de entrada al chroot:

```sh
sudo chroot $LFS /bin/sh
```

---

## 10.2 — Por qué BusyBox y por qué estático

BusyBox es un único binario que implementa cientos de comandos y se
comporta de forma distinta según el nombre con el que se le invoca. Eso
permite tener un sistema usable ocupando ~1,2 MB.

Se compila **estático** por tres razones:

1. **Es lo primero que se ejecuta.** Si dependiera de `libstdc++` o de
   rutas del loader, cualquier error de instalación dejaría el sistema
   sin shell.
2. **Sirve de rescate.** Un BusyBox estático funciona incluso si musl o
   el árbol de `/usr/lib` se rompen.
3. **No hay huevo ni gallina.** Para compilar *dentro* del chroot haría
   falta un `make` y un `/bin/sh` que todavía no existen. Por eso BusyBox
   se cross-compila desde el host con el toolchain de `$LFS/tools`.

La configuración incluida desactiva `CONFIG_PIE`, así que el binario es
un ejecutable clásico (no PIE) sin intérprete dinámico.

---

## 10.3 — Dependencias

- Cross-compiler de `$LFS/tools` (capítulo 5), con los ajustes de §9.3.
- musl 1.2.6 instalado en `$LFS/usr` (capítulo 8): `libc.a`, `crt1.o`,
  `crti.o`, `crtn.o`.
- `bzip2` en el host para extraer el tarball.
- Conectividad para descargar la fuente.

```sh
export LFS=/run/media/neonatox/n27-aarch64

export PATH=$LFS/tools/bin:$LFS/tools/aarch64-linux-musl/bin:/usr/bin:/bin
```

> **Importante**: el `PATH` necesita **los dos** directorios. Si falta
> `$LFS/tools/aarch64-linux-musl/bin`, el ensamblador que se usa es el del
> host (aarch64) y el build falla con:
>
> ```bash
> as: unrecognized option `-EL'
> ```

---

## 10.4 — El archivo de configuración

Este capítulo usa la configuración de BusyBox del proyecto
`neonatox-live-boot`, incluida en el repositorio del libro como recurso:

```
recursos/busybox-config
```

Procedencia: `neonatox-live-boot/tools/src/busybox-config`
(BusyBox 1.37.0). Es la misma configuración con la que se construye el
BusyBox de la imagen live, así que el sistema del libro y la live
comparten comportamiento.

Verificación del recurso:

```sh
sha256sum recursos/busybox-config
# 3870b6545db483ac335724f9f57b3078f5515049c7ffb7a2ae4cb85785daa084
```

### Opciones destacadas

| Opción | Valor | Efecto |
|--------|-------|--------|
| `CONFIG_STATIC` | `y` | Enlazado estático con `libc.a` |
| `CONFIG_PIE` | no | Sin PIE (binario clásico) |
| `CONFIG_STATIC_LIBGCC` | `y` | `libgcc` también estático |
| `CONFIG_INSTALL_APPLET_SYMLINKS` | `y` | Applets como symlinks a `busybox` |
| `CONFIG_LONG_OPTS` | `y` | Opciones largas (`--help`, `--color`…) |
| `CONFIG_LFS` / `CONFIG_TIME64` | `y` | Ficheros grandes y `time_t` de 64 bits |
| `CONFIG_INIT`, `CONFIG_MDEV` | `y` | `init` y gestión de `/dev` |
| `CONFIG_UDHCPC`, `CONFIG_HTTPD` | `y` | Cliente DHCP y servidor web |
| `CONFIG_FEATURE_SH_MATH` | `y` | Aritmética en `ash` |
| `CONFIG_CROSS_COMPILER_PREFIX` | `""` | Vacío: el compilador se pasa por `CC` |

---

## 10.5 — Preparar las fuentes

```sh
cd $LFS/sources

wget https://busybox.net/downloads/busybox-1.37.0.tar.bz2

tar -xf busybox-1.37.0.tar.bz2
cd busybox-1.37.0
```

> La configuración es de la versión **1.37.0**. Con otra versión,
> `make oldconfig` pedirá decidir sobre las opciones nuevas y el
> resultado puede diferir.

---

## 10.6 — Configurar

Copia el recurso como `.config`:

```sh
export BOOK=/ruta/al/neonatox-from-scratch-musl-aarch64-book

cp $BOOK/recursos/busybox-config .config
```

### 10.6.1 — Dos ajustes para aarch64

La configuración original activa dos optimizaciones que **solo existen
en x86**:

```sh
# CONFIG_SHA1_HWACCEL is not set
# CONFIG_SHA256_HWACCEL is not set
```

En aarch64 hay que desactivarlas. Si se dejan activas, BusyBox 1.37.0 no
compila:

```bash
libbb/hash_md5_sha.c:1316:35: error: 'sha1_process_block64_shaNI' undeclared
```

El motivo es que `sha1_end()` referencia la versión acelerada con
instrucciones SHA-NI sin comprobar la arquitectura. En aarch64 esas
funciones nunca se definen, así que el error es inevitable. Desactivarlas
no cambia nada funcionalmente: en ARM no se usaban.

```sh
sed -i 's/^CONFIG_SHA1_HWACCEL=y/# CONFIG_SHA1_HWACCEL is not set/'   .config
sed -i 's/^CONFIG_SHA256_HWACCEL=y/# CONFIG_SHA256_HWACCEL is not set/' .config
```

### 10.6.2 — Regenerar el config

```sh
yes "" | make oldconfig
```

`oldconfig` completa las opciones que falten con sus valores por defecto
y deja el `.config` consistente. Comprueba lo importante:

```sh
grep -E '^CONFIG_(STATIC|PIE|INSTALL_APPLET)' .config
```

Debe mostrar `CONFIG_STATIC=y`, `# CONFIG_PIE is not set` y
`CONFIG_INSTALL_APPLET_SYMLINKS=y`.

> Si quieres cambiar algo (por ejemplo, quitar `httpd` o añadir applets),
> usa `make menuconfig` antes de compilar.

---

## 10.7 — Compilar

```sh
make -j$(nproc) \
  CC="aarch64-linux-musl-gcc -B$LFS/tools/aarch64-linux-musl/lib64/ --sysroot=$LFS" \
  HOSTCC=gcc 2>&1 | tee build.log
```

Qué hace cada pieza:

| Parte | Función |
|-------|---------|
| `CC="aarch64-linux-musl-gcc …"` | Compilador que **corre en aarch64** y genera aarch64 |
| `-B$LFS/tools/aarch64-linux-musl/lib64/` | Fuerza el uso de `libgcc`/`libatomic` del toolchain, no los del host |
| `--sysroot=$LFS` | Usa `libc.a`, `crt*.o` y headers de musl stage 2 |
| `HOSTCC=gcc` | Compila las herramientas que se ejecutan en el host (`kconfig`, `fixdep`) |

El build tarda menos de un minuto con 16 hilos. Al terminar debe existir
`busybox`:

```sh
ls -l busybox
# -rwxr-xr-x … 1247112 … busybox
```

Comprobación rápida de que es estático:

```sh
file busybox
# ELF 64-bit LSB executable, ARM aarch64, statically linked, stripped
```

Si aparece `dynamically linked`, el `--sysroot` o `CONFIG_STATIC` no se
aplicaron: revisa que `.config` tenga `CONFIG_STATIC=y` y que el comando
`make` haya recibido el `CC` completo.

---

## 10.8 — Instalar

```sh
sudo make CONFIG_PREFIX=$LFS install-noclobber
```

> **Usa `install-noclobber`, no `install`.** El objetivo `install` crea los
> symlinks con `ln -fs`, es decir, **sobrescribe** cualquier archivo que ya
> exista con ese nombre. En este punto `$LFS/usr/bin` ya contiene los
> binarios reales de binutils (`ar`, `strings`, …); un `make install` normal
> los reemplazaría por applets de BusyBox y rompería el toolchain.
> `install-noclobber` es un objetivo propio de BusyBox que pasa
> `--noclobber` al script de instalación: crea los enlaces que faltan y
> respeta lo que ya existe.

`CONFIG_PREFIX` hace que BusyBox se instale en el sistema en construcción
y no en `/`:

- `$LFS/usr/bin/busybox` (el binario)
- symlinks en `bin/`, `sbin/`, `usr/bin/` y `usr/sbin/` según la tabla
  `busybox.links`

Recuerda que en esta guía `/bin` y `/sbin` son symlinks a `usr/bin`, así
que en la práctica todo queda bajo `$LFS/usr/bin`:

```sh
ls -l $LFS/usr/bin/busybox $LFS/usr/bin/sh $LFS/usr/bin/ls
# busybox          → binario
# sh    -> busybox
# ls    -> busybox
```

> **Aviso normal**: al final, `install.sh` imprime
> `You will probably need to make your busybox binary setuid root…`.
> Es un mensaje genérico para sistemas multiusuario. Aquí el sistema
> arranca como `root`, así que **no** hagas `chmod u+s` sobre `busybox`.

> **Nota**: `make install-noclobber` no ejecuta el binario aarch64; solo
> crea enlaces con `ln` del host. Por eso no hace falta `binfmt_misc` para
> instalar. Para *ejecutarlo*, sí (§10.9).

Comprueba que binutils sigue intacto:

```sh
ls -l $LFS/usr/bin/ar $LFS/usr/bin/strings
```

Deben ser archivos ELF reales, **no** symlinks a `busybox`.

### Alternativa sin `sudo`

Si prefieres no dar `root` a `make`, instala en un directorio temporal y
copia el resultado:

```sh
make CONFIG_PREFIX=/tmp/bb-root install-noclobber
sudo cp -a /tmp/bb-root/. $LFS/
```

---

## 10.9 — Verificar

### El binario

```sh
file $LFS/usr/bin/busybox
readelf -d $LFS/usr/bin/busybox
```

Resultado esperado:

```bash
ELF 64-bit LSB executable, ARM aarch64, statically linked, stripped

There is no dynamic section in this file.
```

Ese `There is no dynamic section` es la prueba definitiva de que no
depende de musl en tiempo de ejecución.

### Los applets

```sh
qemu-aarch64 $LFS/usr/bin/busybox --list | wc -l
# 415

qemu-aarch64 $LFS/usr/bin/busybox --list | head
# [
# [[
# acpid
# add-shell
# ...
```

Aquí `qemu-aarch64` sin `-L` es suficiente porque `busybox` es estático y
`busybox <applet>` ejecuta el applet dentro del mismo proceso.

### El chroot

Con `binfmt_misc` registrado (§9.9.1) ya se puede entrar al sistema:

```sh
sudo chroot $LFS /bin/sh -c 'busybox | head -1; echo "chroot OK"'
# BusyBox v1.37.0 (…) multi-call binary.
# chroot OK
```

Y un par de applets reales:

```sh
sudo chroot $LFS /bin/sh -c 'ls -l / | head -5'
sudo chroot $LFS /bin/sh -c 'uname -m; id'
```

Si `chroot` falla con `Exec format error`, falta el registro de
`binfmt_misc` del capítulo anterior.

---

## 10.10 — Qué queda instalado

Con BusyBox el sistema ya tiene, entre otros:

| Área | Ejemplos |
|------|----------|
| Shell | `sh`, `ash`, `echo`, `printf`, `test`, `[` |
| Coreutils | `ls`, `cp`, `mv`, `rm`, `cat`, `ln`, `mkdir`, `chmod` |
| Texto | `sed`, `grep`, `awk`, `cut`, `sort`, `vi` |
| Procesos | `ps`, `kill`, `top`, `free`, `uptime` |
| Red | `ip`, `ifconfig`, `udhcpc`, `wget`, `httpd`, `nc` |
| Sistema | `init`, `mdev`, `mount`, `switch_root`, `dmesg`, `sysctl` |
| Módulos | `insmod`, `rmmod`, `modprobe`, `lsmod`, `depmod` |

`init` y `mdev` todavía no se usan: el arranque se configura en un
capítulo posterior, cuando existan `/etc/inittab` y `/etc/init.d/rcS`.

---

## 10.11 — Notas

### Reinstalar o actualizar applets

Si cambias `.config` y recompilas, reinstala normalmente:

```sh
make -j$(nproc) CC="…" HOSTCC=gcc
sudo make CONFIG_PREFIX=$LFS install-noclobber
```

BusyBox no elimina los symlinks de los applets que ya no existen. Para
limpiarlos, ejecuta el script de instalación con `--cleanup --noclobber`:

```sh
sudo applets/install.sh $LFS --symlinks --noclobber --cleanup
```

### `busybox --install`

Dentro del sistema, `busybox --install -s /usr/bin` recrea todos los
symlinks **sobrescribiendo** lo que encuentre. Es útil si se perdió algún
applet, pero **no** lo uses sin mirar: reemplazaría binarios reales de
binutils (`ar`, `strings`, …) por enlaces a BusyBox.

Para recuperar solo un applet concreto, crea el enlace a mano:

```sh
ln -sf busybox /usr/bin/awk
```

### Por qué no se compiló con el GCC del capítulo 9

Porque ese GCC corre en aarch64 y dentro de `$LFS` aún no hay `make`.
BusyBox resuelve la mitad del problema (el shell), pero para construir
cualquier cosa hace falta `make`, y para tener `make` hace falta
construirlo antes. Por eso el capítulo 11 cross-compila GNU Make desde el
host; a partir de ahí todos los paquetes se compilan dentro del chroot,
de forma nativa, con el compilador de transición.

---

<< [Anterior](09-gcc-stage2.md) | [Índice](README.md) | [Siguiente](11-make.md) >>
