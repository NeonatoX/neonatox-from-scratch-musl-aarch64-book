# Capítulo 62: setuptools 82.0.1

## 62.1 — Objetivo

Setuptools is a tool used to download, build, install, upgrade, and uninstall Python packages. 
 
## 62.2 — Descargar

```bash
wget https://pypi.org/packages/source/s/setuptools/setuptools-82.0.1.tar.gz

```

## 62.3 — Compilar

```bash
cd /sources
tar xvf setuptools-82.0.1.tar.gz
cd setuptools-82.0.1


pip3 wheel -w dist --no-cache-dir --no-build-isolation --no-deps $PWD

pip3 install --no-index --find-links dist --ignore-installed --force-reinstall setuptools

```



---

<< [Anterior](61-wheel.md) | [Índice](README.md) | [Siguiente](63-libpwquality.md) >>
