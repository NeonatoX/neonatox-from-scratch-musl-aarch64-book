# Capítulo 48: libuv 1.52.1

## 48.1 — Objetivo

The libuv package is a multi-platform support library with a focus on asynchronous I/O.

## 48.2 — Descargar

```bash
wget --no-check-cert https://dist.libuv.org/dist/v1.52.1/libuv-v1.52.1.tar.gz
# MD5:	db711bbe3822dd4fe51fad0cd78baa57
```

## 48.3 — Compilar

```bash
cd /sources
tar xvf libuv-1.52.1.tar.bz2 
cd libuv-1.52.1

sh autogen.sh
./configure --prefix=/usr --disable-static --cache-file=/root/libuv.cache
make -j$(nproc)

make install
```

## 48.4 — Verificar

```bash
file /usr/lib/libuv.so.1.0.0 
/usr/lib/libuv.so.1.0.0: ELF 64-bit LSB shared object, ARM aarch64, version 1 (SYSV), dynamically linked, with debug_info, not stripped


```


---

<< [Anterior](47-libtool.md) | [Índice](README.md) | [Siguiente](49-cmake.md) >>
