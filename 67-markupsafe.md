# Capítulo 67: Markupsafe 3.0.3

## 67.1 — Objetivo

MarkupSafe is a Python module that implements an XML/HTML/XHTML Markup safe string. 
  
## 67.2 — Descargar

```bash
wget https://pypi.org/packages/source/M/MarkupSafe/markupsafe-3.0.3.tar.gz

```

## 67.3 — Compilar

```bash
cd /sources
tar xvf markupsafe-3.0.3.tar.gz
cd markupsafe-3.0.3


pip3 wheel -w dist --no-cache-dir --no-build-isolation --no-deps $PWD

pip3 install --no-index --find-links dist --ignore-installed --force-reinstall Markupsafe

```



---

<< [Anterior](66-shadow.md) | [Índice](README.md) | [Siguiente](68-jinja2.md) >>
