# Capítulo 38: Gettext 1.0

## 38.1 — Objetivo

The Gettext package contains utilities for internationalization and localization. These allow programs to be compiled with NLS (Native Language Support), enabling them to output messages in the user's native language.    

> Dependency for nhopkg and optional dependency for SystemD

## 38.2 — Descargar

```bash
wget https://ftpmirror.gnu.org/gettext/gettext-1.0.tar.xz

```

## 38.3 — Compilar

```bash
cd /sources
tar xvf gettext-1.0.tar.xz
cd gettext-1.0

./configure --prefix=/usr    \
            --disable-static \
            --docdir=/usr/share/doc/gettext 
make -j$(nproc)

make install

```

## 38.4 — Verificar

```bash
gettext --version
# GNU gettext 1.0
# Copyright (C) 2025 Free Software Foundation, Inc.

file /usr/bin/gettext          
# /usr/bin/gettext: ELF 64-bit LSB pie executable, ARM aarch64, version 1 (SYSV), dynamically linked, interpreter /lib/ld-musl-x86_64.so.1, with debug_info, not stripped
```


---

<< [Anterior](37-findutils.md) | [Índice](README.md) | [Siguiente](39-grep.md) >>
