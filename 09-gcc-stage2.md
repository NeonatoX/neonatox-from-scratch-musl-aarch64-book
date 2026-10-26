# Capítulo 9: GCC Stage 2 (GCC de transición hacia musl ARM64)

## 9.1 — Objetivo

Construir un GCC de transición con estas tres características:

- **Se compila** (`build`) en el host físico `aarch64-linux-musl`.
- **Se ejecuta** (`host`) en el futuro chroot `aarch64-linux-musl`.
- **Genera código** (`target`) para `aarch64-linux-musl`.

El compilador se instala en `$LFS/usr` y es el que permite terminar el
bootstrap, entrar al chroot y recompilar un GCC completamente nativo y
autohospedado (capítulo 15).

No es todavía el GCC definitivo: es el compilador de transición.

---

## 9.2 — Contexto: Canadian Cross

La combinación de triples es:

| Rol | Triple | Dónde está |
|-----|--------|------------|
| `build` | `aarch64-linux-musl` | Tu máquina física actual |
| `host` | `aarch64-linux-musl` | El chroot ARM64 |
| `target` | `aarch64-linux-musl` | Los binarios que este GCC produce |

Esto se conoce como **Canadian Cross**: `build != host` y `host == target`.

La consecuencia práctica más importante es que el compilador recién
construido (`xgcc`) es un binario **aarch64** y, por lo tanto, **no puede
ejecutarse en el host aarch64**. Por eso las bibliotecas del target
(`libgcc`, `libstdc++`, etc.) no se compilan con `xgcc`, sino con el
cross-compiler de la etapa 1 (`$LFS/tools`), que sí corre en aarch64 y
genera código aarch64.

Variables que hay que distinguir:

| Variable | Compilador | Corre en | Genera |
|----------|-----------|----------|--------|
| `CC` / `CXX` | `aarch64-linux-musl-gcc/g++` de `$LFS/tools` | aarch64 | aarch64 |
| `CC_FOR_BUILD` | `/usr/bin/gcc/g++` del host | aarch64 | aarch64 |
| `CC_FOR_TARGET` | `aarch64-linux-musl-gcc` de `$LFS/tools` | aarch64 | aarch64 |

`CC_FOR_BUILD` se usa para los programas generadores (`genattrtab`,
`genrecog`, etc.). `CC_FOR_TARGET` se usa para construir las bibliotecas
del target.

---

## 9.3 — Ajustes previos del toolchain de `$LFS/tools`

El GCC 16.1.0 de `$LFS/tools` se construyó con:

```sh
--prefix=$LFS/tools
--host=aarch64-linux-musl
--target=aarch64-linux-musl
--libdir=/usr/lib
```

Esa instalación dejó el toolchain en un estado particular que hay que
tener en cuenta al usarlo como compilador de host:

1. **`libgcc.a` quedó fuera de `$LFS/tools`**. Al usar `--libdir=/usr/lib`
   (ruta absoluta), GCC instaló `libgcc.a`, `crtbegin.o`, etc. en
   `/usr/lib/gcc/aarch64-linux-musl/16.1.0` del host. El driver los
   encuentra porque esa es su ruta interna.
2. **`libgcc_s_asneeded.so`, `libatomic_asneeded.so` y `libstdc++`** sí
   quedaron en `$LFS/tools/aarch64-linux-musl/lib64`, que **no** forma
   parte de la ruta de búsqueda del driver. Por eso hay que pasar `-B`.
3. **El sysroot del stage 1 apunta a `$LFS/tools`**, y su directorio
   `$LFS/tools/usr/include` no contiene los headers de musl (están en
   `$LFS/tools/include` y `$LFS/tools/aarch64-linux-musl/include`). Por
   eso hay que pasar `--sysroot=$LFS`.
4. **Los headers de libstdc++ del stage 1** están en
   `$LFS/tools/aarch64-linux-musl/include/c++/16.1.0`, pero el `g++`
   instalado no los busca. Para compilar C++ hay que añadirlos con
   `-isystem`.

Definimos las rutas:

```sh
export LFS=/run/media/neonatox/n27-aarch64

TOOLS_LIBDIR=$LFS/tools/aarch64-linux-musl/lib64
TOOLS_CXXINC=$LFS/tools/aarch64-linux-musl/include/c++/16.1.0
```

Prueba rápida antes de empezar:

```sh
export PATH=$LFS/tools/bin:$LFS/tools/aarch64-linux-musl/bin:/usr/bin:/bin

cat > /tmp/hello.c <<'EOF'
#include <stdio.h>
int main(void) { puts("OK"); return 0; }
EOF

aarch64-linux-musl-gcc \
  -B$TOOLS_LIBDIR/ \
  --sysroot=$LFS \
  /tmp/hello.c -o /tmp/hello

file /tmp/hello
# → ELF 64-bit LSB executable, ARM aarch64, ...
#   interpreter /lib/ld-musl-aarch64.so.1, dynamically linked
```

> **Nota**: estos ajustes son necesarios únicamente porque el toolchain de
> `$LFS/tools` se instaló con esa disposición. En el capítulo 15, el GCC
> nativo se compila ya dentro del sistema aarch64 y no necesita nada de
> esto.

---

## 9.4 — Preparar las fuentes

```sh
cd $LFS/sources
rm -rf gcc-16.1.0
tar -xf gcc-16.1.0.tar.xz
cd gcc-16.1.0

# GMP, MPFR y MPC dentro del árbol
tar -xf ../gmp-6.3.0.tar.xz  && mv gmp-6.3.0   gmp
tar -xf ../mpfr-4.2.1.tar.xz && mv mpfr-4.2.1  mpfr
tar -xf ../mpc-1.3.1.tar.gz  && mv mpc-1.3.1   mpc

mkdir build
```

---

## 9.5 — Configurar

```sh
cd $LFS/sources/gcc-16.1.0/build

export LFS=/run/media/neonatox/n27-aarch64
export PATH=$LFS/tools/bin:$LFS/tools/aarch64-linux-musl/bin:/usr/bin:/bin

TOOLS_LIBDIR=$LFS/tools/aarch64-linux-musl/lib64
TOOLS_CXXINC=$LFS/tools/aarch64-linux-musl/include/c++/16.1.0

CC="aarch64-linux-musl-gcc -B$TOOLS_LIBDIR/ --sysroot=$LFS" \
CXX="aarch64-linux-musl-g++ -B$TOOLS_LIBDIR/ --sysroot=$LFS \
  -isystem $TOOLS_CXXINC \
  -isystem $TOOLS_CXXINC/aarch64-linux-musl \
  -isystem $TOOLS_CXXINC/backward" \
CC_FOR_BUILD=/usr/bin/gcc \
CXX_FOR_BUILD=/usr/bin/g++ \
CC_FOR_TARGET="aarch64-linux-musl-gcc -B$TOOLS_LIBDIR/ --sysroot=$LFS" \
CXX_FOR_TARGET="aarch64-linux-musl-g++ -B$TOOLS_LIBDIR/ --sysroot=$LFS" \
AR=aarch64-linux-musl-ar \
RANLIB=aarch64-linux-musl-ranlib \
AR_FOR_TARGET=aarch64-linux-musl-ar \
RANLIB_FOR_TARGET=aarch64-linux-musl-ranlib \
target_configargs="--with-toolexeclibdir=/usr/lib --with-gxx-include-dir=/usr/include/c++/16.1.0" \
host_configargs="--with-gxx-include-dir=/usr/include/c++/16.1.0" \
../configure \
  --prefix=/usr \
  --build=aarch64-linux-musl \
  --host=aarch64-linux-musl \
  --target=aarch64-linux-musl \
  --with-build-sysroot=$LFS \
  --with-sysroot=/ \
  --enable-default-pie \
  --enable-default-ssp \
  --disable-nls \
  --enable-languages=c,c++ \
  --enable-shared \
  --enable-threads=posix \
  --enable-__cxa_atexit \
  --disable-libsanitizer \
  --disable-libvtv \
  --disable-multilib \
  --disable-bootstrap \
  --libdir=/usr/lib
```

### Explicación de los parámetros

| Parámetro | Motivo |
|-----------|--------|
| `--build=aarch64-linux-musl` | El sistema que ejecuta la compilación es el host aarch64 |
| `--host=aarch64-linux-musl` | El GCC instalado correrá dentro del chroot ARM64 |
| `--target=aarch64-linux-musl` | Genera código para musl ARM64 |
| `--with-build-sysroot=$LFS` | Usa el sistema en construcción durante el build |
| `--with-sysroot=/` | Dentro del chroot la raíz es `/`, no `$LFS` |
| `--enable-default-pie` | Binarios PIE por defecto |
| `--enable-default-ssp` | Stack protector por defecto |
| `--enable-languages=c,c++` | C y C++ |
| `--enable-shared` | `libstdc++` y `libgcc_s` compartidas |
| `--enable-threads=posix` | Hilos POSIX sobre musl |
| `--enable-__cxa_atexit` | Destructores C++ correctos |
| `--disable-libsanitizer` | Sanitizers no soportados en musl |
| `--disable-libvtv` | Vtable verification no disponible en musl |
| `--disable-multilib` | Solo 64 bits |
| `--disable-bootstrap` | Un solo pase (compilador de transición) |
| `--libdir=/usr/lib` | Esquema de bibliotecas de musl |

### `target_configargs` y `host_configargs`

- `target_configargs="--with-toolexeclibdir=/usr/lib ..."` fija `/usr/lib`
  como base para las bibliotecas del target. Aun así, GCC añade después
  el directorio *multilib* (`../lib64`), por lo que las bibliotecas
  acaban en `/usr/lib64`; en §9.6.1 las movemos a `/usr/lib`, que es
  donde musl las busca.
- `--with-gxx-include-dir=/usr/include/c++/16.1.0` instala los headers de
  C++ en la ruta habitual del sistema final.

---

## 9.6 — Compilar e instalar

```sh
set -o pipefail
make -j$(nproc) 2>&1 | tee build.log
```

El build no hace los tres pases del bootstrap (`--disable-bootstrap`),
pero sí compila todos los archivos fuente de GCC con el cross-compiler de
`$LFS/tools`: más de un millar de objetos, incluidos los del frontend de
C++ (`cp/*.cc`). Por eso tarda bastante y conviene dejarlo correr sin
interrupciones.

> **Consejo**: si quieres verificar el resultado antes de tocar `$LFS`,
> instala primero en un directorio temporal y revísalo. La instalación no
> necesita `root` si el destino es tuyo:
>
> ```sh
> make DESTDIR=/tmp/stage2 install
> find /tmp/stage2/usr -name 'cc1*' -o -name 'libstdc++.so*'
> ```
>
> Cuando esté correcto, cópialo a `$LFS/usr` con `root`.

La instalación definitiva debe hacerla `root`, porque escribe en
`$LFS/usr`:

```sh
sudo env PATH="$PATH" make DESTDIR=$LFS install
```

`env PATH="$PATH"` es necesario porque `sudo` reemplaza el `PATH` por su
`secure_path`. Sin él, la instalación de `libgcc` falla con:

```bash
make[2]: aarch64-linux-musl-ranlib: No such file or directory
```

### 9.6.1 — Mover las bibliotecas de `/usr/lib64` a `/usr/lib`

En aarch64, GCC calcula el directorio *multilib* del sistema como
`../lib64`. Por eso `libgcc_s`, `libstdc++`, `libatomic`, `libgomp` y
`libitm` se instalan en `$LFS/usr/lib64`, aunque se haya pasado
`--with-toolexeclibdir=/usr/lib`: ese valor se combina después con el
directorio multilib.

musl solo busca bibliotecas en `/lib`, `/usr/local/lib` y `/usr/lib`
(recordemos que en esta guía `/lib` es un enlace a `usr/lib`), así que
hay que mover las bibliotecas y eliminar `lib64`:

```sh
sudo mv $LFS/usr/lib64/* $LFS/usr/lib/
sudo rmdir $LFS/usr/lib64
```

No hay conflictos de nombres: `/usr/lib64` solo contiene las bibliotecas
del compilador. Tras el movimiento, GCC las encuentra igualmente porque
`/usr/lib` está en su lista de búsqueda (compruébalo con
`gcc -print-search-dirs`).

---

## 9.7 — Verificar

### El compilador

```sh
file $LFS/usr/bin/gcc $LFS/usr/bin/g++
```

Debe mostrar:

```bash
ELF 64-bit LSB pie executable, ARM aarch64, ...
interpreter /lib/ld-musl-aarch64.so.1
```

Y con QEMU (los binarios son aarch64, no pueden ejecutarse directamente
en el host aarch64):

```sh
qemu-aarch64 -L $LFS $LFS/usr/bin/gcc -dumpmachine
# → aarch64-linux-musl

qemu-aarch64 -L $LFS $LFS/usr/bin/gcc -print-sysroot
# → /
```

> **Ojo**: `qemu-aarch64 -L` solo sirve para consultas y para ejecutar
> binarios sueltos. Para *compilar* hace falta `chroot` y `binfmt_misc`;
> lo vemos en §9.9.

### Componentes internos

```sh
file $LFS/usr/libexec/gcc/aarch64-linux-musl/16.1.0/cc1
file $LFS/usr/libexec/gcc/aarch64-linux-musl/16.1.0/cc1plus
```

Resultado esperado:

```bash
ELF 64-bit LSB pie executable, ARM aarch64, ...
interpreter /lib/ld-musl-aarch64.so.1
```

### Bibliotecas

```sh
ls -l $LFS/usr/lib/libstdc++.so.6
readelf -d $LFS/usr/lib/libstdc++.so.6 | grep NEEDED
```

Debe depender de:

```bash
libc.so
libgcc_s.so.1
```

La `libgcc_s.so.1` queda en `$LFS/usr/lib` (después de moverla desde
`lib64` en §9.6.1) y el enlace `libgcc_s.so` lo resuelve el propio GCC.

---

## 9.8 — Sysroot y wrappers

En versiones anteriores de esta guía, el GCC de transición se configuraba
con `--with-sysroot=$LFS`. Eso obligaba a usar wrappers que añadieran
`--sysroot=/` al entrar al chroot:

```sh
exec /usr/bin/aarch64-linux-musl-gcc --sysroot=/ "$@"
```

En esta versión **ya no son necesarios**: el compilador se configura
directamente con `--with-sysroot=/`, que es la raíz correcta una vez
dentro del chroot. El `--with-build-sysroot=$LFS` se usa solo durante la
compilación.

Por tanto, no hay que crear nada en `$LFS/usr/local/bin`.

---

## 9.9 — Pruebas

`qemu-aarch64 -L $LFS` sirve para ejecutar un binario aarch64 suelto,
pero **no** para conducir a `gcc`: el compilador lanza subprocesos
(`cc1`, `as`, `collect2`) y qemu no traduce esas rutas. Si se intenta, el
resultado es:

```bash
gcc: fatal error: cannot execute '.../cc1': posix_spawn: Exec format error
```

Para compilar de verdad hay que ejecutar **dentro** del sistema, con
`binfmt_misc` registrado. Ese registro también lo necesitará el chroot
del capítulo 10.

### 9.9.1 — Registrar binfmt_misc en el host

Se hace una sola vez:

```sh
sudo install -m 755 /usr/bin/qemu-aarch64-static /usr/local/bin/

sudo sh -c 'printf "%s\n" ":qemu-aarch64:M::\x7fELF\x02\x01\x01\x00\x00\x00\x00\x00\x00\x00\x00\x00\x02\x00\xb7:\xff\xff\xff\xff\xff\xff\xff\x00\xff\xff\xff\xff\xff\xff\xff\xff\xfe\xff\xff:/usr/local/bin/qemu-aarch64-static:F" > /etc/binfmt.d/qemu-aarch64.conf'

sudo systemctl restart systemd-binfmt
```

Comprobación:

```sh
cat /proc/sys/fs/binfmt_misc/qemu-aarch64
# → enabled
```

A partir de aquí, cualquier ELF aarch64 se ejecuta de forma transparente.

### 9.9.2 — Prueba en C

Los fuentes de prueba se crean dentro de `$LFS` y se compilan con
`chroot`, porque el compilador necesita que `/usr` sea su propia raíz:

```sh
cat > $LFS/tmp/test.c <<'EOF'
#include <stdio.h>
int main(void) { puts("C OK"); return 0; }
EOF

sudo chroot $LFS /usr/bin/gcc /tmp/test.c -o /tmp/test
file $LFS/tmp/test
# → ARM aarch64, interpreter /lib/ld-musl-aarch64.so.1

sudo chroot $LFS /tmp/test
# → C OK
```

### 9.9.3 — Prueba en C++

```sh
cat > $LFS/tmp/test.cpp <<'EOF'
#include <iostream>
#include <vector>
#include <string>

int main()
{
    std::vector<std::string> v{"C++", "OK"};
    std::cout << v[0] << " " << v[1] << "\n";
    return 0;
}
EOF

sudo chroot $LFS /usr/bin/g++ /tmp/test.cpp -o /tmp/testcpp
sudo chroot $LFS /tmp/testcpp
# → C++ OK
```

Si la prueba C++ falla al ejecutarse con un error de `libstdc++.so.6`,
comprueba que las bibliotecas se hayan movido de `$LFS/usr/lib64` a
`$LFS/usr/lib` (§9.6.1) y que el archivo exista.

---

## 9.10 — Notas adicionales

### Flujo completo del bootstrap

```bash
Binutils
      ↓
GCC Cross (tools)
      ↓
musl + Binutils Stage 2
      ↓
GCC Stage 2 (este capítulo)
      ↓
chroot
      ↓
GCC completamente nativo (capítulo 15)
```

### Conclusión

Este GCC constituye el compilador de transición de NeonatoX.

Su función es proporcionar un entorno estable para acceder al chroot y
realizar la recompilación final del toolchain completamente nativo,
donde la configuración pasará a ser:

```
build=aarch64-linux-musl
host=aarch64-linux-musl
target=aarch64-linux-musl
```

---

<< [Anterior](08-musl-stage2.md) | [Índice](README.md) | [Siguiente](10-busybox.md) >>
