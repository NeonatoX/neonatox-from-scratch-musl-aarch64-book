# Capítulo 65: libcap-2.78

## 65.1 - Objetivo

POSIX 1003.1e capabilities.

## 65.3 — Descargar

```bash
wget https://www.kernel.org/pub/linux/libs/security/linux-privs/libcap2/libcap-2.78.tar.xz
```

## 65.4 — Compilar

```bash
sed -i '/install -m.*STA/d' libcap/Makefile
make -j$(nproc) prefix=/usr lib=lib

make DESTDIR=/usr lib=lib install
```

## 65.5 — Verificar

```bash
file /usr/lib/libcap.so.2.78 
/usr/lib/libcap.so.2.78: ELF 64-bit LSB shared object, ARM aarch64, version 1 (SYSV), dynamically linked, interpreter /lib/ld-musl-aarch64.so.1, with debug_info, not stripped

```


---

<< [Anterior](64-acl.md) | [Índice](README.md) | [Siguiente](66-shadow.md) >>
