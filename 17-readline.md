# Capítulo 17: readline

## 17.1 — Objetivo

Compilar readline contra musl. Necesario para bash completo.

## 17.2 — Dependencias

- ncurses (capítulo anterior)

---

## 17.3 — Compilar

```
cd /sources
rm -rf readline-8.3
tar -xf readline-8.3.tar.gz
cd readline-8.3

./configure \
  --prefix=/usr \
  --disable-static \
  --with-curses

make -j$(nproc) SHLIB_LIBS="-lncursesw"
make install 
```

## 17.4 — Verificar

```
ls /usr/lib/libreadline.so*
ls /usr/include/readline/readline.h
```

---

<< [Anterior](16-ncurses.md) | [Índice](README.md) | [Siguiente](18-bash-final.md) >>
