# Capítulo 55: LZO 2.10

## 55.1 — Objetivo

Compilar LZO 2.10 en `/usr`. Biblioteca de compresión rápida sin pérdida.
Opcional para btrfs-progs (compresión lzo en btrfs).

## 55.2 — Descargar

```bash
wget https://www.oberhumer.com/opensource/lzo/download/lzo-2.10.tar.gz
```

## 55.3 — Compilar

```bash
cd /sources
rm -rf lzo-2.10
tar xf lzo-2.10.tar.gz
cd lzo-2.10

./configure --prefix=/usr --enable-shared
make -j$(nproc)

make install
```

## 55.4 — Verificar

```bash
file /usr/lib/liblzo2.so.2.0.0 
/usr/lib/liblzo2.so.2.0.0: ELF 64-bit LSB shared object, ARM aarch64, version 1 (SYSV), dynamically linked, with debug_info, not stripped

pkg-config --libs lzo2
# → -llzo2
```


---

<< [Anterior](54-coreutils.md) | [Índice](README.md) | [Siguiente](56-cpio.md) >>
