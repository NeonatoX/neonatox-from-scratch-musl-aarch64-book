# Capítulo 31: expat 2.7.1

## 31.1 — Objetivo

Compilar expat 2.7.1 en `/usr`. Parser XML. Necesario para Python
(módulo `pyexpat`).

## 31.2 — Dependencias

- gcc, make (ya en el sistema)

## 31.3 — Descargar

```bash
wget https://github.com/libexpat/libexpat/releases/download/R_2_7_1/expat-2.7.1.tar.xz
```

## 31.4 — Compilar

```bash
cd /sources
tar xf expat-2.7.1.tar.xz
cd expat-2.7.1

./configure --prefix=/usr --disable-static
make -j$(nproc)

make install
```

## 31.5 — Verificar

```bash
ldd /usr/lib/libexpat.so
# → libc.so => /lib/ld-musl-aarch64.so.1
```

## 31.6 — Notas

- expat es un parser XML stream-oriented, minimalista
- `--disable-static` ahorra espacio
- Python lo usa para `pyexpat` y `xml.parsers.expat`
- Si no está, Python compila con su copia interna; es mejor usar la
  del sistema para mantener consistencia


---

<< [Anterior](30-bison.md) | [Índice](README.md) | [Siguiente](31-sqlite.md) >>
