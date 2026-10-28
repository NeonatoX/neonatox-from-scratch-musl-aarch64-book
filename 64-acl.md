# Capítulo 64: Acl-2.6.0

## 64.1 - Objetivo

Access control list utilities, libraries and headers.

## 64.3 — Descargar

```bash
wget https://download.savannah.gnu.org/releases/acl/acl-2.6.0.tar.gz
```

## 64.4 — Compilar

```bash
./configure --prefix=/usr    \
            --disable-static \
            --docdir=/usr/share/doc/acl
make -j$(nproc)

make install
```

## 64.5 — Verificar

```bash
file /usr/lib/libacl.so.1.1.2302
/usr/lib/libacl.so.1.1.2302: ELF 64-bit LSB shared object, ARM aarch64, version 1 (SYSV), dynamically linked, with debug_info, not stripped
```


---

<< [Anterior](63-attr.md) | [Índice](README.md) | [Siguiente](65-libcap.md) >>
