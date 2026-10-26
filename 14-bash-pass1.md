# Capítulo 14: Bash 5.3 (Pass 1 — primer paquete nativo)

## 14.1 — Objetivo

Instalar GNU Bash en `/usr/bin/bash` y convertirlo en el `/bin/sh` del
sistema, reemplazando al `sh` de BusyBox. Es el **primer paquete que se
compila dentro del chroot**, con el GCC nativo del capítulo 9 y el `make`
del capítulo 11.

A partir de aquí el sistema empieza a construirse a sí mismo: no se usa
`--sysroot`, no se usa el toolchain de `$LFS/tools`, y el compilador que
ejecuta el build es exactamente el que quedará en el sistema final.

---

## 14.2 — Por qué se compila dentro del chroot

El capítulo 11 dejó `make` en `/usr/bin/make`. Con `make`, el GCC nativo y
`binfmt_misc`, ya no hay razón para seguir cross-compilando desde el
host:

| Antes (capítulos 10–11) | Ahora (capítulo 14 en adelante) |
|---|---|
| Build en el host aarch64 | Build dentro de `$LFS` (aarch64) |
| `CC="aarch64-linux-musl-gcc … --sysroot=$LFS"` | `CC=gcc` (el del sistema) |
| `--host=aarch64-linux-musl` | Build nativo, sin `--host` |
| `HOSTCC=gcc` | No aplica |

**¿Por qué «Pass 1»?** Bash usa `readline` para la edición de línea
interactiva. Todavía no hay `ncurses` instalado, así que este bash se
enlaza con el `termcap` que trae incluido. Cuando el capítulo de
`ncurses` esté hecho, se puede reconstruir bash «Pass 2» con
`--with-curses` para tener un `readline` completo (historial, `Ctrl-R`,
colores). No es obligatorio para que el sistema funcione.

En LFS «Pass 1/Pass 2» existe porque el toolchain cambia entre pasadas.
Aquí el compilador ya es el definitivo, así que la segunda pasada es solo
una mejora opcional.

---

## 14.3 — Descargar y extraer (desde el host)

La descarga se hace en el host, antes de entrar al chroot: dentro de
`$LFS` todavía no hay resolución DNS configurada.

```sh
export LFS=/run/media/neonatox/n27-aarch64

cd $LFS/sources

wget https://ftp.gnu.org/gnu/bash/bash-5.3.tar.gz

sha256sum bash-5.3.tar.gz
# 0d5cd86965f869a26cf64f4b71be7b96f90a3ba8b3d74e27e8e9d9d5550f31ba

tar -xzf bash-5.3.tar.gz
```

Resultado: `$LFS/sources/bash-5.3/`, que dentro del chroot será
`/sources/bash-5.3`.

### 14.3.1 — Opcional: parches oficiales

GNU publica correcciones para bash 5.3 después de la release
(`bash53-001` … `bash53-020` a la fecha). Aplicarlas es recomendable en
un sistema que va a usarse de verdad. Se bajan y aplican **desde el
host**, antes de compilar:

```sh
cd $LFS/sources
mkdir -p bash53-patches && cd bash53-patches

wget -q -O index.html https://ftp.gnu.org/gnu/bash/bash-5.3-patches/
for p in $(grep -o 'bash53-[0-9][0-9][0-9]' index.html | sort -u); do
  wget -q "https://ftp.gnu.org/gnu/bash/bash-5.3-patches/$p"
done
rm index.html

cd ../bash-5.3
for p in ../bash53-patches/bash53-*; do
  patch -p0 -i "$p" || { echo "FALLÓ $p"; break; }
done
```

Cada parche debe aplicarse sin avisos de `Hunk #N FAILED`. Con los
parches, `bash --version` mostrará `5.3.20(1)-release` en lugar de
`5.3.0(1)-release`.

---

## 14.4 — Entrar al chroot

`$LFS/dev` está vacío. Antes de entrar hay que exponer `/dev` del host:
`configure` necesita `/dev/null`, y bash usará `/dev/tty` cuando se
ejecute de forma interactiva.

```sh
sudo mount --bind /dev $LFS/dev

sudo chroot $LFS /bin/sh
```

Ya dentro del sistema:

```sh
export PATH=/usr/bin:/usr/sbin
export PS1='(chroot) \w \$ '
cd /sources/bash-5.3
```

> El `PATH` del host no sirve dentro del chroot: apunta a directorios que
> ahí no existen. `/usr/bin:/usr/sbin` es el `PATH` correcto.

Comprobación rápida de que el entorno está completo:

```sh
gcc --version | head -1    # gcc (GCC) 16.1.0
make --version | head -1   # GNU Make 4.4.1
```

Y que BusyBox tiene enlazados los applets que usa `configure`:

```sh
for a in egrep diff awk basename tr cmp grep sed; do
  command -v "$a" >/dev/null || echo "FALTA: $a"
done
```

Si falta alguno, créalo antes de configurar:

```sh
ln -sf busybox /usr/bin/awk
```

> **Trampa de autoconf con BusyBox**: el test de `egrep` de `configure`
> compara la salida con `diff`. Si `diff` no existe, el test falla y el
> error que aparece es engañoso:
> `configure: error: no acceptable egrep could be found in $PATH`.
> No falta `egrep`: falta `diff`. Con `egrep` y `diff` enlazados, BusyBox
> pasa el test sin problema; no hace falta compilar GNU grep ni gawk antes
> de bash.

---

## 14.5 — Configurar

```sh
./configure \
  --prefix=/usr \
  --without-bash-malloc \
  --disable-nls
```

| Opción | Por qué |
|--------|---------|
| `--prefix=/usr` | Bash en `/usr/bin`, man en `/usr/share/man` |
| `--without-bash-malloc` | Usa el `malloc` de musl en vez del `malloc` propio de bash (pensado para glibc; fragmenta y es más lento) |
| `--disable-nls` | Sin catálogos de traducción: musl no trae `gettext` |

No se pasa `--host` ni `--build`: es un build nativo, `configure` detecta
solo la arquitectura. Tampoco `--with-curses`, porque `ncurses` aún no
existe; bash usará su `termcap` interno.

---

## 14.6 — Compilar

```sh
make y.tab.c
make builtins/libbuiltins.a
make -j$(nproc)
```

Las dos primeras órdenes no son decorativas: el `Makefile` de bash tiene
una **carrera** al generar `y.tab.c` y `builtins/libbuiltins.a` en
paralelo, y `make -j` falla de forma intermitente con errores del tipo
`y.tab.c: No such file`. Generándolos primero, el `-j` es seguro.

> Si preferís no pensar en esto, `make` sin `-j` compila bien siempre;
> tarda un par de minutos en vez de unos segundos.

`make y.tab.c` no regenera nada: el tarball de la release ya incluye
`y.tab.c` y `y.tab.h` generados, así que **no hace falta `bison`** dentro
del chroot. La orden solo garantiza que existan antes de lanzar el
paralelo.

El build genera `bash` en el directorio actual. No se instala todavía.

---

## 14.7 — Instalar y activar como `/bin/sh`

Dentro del chroot ya se es `root`, así que no hace falta `sudo`:

```sh
make install
```

Instala:

- `/usr/bin/bash` (el binario)
- `/usr/bin/bashbug` (script de reporte de bugs)
- `/usr/share/man/man1/bash.1`, `bashbug.1`

Ahora el shell por defecto:

```sh
ln -sf bash /bin/sh
```

`/bin` es un symlink a `usr/bin`, así que esto crea
`/usr/bin/sh -> bash` y reemplaza el `sh` de BusyBox (que seguirá
disponible como `busybox sh`).

> Al invocarse como `sh`, bash arranca en **modo POSIX**
> automáticamente. Los scripts con `#!/bin/sh` siguen funcionando igual,
> pero ahora los interpreta bash.

---

## 14.8 — Verificar

### Dentro del chroot

```sh
bash --version | head -1
# GNU bash, version 5.3.0(1)-release (aarch64-unknown-linux-musl)

readlink -f /bin/sh
# /usr/bin/bash

sh -c 'echo $BASH_VERSION; set -o | grep posix'
# 5.3.0(1)-release
# posix          	on

bash -c 'echo "hola desde bash: $((2+2))"'
# hola desde bash: 4
```

`sh -c` mostrando `BASH_VERSION` y `posix on` demuestra que `/bin/sh` ya
es bash y que entra en modo POSIX.

### Desde el host

Antes de salir del chroot, en otra terminal:

```sh
file $LFS/usr/bin/bash
```

```bash
ELF 64-bit LSB pie executable, ARM aarch64, version 1 (SYSV),
dynamically linked, interpreter /lib/ld-musl-aarch64.so.1, …
```

```sh
readelf -d $LFS/usr/bin/bash | grep NEEDED
# (NEEDED)  Shared library: [libc.so]
```

Solo `libc.so`: readline y termcap van dentro del binario, y `libm` forma
parte de musl.

```sh
qemu-aarch64 -L $LFS $LFS/usr/bin/bash --version | head -1
# GNU bash, version 5.3.0(1)-release (aarch64-unknown-linux-musl)
```

> El triplet dice `musl`: el `config.guess` de bash 5.3 detecta musl por
> `__DEFINED_va_list`, que musl define en `bits/alltypes.h`. No es
> cosmético: el mismo triplet aparece en `gcc -dumpmachine` desde el
> capítulo 9, y confirma que todo el sistema es coherente.

> **No existe `ldd` en musl.** Cualquier guía que verifique con
> `ldd $LFS/usr/bin/bash` va a fallar. Usa `readelf -d` o `file`.

---

## 14.9 — Salir y desmontar

```sh
exit

sudo umount $LFS/dev
```

En los próximos capítulos habrá que volver a montar `/dev` (y
probablemente `/proc` y `/sys`) antes de entrar al chroot. Es normal: el
`chroot` comparte el kernel, pero no los sistemas de archivos virtuales.

---

## 14.10 — Notas

### ¿Y un bash estático de emergencia?

No hace falta: BusyBox ya es el shell estático de rescate (capítulo 10).
Si aun así se quisiera uno, se configura con `--enable-static-link` y
`LDFLAGS=-static`, pero pierde los builtins cargables y no aporta nada
que BusyBox no cubra.

### Espacio

Bash ocupa alrededor de 1,5 MB (sin `strip`). `make install` también
deja las páginas de manual; si más adelante se busca un sistema mínimo,
se pueden borrar sin romper nada. `strip /usr/bin/bash` lo reduce a
menos de 1 MB.

### `bashbug`

`bashbug` invoca un editor y un cliente de correo. En este sistema no
sirve de mucho; se puede borrar sin consecuencias:

```sh
rm -f /usr/bin/bashbug /usr/share/man/man1/bashbug.1
```

---

<< [Anterior](13-chroot-musl.md) | [Índice](README.md) | [Siguiente](15-gawk.md) >>
