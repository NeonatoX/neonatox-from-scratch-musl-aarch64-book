# Capítulo 56: Cpio 2.15

## 56.1 — Objetivo

Compilar GNU Cpio 2.15 en `/usr`. Necesario como dependencia de
`neonatox-mkinitramfs`.

## 56.2 — Dependencias

- make (capítulo 15)
- gcc (capítulo 27)

## 56.3 — Descargar

```bash
wget --no-check-cert https://ftp.gnu.org/gnu/cpio/cpio-2.15.tar.bz2
```

## 56.4 — Compilar

```bash
cd /sources
rm -rf cpio-2.15
tar -xf cpio-2.15.tar.bz2
cd cpio-2.15

# Parchear declaraciones de funciones externas para musl
sed -e "/^extern int (\*xstat)/s/()/(const char * restrict,  struct stat * restrict)/" \
    -i src/extern.h
sed -e "/^int (\*xstat)/s/()/(const char * restrict,  struct stat * restrict)/" \
    -i src/global.c

./configure --prefix=/usr --enable-mt --with-rmt=/usr/libexec/rmt
make -j$(nproc)
make install
```

## 56.5 — Verificar

```bash
cpio --version
# cpio (GNU cpio) 2.15

echo test | cpio -o > /dev/null && echo "OK"
```


---

<< [Anterior](55-lzo.md) | [Índice](README.md) | [Siguiente](57-cracklib.md) >>
