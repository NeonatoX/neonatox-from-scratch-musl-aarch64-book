# Capítulo 44: Automake 1.18.1

## 44.1 — Objetivo

A GNU tool for automatically creating Makefiles.

## 44.2 — Descargar

```bash
wget https://ftp.gnu.org/gnu/automake/automake-1.18.1.tar.xz
```

## 44.3 — Compilar

```bash
cd /sources
tar xvf automake-1.18.1.tar.xz 
cd automake-1.18.1

./configure --prefix=/usr --docdir=/usr/share/doc/automake 
make

make install
```

## 44.4 — Verificar

```bash
automake --version
# automake (GNU Automake) 1.18.1

```


---

<< [Anterior](43-autoconf.md) | [Índice](README.md) | [Siguiente](45-pcre2.md) >>
