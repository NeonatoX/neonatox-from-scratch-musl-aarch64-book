# Capítulo 42: Curl 8.20.0

## 42.1 — Objetivo

An URL retrival utility and library.

## 42.2 — Dependencias 

- libpsl
- nghttp2

## 42.3 — Descargar

```bash
wget https://curl.se/download/curl-8.20.0.tar.xz
```

## 42.4 — Compilar

```bash
cd /sources
tar xvf curl-8.20.0.tar.xz
cd curl-8.20.0

sed -i 's/2F 5C/%2F %5C/' scripts/wcurl

mkdir -p build && cd build

../configure --prefix=/usr --disable-static --with-openssl --with-ca-path=/etc/ssl/certs  
make -j$(nproc)

make install
```

## 42.5 — Verificar

```bash

file /bin/curl
# /bin/curl: ELF 64-bit LSB pie executable, ARM aarch64, version 1 (SYSV), dynamically linked, interpreter /lib/ld-musl-aarch64.so.1, not stripped


```


---

<< [Anterior](41-nghttp2.md) | [Índice](README.md) | [Siguiente](43-autoconf.md) >>
