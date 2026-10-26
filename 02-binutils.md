# Capítulo 2: Linux API Headers + Binutils 2.46.1

## 2.1 — Objetivo

Instalar los headers del kernel (para que musl y GCC sepan cómo hablar con
Linux) y compilar binutils (as, ld, ar, etc.) como primer componente de la
toolchain temporal.

## 2.2 — Dependencias

- musl-cross en `$LFS/cross/bin/`
- Nada más (binutils no necesita una libc para compilarse)

---

## 2.3 — Linux API Headers

```bash
cd $LFS/sources
tar -xf linux-7.1.tar.xz
cd linux-7.1
make mrproper
make ARCH=arm64 headers
find usr/include -type f ! -name '*.h' -delete
cp -rv usr/include/ $LFS/tools/usr
cd ..
rm -rf linux-7.1
```

Verificar:

```bash
ls $LFS/tools/usr/include/linux/kernel.h
# debe existir
```

---

## 2.4 — Compilar binutils

```bash
cd $LFS/build
tar -xf $LFS/sources/binutils-2.46.1.tar.xz
cd binutils-2.46.1

mkdir build && cd build

../configure \
  --prefix=$LFS/tools \
  --target=aarch64-linux-musl \
  --with-sysroot=$LFS/tools \
  --disable-nls \
  --enable-gprofng=no \
  --disable-werror \
  --enable-static \
  --disable-shared

make -j$(nproc)
make install

```

### Verificar

```bash
$LFS/tools/bin/aarch64-linux-musl-ld --version
# → GNU ld (GNU Binutils) 2.46.1
```

---

<< [Anterior](01-entorno.md) | [Índice](README.md) | [Siguiente](03-gcc-minimal.md) >>
