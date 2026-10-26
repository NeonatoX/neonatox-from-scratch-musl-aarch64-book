# Capítulo 29: flex 2.6.4

## 29.1 — Objetivo

Compilar flex 2.6.4 en `/usr`. Generador de analizadores léxicos.
Necesario para Linux-PAM y otros paquetes.

## 29.2 — Dependencias

- gcc, make (ya en el sistema)

## 29.3 — Descargar

```bash
wget https://github.com/westes/flex/releases/download/v2.6.4/flex-2.6.4.tar.gz
```

## 29.4 — Compilar

```bash
cd /sources
tar xf flex-2.6.4.tar.gz
cd flex-2.6.4

./configure --prefix=/usr \
            --docdir=/usr/share/doc/flex \
            --disable-static
make -j$(nproc)

make install
ln -sv flex /usr/bin/lex
```

## 29.5 — Verificar

```bash
flex --version
# → flex 2.6.4
```

## 29.6 — Notas

- flex reemplaza a lex. Se crea symlink `lex → flex` para compatibilidad
- En musl, flex compila sin problemas
- Necesario para Linux-PAM (parsea archivos de configuración)


---

<< [Anterior](28-libffi.md) | [Índice](README.md) | [Siguiente](30-bison.md) >>
