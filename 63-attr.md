# Capítulo 63: Attr-2.6.0

## 63.1 - Objetivo

Extended attribute support library for ACL support.

## 63.3 — Descargar

```bash
wget --no-check-cert https://download.savannah.gnu.org/releases/attr/attr-2.6.0.tar.gz
```

## 63.4 — Compilar

```bash
./configure --prefix=/usr     \
            --disable-static  \
            --sysconfdir=/etc \
            --docdir=/usr/share/doc/attr
make -j$(nproc)


make install
```

## 63.5 — Verificar

```bash
 file /usr/lib/libattr.so.1.1.2600 
/usr/lib/libattr.so.1.1.2600: ELF 64-bit LSB shared object, ARM aarch64, version 1 (SYSV), dynamically linked, with debug_info, not stripped

```

---

<< [Anterior](62-setuptools.md) | [Índice](README.md) | [Siguiente](64-acl.md) >>
