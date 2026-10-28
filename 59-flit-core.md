# Capítulo 59: flit-core 3.12.0

## 59.1 — Objetivo

Flit-core is the distribution-building parts of Flit (a packaging tool for simple Python modules). 
 
## 59.2 — Descargar

```bash
wget --no-check-cert  https://pypi.org/packages/source/f/flit-core/flit_core-3.12.0.tar.gz

```

## 59.3 — Compilar

```bash
cd /sources
tar xvf flit_core-3.12.0.tar.gz
cd flit_core-3.12.0

pip3 wheel -w dist --no-cache-dir --no-build-isolation --no-deps $PWD

pip3 install --no-index --find-links dist --ignore-installed --force-reinstall flit_core

```



---

<< [Anterior](58-linux-pam.md) | [Índice](README.md) | [Siguiente](60-packaging.md) >>
