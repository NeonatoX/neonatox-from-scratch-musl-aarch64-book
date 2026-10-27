# Capítulo : ninja 1.13.2 + meson 1.12.1

## 35.1 — Objetivo

Instalar ninja y meson vía pip. Necesarios para Linux-PAM y kmod.

## 35.2 — Dependencias

- Python 3.14.6 (capítulo 34) con pip

## 35.3 — Instalar

```bash
pip3 install ninja meson
```

## 35.4 — Verificar

```bash
ninja --version
# → 1.13.2.git.kitware.jobserver-pipe-1

meson --version
# → 1.12.1
```

## 35.5 — Notas

- Se instalan vía `pip3` (incluido con `--with-ensurepip=yes`)
- Los binarios quedan en `/usr/bin/`
- meson necesita ninja para compilar; pip instala ambas dependencias
- No compilar ninja desde fuente; es más rápido vía pip


---

<< [Anterior](34-python.md) | [Índice](README.md) | [Siguiente](36-libarchive.md) >>
