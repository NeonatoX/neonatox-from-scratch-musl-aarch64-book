# Capítulo 20: file-5.48

## 20.1 — Objetivo

Compilar file 5.48, comando que determina el tipo de archivos binarios.

## 20.2 — Dependencias

- GCC musl nativo en `/usr/bin/gcc`
- make

---

## 20.3 — Compilar

```
cd /sources
rm -rf file-5.48
tar -xf file-5.48.tar.gz
cd file-5.48

./configure --prefix=/usr 

make -j$(nproc)
make install
```

## 20.4 — Verificar

```
file /usr/bin/bash
# → ELF ... dynamically linked, interpreter /lib/ld-musl-aarch64.so.1
```

---

<< [Anterior](18-nano.md) | [Índice](README.md) | [Siguiente](20-m4.md) >>
