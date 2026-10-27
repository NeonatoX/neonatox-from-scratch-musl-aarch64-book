# Capítulo 40: Libpsl 0.21.5 

## 40.1 — Objetivo

Public Suffix List library is a curl dependency

## 40.2 — Descargar

```bash
wget https://github.com/rockdaboot/libpsl/releases/download/0.21.5/libpsl-0.21.5.tar.gz
```

## 40.3 — Compilar

```bash
cd /sources
tar xvf libpsl-0.21.5.tar.gz 
cd libpsl-0.21.5

mkdir build && cd build

meson setup --prefix=/usr --buildtype=release
ninja

ninja install
```

## 40.4 — Verificar

```bash

file /lib/libpsl.so.5.3.5
# /lib/libpsl.so.5.3.5: ELF 64-bit LSB shared object, ARM aarch64, version 1 (SYSV), dynamically linked, not stripped


```


---

<< [Anterior](43-grep.md) | [Índice](README.md) | [Siguiente](45-nghttp2.md) >>
