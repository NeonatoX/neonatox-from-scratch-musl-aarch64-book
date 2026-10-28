# Capítulo 68: Jinja2 3.1.6

## 68.1 — Objetivo

Jinja2 is a Python module that implements a simple pythonic template language. 
  
## 68.2 — Descargar

```bash
wget https://pypi.org/packages/source/J/Jinja2/jinja2-3.1.6.tar.gz

```

## 68.3 — Compilar

```bash
cd /sources
tar xvf jinja2-3.1.6.tar.gz
cd jinja2-3.1.6


pip3 wheel -w dist --no-cache-dir --no-build-isolation --no-deps $PWD

pip3 install --no-index --find-links dist --no-deps --ignore-installed --force-reinstall Jinja2

```



---

<< [Anterior](67-markupsafe.md) | [Índice](README.md) | [Siguiente](69-systemd.md) >>
