# Capítulo 7: Binutils 2.46.1 (Stage 2 — $LFS/usr)

## 7.1 — Objetivo

Recompilar binutils para el target `aarch64-linux-musl`, instalándolos en
`$LFS/usr`. Esta vez serán musl nativos (a diferencia de los de stage 1,
que son glibc-linked).

La compilación es cross (canadiense): `--build=aarch64-pc-linux-gnu`
(build host) → `--host=aarch64-linux-musl` (target final).

## 7.2 — Dependencias

- `aarch64-linux-musl-gcc` en `$LFS/tools/bin/`
- Linux API headers en `$LFS/usr/include`

---

## 7.3 — Compilar

```bash
cd $LFS/sources
rm -rf binutils-2.46.1
tar -xf binutils-2.46.1.tar.xz
cd binutils-2.46.1

mkdir build && cd build

# Forzamos el uso del compilador cruzado ARM64 de la etapa 1
CC=aarch64-linux-musl-gcc \
AR=aarch64-linux-musl-ar \
RANLIB=aarch64-linux-musl-ranlib \
../configure \
  --prefix=/usr \
  --build=$(../config.guess) \
  --host=aarch64-linux-musl \
  --target=aarch64-linux-musl \
  --with-sysroot=/ \
  --disable-nls \
  --disable-multilib

make -j$(nproc) 2>&1 | tee build.log

make DESTDIR=$LFS install

# Enlaces simbólicos de compatibilidad internos del chroot
ln -sf ar      $LFS/usr/bin/aarch64-linux-musl-ar
ln -sf as      $LFS/usr/bin/aarch64-linux-musl-as
ln -sf ld      $LFS/usr/bin/aarch64-linux-musl-ld
ln -sf ranlib  $LFS/usr/bin/aarch64-linux-musl-ranlib
ln -sf nm      $LFS/usr/bin/aarch64-linux-musl-nm
ln -sf strip   $LFS/usr/bin/aarch64-linux-musl-strip

```



### Explicación

| Flag | Significado |
|------|-------------|
| `--prefix=/usr` | Ruta en el sistema final (DESTDIR=$LFS lo mueve) |
| `--host=aarch64-linux-musl` | Los binarios correrán en musl |
| `--target=aarch64-linux-musl` | Producen código para musl |
| `--with-sysroot=/` | Busca headers/libs en `/` del target |
| `CC=aarch64-linux-musl-gcc` | Cross-compiler build→host |

---

## 7.4 — Verificar

```bash
file $LFS/usr/bin/ld
# → ELF 64-bit LSB executable, ARM aarch64, version 1 (SYSV), dynamically linked, interpreter /lib/ld-musl-aarch64.so.1, ...

# También verificar as y ar
file $LFS/usr/bin/as $LFS/usr/bin/ar
# → Ambos deben indicar "ARM aarch64" y apuntar a "/lib/ld-musl-aarch64.so.1"
```

Si ves `ld-linux-aarch64.so.2`, algo falló (sigue glibc-linked).

---

<< [Anterior](06-linux-headers.md) | [Índice](README.md) | [Siguiente](08-musl-stage2.md) >>
