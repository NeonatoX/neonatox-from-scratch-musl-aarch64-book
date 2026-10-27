# Capítulo 43: autoconf 2.72

## 43.1 — Objetivo

A GNU tool for automatically configuring source code.

## 43.2 — Descargar

```bash
wget https://ftp.gnu.org/gnu/autoconf/autoconf-2.72.tar.xz
```

## 43.3 — Compilar

```bash
cd /sources
tar xvf autoconf-2.72.tar.xz 
cd autoconf-2.72

./configure --prefix=/usr 
make -j$(nproc)

make install
```

## 43.4 — Verificar

```bash
autoconf --version
# autoconf (GNU Autoconf) 2.72

```


---

<< [Anterior](42-curl.md) | [Índice](README.md) | [Siguiente](44-automake.md) >>
