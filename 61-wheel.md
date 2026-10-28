# Capítulo 61: Wheel 0.46.1

## 61.1 — Objetivo

Wheel is a Python library that is the reference implementation of the Python wheel packaging standard. 

## 61.2 — Descargar

```bash
wget https://pypi.org/packages/source/w/wheel/wheel-0.46.1.tar.gz

```

## 61.3 — Compilar

```bash
cd /sources
tar xvf wheel-0.46.1.tar.gz
cd wheel-0.46.1


pip3 wheel -w dist --no-cache-dir --no-build-isolation --no-deps $PWD

pip3 install --no-index --find-links dist --no-deps --ignore-installed --force-reinstall wheel

```



---

<< [Anterior](60-packaging.md) | [Índice](README.md) | [Siguiente](62-setuptools.md) >>
