# Capítulo 23: Sed-4.10 

## 23.1 — Objetivo

### Nota
Aunque BusyBox incluye una implementación de sed, para construir el GCC final se recomienda utilizar GNU Sed. Junto con GNU Awk, forma parte de las herramientas de construcción utilizadas durante la compilación final de GCC.

## 23.2 — Dependencias

- make, gcc (STAGE 2) (ya en el sistema)

## 23.3 — Descargar

```
wget https://ftpmirror.gnu.org/sed/sed-4.10.tar.xz
```

## 23.4 — Compilar

```bash
cd /sources
tar xf sed-4.10.tar.xz
cd sed-4.10

./configure --prefix=/usr --cache-file=/root/sed.cache
make -j$(nproc)

make install

```

---

<< [Anterior](22-zlib.md) | [Índice](README.md) | [Siguiente](27-gcc-final.md) >>
