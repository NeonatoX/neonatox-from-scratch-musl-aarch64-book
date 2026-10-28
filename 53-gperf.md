# Capítulo 53: Gperf 3.3

## 53.1 — Objetivo

Gperf generates a perfect hash function from a key set. 
   
## 53.2 — Descargar

```bash
wget https://ftpmirror.gnu.org/gperf/gperf-3.3.tar.gz

```

## 53.3 — Compilar

```bash
cd /sources
tar xvf gperf-3.3.tar.gz
cd gperf-3.3

./configure --prefix=/usr --docdir=/usr/share/doc/gperf
make -j$(nproc)

make install

```

## 53.4 — Verificar

```bash
gperf --version
# GNU gperf 3.3
# Copyright (C) 2025 Free Software Foundation, Inc.

file /usr/bin/gperf          
# /usr/bin/gperf: ELF 64-bit LSB pie executable, ARM aarch64, version 1 (SYSV), dynamically linked, interpreter /lib/ld-musl-aarch64.so.1, with debug_info, not stripped
```


---

<< [Anterior](52-util-linux.md) | [Índice](README.md) | [Siguiente](54-coreutils.md) >>
