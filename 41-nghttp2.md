# Capítulo 41: Nghttp2 1.66.0

## 41.1 — Objetivo

An implementation of HTTP/2 and its header compression algorithm, HPACK. Dependency of curl

## 41.2 — Descargar

```bash
wget https://github.com/nghttp2/nghttp2/releases/download/v1.66.0/nghttp2-1.66.0.tar.xz
```

## 41.3 — Compilar

```bash
cd /sources
tar xvf nghttp2-1.66.0.tar.xz
cd nghttp2-0.21.5

./configure --prefix=/usr --disable-static --docdir=/usr/share/doc/nghttp2
make -j$(nproc)

make install
```

## 41.4 — Verificar

```bash

file /lib/libnghttp2.so.14.28.5
# /lib/libnghttp2.so.14.28.5: ELF 64-bit LSB shared object, ARM aarch64, version 1 (SYSV), dynamically linked, not stripped


```


---

<< [Anterior](40-libpsl.md) | [Índice](README.md) | [Siguiente](42-curl.md) >>
