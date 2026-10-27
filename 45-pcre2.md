# Capítulo 45: pcre2 10.47

## 45.1 — Objetivo

A GNU tool for automatically creating Makefiles.

## 45.2 — Descargar

```bash
wget https://github.com/PCRE2Project/pcre2/releases/download/pcre2-10.47/pcre2-10.47.tar.bz2
```

## 45.3 — Compilar

```bash
cd /sources
tar xvf pcre2-10.47.tar.bz2 
cd pcre2-10.47

./configure --prefix=/usr                       \
            --docdir=/usr/share/doc/pcre2 \
            --enable-unicode                    \
            --enable-jit                        \
            --enable-pcre2-16                   \
            --enable-pcre2-32                   \
            --enable-pcre2grep-libz             \
            --enable-pcre2test-libreadline      \
            --disable-static 
make -j$(nproc)

make install
```

## 45.4 — Verificar

```bash
pcre2-config --version
# 10.47

```


---

<< [Anterior](44-automake.md) | [Índice](README.md) | [Siguiente](46-git.md) >>
