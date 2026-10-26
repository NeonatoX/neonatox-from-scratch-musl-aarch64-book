# Capítulo 27: pkgconf 2.5.1

## 27.1 — Objetivo

Compilar pkgconf 2.5.1 en `/usr`. Proporciona `pkg-config` — necesario
para detectar bibliotecas del sistema durante builds (shadow, Python,
kmod, etc.).

## 27.2 — Dependencias

- gcc, make (ya en el sistema)

## 27.3 — Descargar

```bash
wget https://github.com/pkgconf/pkgconf/releases/download/pkgconf-2.5.1/pkgconf-2.5.1.tar.xz
```

## 27.4 — Compilar

```bash
cd /sources
tar xf pkgconf-2.5.1.tar.xz
cd pkgconf-2.5.1

./configure --prefix=/usr \
            --disable-static \
            --docdir=/usr/share/doc/pkgconf

make
make install

ln -sv pkgconf   /usr/bin/pkg-config
ln -sv pkgconf.1 /usr/share/man/man1/pkg-config.1
```

## 27.5 — Verificar

```bash
pkg-config --version
# → 2.5.1
```

## 27.6 — Notas

- pkgconf es el drop-in replacement de pkg-config, compatible con musl
- Se instala como `pkgconf` y se crea symlink `pkg-config -> pkgconf`
- Busca archivos `.pc` en `/usr/lib/pkgconfig/` y `/usr/share/pkgconfig/`
- Las bibliotecas instaladas (zlib, openssl, libffi) ya tienen sus `.pc` en `/usr/lib/pkgconfig/`


---

<< [Anterior](26-openssl.md) | [Índice](README.md) | [Siguiente](28-libffi.md) >>
