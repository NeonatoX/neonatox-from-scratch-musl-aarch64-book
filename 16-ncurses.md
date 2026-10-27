# Capítulo 16: ncurses

## 16.1 — Objetivo

Compilar ncurses contra musl dentro del chroot. Necesario para
readline (y por tanto para bash completo).

## 16.2 — Dependencias

- GCC musl nativo en `/usr/bin/gcc`
- Linux API headers
- make (ya compilado en capítulos anteriores)

---

## 16.3 — Compilar

```
cd /sources
rm -rf ncurses-6.6
tar -xf ncurses-6.6.tar.gz
cd ncurses-6.6

./configure \
  --prefix=/usr \
  --with-shared \
  --without-debug \
  --without-ada \
  --enable-widec \
  --enable-overwrite

# fix on musl aarch64
# sed -i 's/#     include <iostream.h>/#     include <iostream>/' c++/etip.h

make -j$(nproc)
make install.libs install.includes install.progs

# Generar base de datos terminfo (tic necesita libgcc_s)
tic -s misc/terminfo.src

# Compatibilidad: libncurses.so → libncursesw.so
ln -sf libncursesw.so /usr/lib/libncurses.so
ln -sf libncursesw.a /usr/lib/libncurses.a
ln -sf libncursesw.so /usr/lib/libcurses.so
ln -sf libncursesw.a /usr/lib/libcurses.a
```

> Nota: no usar `make install` directamente porque el target
> `install.data` ejecuta `run_tic.sh` que espera `tic` en PATH.
> Se instalan bibliotecas, includes y programas primero, luego se
> genera terminfo manualmente con `tic`.

---

## 16.4 — Verificar

```
file /usr/lib/libncursesw.so
# → ELF ... dynamically linked, interpreter /lib/ld-musl-aarch64.so.1

ls /usr/include/ncurses.h

tic -V
# → ncurses 6.6
```

---

<< [Anterior](16-gwak.md) | [Índice](README.md) | [Siguiente](17-readline.md) >>
