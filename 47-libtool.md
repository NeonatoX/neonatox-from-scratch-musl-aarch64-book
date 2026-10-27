# Capítulo 47: libtool 2.5.4

## 47.1 — Objetivo

The libtool package is a multi-platform support library with a focus on asynchronous I/O.

## 47.2 — Descargar

```bash
wget --no-check-cert https://ftp.gnu.org/gnu/libtool/libtool-2.5.4.tar.xz
# SHA256:	f81f5860666b0bc7d84baddefa60d1cb9fa6fceb2398cc3baca6afaa60266675  libtool-2.5.4.tar.xz
```

## 47.3 — Compilar

```bash
cd /sources
tar xvf libtool-2.5.4.tar.xz
cd libtool-2.5.4

./configure --prefix=/usr
make -j$(nproc)

make install
```

## 47.4 — Verificar

```bash
file /usr/lib/libltdl.so.7.3.3
/usr/lib/libltdl.so.7.3.3: ELF 64-bit LSB shared object, ARM aarch64, version 1 (SYSV), dynamically linked, not stripped
```


---

<< [Anterior](46-git.md) | [Índice](README.md) | [Siguiente](48-libuv.md) >>
