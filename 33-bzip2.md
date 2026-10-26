# Capítulo 33: bzip2 1.0.8

## 33.1 — Objetivo

A high-quality data compression program.

## 33.2 — Descargar

```bash
wget https://www.sourceware.org/pub/bzip2/bzip2-1.0.8.tar.gz

```

## 33.3 — Compilar

```bash
cd /sources
tar xvf bzip2-1.0.8.tar.gz
cd bzip2-1.0.8

patch -Np1 -i ../bzip2-1.0.8-install_docs-1.patch
sed -i 's@\(ln -s -f \)$(PREFIX)/bin/@\1@' Makefile
sed -i "s@(PREFIX)/man@(PREFIX)/share/man@g" Makefile
make -f Makefile-libbz2_so
make clean
make

make PREFIX=/usr install
cp -av libbz2.so.* /usr/lib
ln -sf libbz2.so.1.0.8 /usr/lib/libbz2.so
ln -sf libbz2.so.1.0.8 /usr/lib/libbz2.so.1
ln -sf libbz2.so.1.0.8 /usr/lib/libbz2.so.1.0
cp -v bzip2-shared /usr/bin/bzip2
for i in /usr/bin/{bzcat,bunzip2}; do
  ln -sfv bzip2 $i
done
rm -fv /usr/lib/libbz2.a
```

## 33.4 — Verificar

```bash
file /lib/libbz2.so.1.0.8 
# /lib/libbz2.so.1.0.8: ELF 64-bit LSB shared object, aarch64, version 1 (SYSV), dynamically linked, with debug_info, not stripped

```


---

<< [Anterior](32-sqlite.md) | [Índice](README.md) | [Siguiente](34-python.md) >>
