# Capítulo 18: Bash-5.3 Final (Dinámico musl)

## 18.1 — Objetivo

Recompilar bash dentro del chroot como binario dinámico contra musl,
con soporte completo de readline, history, NLS.

## 18.2 — Dependencias

- GCC musl nativo en `/usr/bin/gcc`
- ncurses
- readline

---

## 18.3 — Compilar

```
cd /sources
rm -rf bash-5.3
tar -xf bash-5.3.tar.gz
cd bash-5.3

./configure \
  --prefix=/usr \
  --without-bash-malloc \
  --with-installed-readline

make -j$(nproc)

make install

# Symlink /bin/sh → bash
ln -sf bash /bin/sh
```

Sin `--host` (compilación nativa dentro del chroot).

---

## 18.4 — Verificar

```
/usr/bin/bash --version
# → GNU bash, version 5.2.37

ldd /usr/bin/bash
# → libreadline.so.8 => /lib/libreadline.so.8
# → libhistory.so.8 => /lib/libhistory.so.8
# → libncursesw.so.6 => /lib/libncursesw.so.6
# → libc.so => /lib/ld-musl-aarch64.so.1
```

---

<< [Anterior](16-readline.md) | [Índice](README.md) | [Siguiente](18-nano.md) >>
