# Capítulo 60: Packaging 26.2

## 60.1 — Objetivo

This includes utilities for version handling, specifiers, markers, tags, and requirements. 

## 60.2 — Descargar

```bash
wget https://files.pythonhosted.org/packages/source/p/packaging/packaging-26.2.tar.gz

```

## 60.3 — Compilar

```bash
cd /sources
tar xvf packaging-26.2.tar.gz
cd packaging-26.2


pip3 wheel -w dist --no-cache-dir --no-build-isolation --no-deps $PWD

pip3 install --no-index --find-links dist --ignore-installed --force-reinstall packaging

```



---

<< [Anterior](59-flit-core.md) | [Índice](README.md) | [Siguiente](61-wheel.md) >>
