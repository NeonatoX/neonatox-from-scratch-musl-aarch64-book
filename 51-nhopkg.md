# Capítulo 51: Nhopkg desde git

## 51.1 — Descripcion

Nhopkg is a universal package manager designed to work on any GNU/Linux distribution with bash.

## 51.2 — Dependencias

- meson y ninja
- libarchive
- findutils (find)
- plocate (updatedb y locate)
- gettext
- curl
- wget (GNU o BusyBox)

## 51.3 — Descargar

```bash
git clone https://github.com/cargabsj175/neonatox-nhopkg.git
```

## 51.4 — Compilar

```bash
    cd /sources
    cd neonatox-nhopkg/
meson setup builddir \
  -D binlocate=plocate \
  -D repo-version=n27 \
  -D repo-arch=$(uname -m)-musl \
  -D libc=musl \
  -D git-branch=musl \
  -D static-busybox=no \
  -D static-zstd=no \
  -D use-busybox=no
ninja -C builddir

ninja -C builddir install
mkdir -p ${LFS}/var/nhopkg/{cache,files,logs,packages,repo}
```

## 51.4 — Verificar

```bash
nhopkg --version

Versión de Nhopkg 2.1.0
Nhopkg  Copyright (C) 2008-2009  Jaime Gil de Sagredo Luna <jaimegildesagredo@gmail.com>
Nhopkg  Copyright (C) 2009-2026  Carlos Sánchez <cargabsj175@gmail.com>

```



---

<< [Anterior](50-plocate.md) | [Índice](README.md) | [Siguiente](52-util-linux.md) >>
