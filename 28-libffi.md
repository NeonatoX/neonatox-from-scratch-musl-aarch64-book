# Capítulo 28: libffi 3.5.2

## 28.1 — Objetivo

Compilar libffi 3.5.2 en `/usr`. Es necesario para Python (ctypes) y otros
paquetes que necesitan invocar funciones C en tiempo de ejecución.

## 28.2 — Dependencias

- gcc, make (ya en el sistema)

## 28.3 — Descargar

```bash
wget https://github.com/libffi/libffi/releases/download/v3.5.2/libffi-3.5.2.tar.gz
```

## 28.4 — Compilar

```bash
cd /sources
tar xf libffi-3.5.2.tar.gz
cd libffi-3.5.2

./configure --prefix=/usr --disable-static
make -j$(nproc)

make install
```

## 28.5 — Verificar

```bash
file /usr/lib/libffi.so.8.2.0
# → ELF 64-bit LSB shared object, ARM aarch64, ... dynamically linked

ldd /usr/lib/libffi.so.8.2.0
# → libc.so => /lib/ld-musl-aarch64.so.1
```

## 28.6 — Notas

- `--disable-static` ahorra espacio; nadie necesita `libffi.a`
- libffi se instala en `/usr/lib/` (no `/usr/lib64/`)
- Python lo necesita para `ctypes` y `_ctypes`


---

<< [Anterior](27-pkgconf.md) | [Índice](README.md) | [Siguiente](29-flex.md) >>
