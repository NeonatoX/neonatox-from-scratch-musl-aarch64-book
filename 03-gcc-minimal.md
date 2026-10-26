# Capítulo 3: GCC 16.1.0 Pass 1 (mínimo, C estático)

## 3.1 — Objetivo

Compilar un GCC mínimo (solo lenguaje C, estático, sin threads) que sirva
exclusivamente para compilar musl. Este GCC no produce binarios dinámicos
ni tiene C++. Es una herramienta temporal.

## 3.2 — Dependencias

- Binutils 2.46.1 en `$LFS/tools`
- Linux API Headers en `$LFS/tools`

### 3.2.1 — Dependencias extra de GCC (GMP, MPFR, MPC)

GCC necesita las bibliotecas GMP, MPFR y MPC. Las incluimos dentro del árbol
de fuentes de GCC para que se compilen automáticamente:

```bash
cd $LFS/sources

# GMP
wget https://gmplib.org/download/gmp/gmp-6.3.0.tar.xz

# MPFR
wget https://ftp.gnu.org/gnu/mpfr/mpfr-4.2.1.tar.xz

# MPC
wget https://ftp.gnu.org/gnu/mpc/mpc-1.3.1.tar.gz
```

---

## 3.3 — Preparar fuentes

```bash
cd $LFS/build
tar -xf $LFS/sources/gcc-16.1.0.tar.xz
cd gcc-16.1.0

# GMP, MPFR, MPC dentro del árbol
tar -xf ../gmp-6.3.0.tar.xz && mv gmp-*/ gmp
tar -xf ../mpfr-4.2.1.tar.xz && mv mpfr-*/ mpfr
tar -xf ../mpc-1.3.1.tar.gz && mv mpc-*/ mpc

```

---

## 3.4 — Preparar headers donde GCC los espera

GCC con `--target=aarch64-linux-musl --with-sysroot=$LFS/tools` busca headers
en `$LFS/tools/aarch64-linux-musl/include/`. Creamos un enlace simbolico ahí:

```bash
ln -svf ../usr/include/ $LFS/tools/aarch64-linux-musl/include

```

## 3.5 — Compilar GCC pass 1

```bash
cd $LFS/build/gcc-16.1.0
rm -rf build
mkdir build && cd build

../configure \
  --prefix=$LFS/tools \
  --build=$(../config.guess) \
  --host=$(../config.guess) \
  --target=aarch64-linux-musl \
  --with-sysroot=$LFS/tools \
  --libdir=/usr/lib \
  --without-headers \
  --with-newlib \
  --disable-nls \
  --enable-languages=c \
  --disable-shared \
  --disable-threads \
  --disable-libatomic \
  --disable-libgomp \
  --disable-libquadmath \
  --disable-libssp \
  --disable-libvtv \
  --disable-libstdcxx
make -j$(nproc)
make install
```

### Explicación de flags

| Flag | Significado |
|------|-------------|
| `--build=$(../config.guess)` | Máquina que HACE la compilación (host con glibc) |
| `--host=$(../config.guess)` | Máquina que CORRE el compilador (host, no musl) |
| `--target=aarch64-linux-musl` | Máquina para la que GENERA código (musl) |
| `--without-headers` | No necesita headers del target (musl no existe aún) |
| `--with-newlib` | Desactiva partes de libgcc que necesitan una libc completa |
| `--enable-languages=c` | Solo C, nada de C++ aún |
| `--disable-shared` | Estático (no necesita libc.so todavía) |
| `--disable-threads` | Sin hilos (musl no está listo aún) |
| `--disable-lib*` | Desactiva todas las librerías extra |

### Verificar

```bash
$LFS/tools/bin/aarch64-linux-musl-gcc --version
# → gcc (GCC) 16.1.0

# Debe ser un binario estático (no depende de nadie)
ldd $LFS/tools/bin/aarch64-linux-musl-gcc
# → "not a dynamic executable" o solo linux-vdso
```

---

## 3.6 — Propósito de este GCC

Este GCC es **mínimo y estático**. Solo sirve para:

1. Compilar musl 1.2.6 (siguiente capítulo)
2. Nada más

No produce binarios dinámicos. No tiene C++. No tiene threads. Es una
herramienta descartable que se reemplazará después.

---

<< [Anterior](02-binutils.md) | [Índice](README.md) | [Siguiente](04-musl-dinamico.md) >>
