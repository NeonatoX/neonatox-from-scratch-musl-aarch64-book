# Capítulo 6: Linux API Headers (Stage 2 — $LFS/usr)

## 6.1 — Objetivo

Instalar las cabeceras del kernel de Linux en `$LFS/usr/include`.
Estas son necesarias para compilar musl, binutils, GCC y cualquier otro
paquete del sistema base.

## 6.2 — Dependencias

- host glibc (`aarch64-pc-linux-gnu` — compilador nativo para make)
- Linux API headers source

---

## 6.3 — Compilar

```bash
cd $LFS/sources
rm -rf linux-7.1
tar -xf linux-7.1.tar.xz
cd linux-7.1

make mrproper
make ARCH=arm64 headers
find usr/include -name '.*' -delete
rm -rf $LFS/usr/include
cp -rv usr/include/ $LFS/usr/include
```

### Explicación

- `make mrproper` — limpiar cualquier residuo del source
- `make headers` — genera los headers en `usr/include/` dentro del árbol
- `find ... -delete` — elimina archivos ocultos (.install, ..install.cmd)
- `cp -rv usr/include $LFS/usr/include` — copia los headers al destino

> **Nota**: no usamos `make headers_install` porque requiere `rsync`, que
no está disponible en el host. Extraer los headers con `make headers`
+ `cp` es equivalente.

Tras esto, `$LFS/usr/include/linux/`, `$LFS/usr/include/asm/`,
`$LFS/usr/include/asm-generic/` deben existir y contener los headers.

---

## 6.4 — Verificar

```bash
ls $LFS/usr/include/linux/ | head -5
ls $LFS/usr/include/asm/ | head -5

---

<< [Anterior](05-gcc-completo.md) | [Índice](README.md) | [Siguiente](07-binutils-stage2.md) >>
