# Capítulo 70: Texinfo-7.3

## 70.1 - Objetivo

GNU documentation system for on-line information and printed output.

## 70.3 — Descargar

```bash
wget --no-check-cert https://ftpmirror.gnu.org/texinfo/texinfo-7.3.tar.xz
# MD5:	915a09fcdfc2bf0f5ecf9e556d5698ff
```

## 70.4 — Compilar

```bash
cd /sources
tar xvf texinfo-7.3.tar.xz


sed 's/! $output_file eq/$output_file ne/' -i tp/Texinfo/Convert/*.pm
./configure --prefix=/usr
make -j$(nproc)


make install
make TEXMF=/usr/share/texmf install-tex
pushd /usr/share/info
  rm -v dir
  for f in *
    do install-info $f dir 2>/dev/null
  done
popd
```

## 70.5 — Verificar

```bash
texi2any --version
texi2any (GNU texinfo) 7.3

Copyright (C) 2026 Free Software Foundation, Inc.
License GPLv3+: GNU GPL version 3 or later <https://gnu.org/licenses/gpl.html>
This is free software: you are free to change and redistribute it.
There is NO WARRANTY, to the extent permitted by law.

```

---

<< [Anterior](69-texinfo.md) | [Índice](README.md) | [Siguiente](71-e2fsprogs.md) >>
