# Capítulo 11: GNU Make 4.4.1 (bootstrap desde el host)

## 11.1 — Objetivo

Instalar `make` en `$LFS/usr/bin`, enlazado dinámicamente contra el musl
del sistema final. Es la última pieza que se compila **desde el host**:
con `make` y BusyBox instalados, todos los paquetes siguientes se
compilan dentro del chroot.

Hasta ahora el sistema tiene shell (capítulo 10) y compilador (capítulo
9), pero ningún programa capaz de leer un `Makefile`. Sin `make` no se
puede construir el resto del sistema; y sin `make` tampoco se puede
construir `make` a sí mismo. La solución es cross-compilarlo una vez con
el toolchain de `$LFS/tools`, exactamente como se hizo con BusyBox.

---

## 11.2 — Estático o dinámico

BusyBox es estático para poder rescatar el sistema si algo se rompe.
`make` no necesita ese papel: se enlaza **dinámicamente** contra
`$LFS/usr/lib/libc.so`, que es el musl definitivo. Es además una prueba
real de que el loader (`/lib/ld-musl-aarch64.so.1`) y las rutas del
sistema están bien antes de empezar a construir dentro del chroot.

Si se quisiera estático, bastaría añadir `LDFLAGS=-static` al
`configure`; no es lo que haremos.

---

## 11.3 — Dependencias y entorno

- Cross-compiler de `$LFS/tools` (capítulos 3 y 5), con los ajustes de
  §9.3.
- musl 1.2.6 en `$LFS/usr` (capítulo 8): `libc.so`, `libc.a`, `crt*.o`.
- BusyBox instalado (capítulo 10), para poder entrar luego al chroot.
- `binfmt_misc` registrado (§9.9.1), para ejecutar los binarios aarch64
  del sistema en el host aarch64.

```sh
export LFS=/run/media/neonatox/n27-aarch64

export PATH=$LFS/tools/bin:$LFS/tools/aarch64-linux-musl/bin:/usr/bin:/bin
```

Igual que en el capítulo 10: si falta
`$LFS/tools/aarch64-linux-musl/bin` en el `PATH`, el driver usa el `as`
del host y el build falla con `as: unrecognized option '-EL'`.

---

## 11.4 — Preparar las fuentes

```sh
cd $LFS/sources

wget https://ftp.gnu.org/gnu/make/make-4.4.1.tar.gz

sha256sum make-4.4.1.tar.gz
# dd16fb1d67bfab79a72f5e8390735c49e3e8e70b4945a15ab1f81ddb78658fb3

tar -xzf make-4.4.1.tar.gz
cd make-4.4.1
```

4.4.1 es la última versión estable de GNU Make.

---

## 11.5 — Parchear gnulib para musl + GCC 16

GNU Make trae una copia de **gnulib** (glob, fnmatch, getopt) con
declaraciones de función en estilo K&R, sin prototipo:

```c
extern char *getenv ();
extern int getopt ();
```

En C23 —el estándar por defecto de GCC 16— unos paréntesis vacíos
significan **cero argumentos**, no «argumentos desconocidos». La
declaración entra en conflicto con la de musl (`getenv(const char *)`,
`getopt(int, char *const *, const char *)`) y el build falla con:

```bash
error: conflicting types for 'getenv'; have 'char *(void)'
```

Hay que prototipar las tres declaraciones:

```sh
sed -i 's/extern char \*getenv ();/extern char *getenv (const char *);/' \
  lib/fnmatch.c src/getopt.c

sed -i 's/extern int getopt ();/extern int getopt (int, char *const *, const char *);/' \
  src/getopt.h
```

### Por qué `lib/fnmatch.c` también

Al cross-compilar, `configure` no puede ejecutar programas aarch64, así
que decide si usar el `glob` del sistema con una comprobación de
compilación que incluye `<gnu-versions.h>`. musl no tiene ese header: la
comprobación falla y Make usa su **gnulib glob/fnmatch** en lugar del de
la libc. Por eso `lib/fnmatch.c` se compila y también necesita el parche.

`src/getopt.c` y `src/getopt.h` se compilan siempre, sin condición.

Comprobación de que los parches se aplicaron:

```sh
grep -n "getenv (const char" lib/fnmatch.c src/getopt.c
grep -n "getopt (int, char" src/getopt.h
```

Deben aparecer tres líneas. Si un `sed` no imprime nada, la versión de la
fuente no es la esperada: no continúes.

---

## 11.6 — Configurar

```sh
CC="aarch64-linux-musl-gcc -B$LFS/tools/aarch64-linux-musl/lib64/ --sysroot=$LFS" \
./configure \
  --prefix=/usr \
  --host=aarch64-linux-musl \
  --disable-nls
```

Qué hace cada opción:

| Opción | Función |
|--------|---------|
| `CC="… --sysroot=$LFS"` | Compila y enlaza contra el musl de `$LFS`, no el del host |
| `-B$LFS/tools/aarch64-linux-musl/lib64/` | Usa el `libgcc` del toolchain |
| `--host=aarch64-linux-musl` | Activa el **modo cross**: `configure` no intenta ejecutar los binarios que compila |
| `--disable-nls` | Sin catálogos de traducción (musl no trae `gettext`) |

`--host` se normaliza a `aarch64-unknown-linux-musl`; es lo correcto.

> **No** se pasa `--target`. Make no es un compilador: `--target` se
> acepta pero se ignora. Lo que importa es `--host`.

Durante el `configure` es normal ver mensajes como
`checking whether we are cross compiling... yes` o
`cannot run C compiled programs`. En modo cross no se ejecuta nada
compilado, así que esos avisos no son errores.

---

## 11.7 — Compilar

```sh
make -j$(nproc) 2>&1 | tee build.log
```

Tarda menos de un minuto. Al terminar debe existir `make` en el
directorio de build:

```sh
ls -l make
# -rwxr-xr-x … 2xxxxx … make
```

---

## 11.8 — Instalar

```sh
sudo env PATH="$PATH" make DESTDIR=$LFS install
```

`DESTDIR=$LFS` instala en el sistema en construcción conservando las
rutas reales (`/usr/bin/make`), de modo que los binarios no contienen
referencias a `$LFS`.

`env PATH="$PATH"` evita el problema del capítulo 9: `sudo` puede
sustituir el `PATH` por su `secure_path`, y `make install` necesita
encontrar las herramientas del build.

Resultado:

```sh
ls -l $LFS/usr/bin/make
# -rwxr-xr-x 1 root root … /run/media/neonatox/n27-aarch64/usr/bin/make
```

También se instalan las páginas de manual en `$LFS/usr/share/man`. No
estorban.

---

## 11.9 — Verificar

### El binario

```sh
file $LFS/usr/bin/make
```

```bash
ELF 64-bit LSB pie executable, ARM aarch64, dynamically linked,
interpreter /lib/ld-musl-aarch64.so.1, …
```

Ese `interpreter /lib/ld-musl-aarch64.so.1` es lo que queremos: el
loader que existe en el sistema final.

```sh
readelf -d $LFS/usr/bin/make | grep NEEDED
# (NEEDED)  Shared library: [libc.so]
```

Solo `libc.so`. Si apareciera `libgcc_s.so.1`, el enlace no usó la
variante *as-needed* del toolchain: revisa el `-B` y el `CC` completo.

### Ejecutarlo

En el host **no** se puede ejecutar directamente; es un binario aarch64:

```sh
$LFS/usr/bin/make --version
# bash: … cannot execute binary file: Exec format error
```

Con QEMU y el `--version` (que no lanza subprocesos) alcanza:

```sh
qemu-aarch64 -L $LFS $LFS/usr/bin/make --version
# GNU Make 4.4.1
# Built for aarch64-unknown-linux-musl
```

Y dentro del sistema real, con `binfmt_misc` ya registrado:

```sh
sudo chroot $LFS /usr/bin/make --version
# GNU Make 4.4.1
```

### Prueba funcional sin compilar

Un `Makefile` mínimo, que solo ejecuta `echo`, comprueba que `make`
encuentra `/bin/sh` y ejecuta recetas dentro del chroot:

```sh
sudo chroot $LFS /bin/sh -c '
  mkdir -p /tmp/mk && cd /tmp/mk &&
  printf "all:\n\t@echo make-funciona\n" > Makefile &&
  make'
# make-funciona
```

> `qemu-aarch64 -L $LFS` **no** sirve para esto: `make` lanzaría `/bin/sh`
> con `execve`, y QEMU no traduce los procesos hijos. Para recetas hace
> falta `chroot` + `binfmt_misc` (§9.9).

---

## 11.10 — A partir de aquí: todo dentro del chroot

Este es el punto de inflexión del libro. Desde el capítulo 12, los
paquetes se compilan **dentro** de `$LFS`, con el GCC nativo del capítulo
9 y sin `--sysroot`:

```sh
sudo chroot $LFS /bin/sh

# ya dentro del sistema
export PATH=/usr/bin:/usr/sbin
cd /sources/make-4.4.1
make --version
```

Dentro del chroot:

- `gcc` es `/usr/bin/gcc` (aarch64), con `as`/`ld` de binutils stage 2.
- Los headers están en `/usr/include` y las librerías en `/usr/lib`.
- `/bin`, `/sbin` y `/lib` son symlinks a `usr/bin` y `usr/lib`.
- `$LFS/sources` aparece como `/sources`.

El toolchain de `$LFS/tools` ya no se usa para construir paquetes del
sistema; queda solo como referencia y para rehacer BusyBox o Make si
alguna vez hay que volver a arrancar desde el host.

> **No borres `$LFS/tools` todavía.** Algunos capítulos de limpieza lo
> eliminan al final, cuando el sistema es autocontenido y arranca solo.

---

<< [Anterior](10-busybox.md) | [Índice](README.md) | [Siguiente](12-bash-pass1.md) >>
