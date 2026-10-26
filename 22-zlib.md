# Capítulo 22: zlib 1.3.2

## 22.1 — Objetivo

Compilar zlib como biblioteca compartida en `/usr`. Es dependencia de
openssl, curl, python, ssh, y muchas otras herramientas del sistema.

## 22.2 — Dependencias

- make, gcc (ya en el sistema)

## 22.3 — Descargar

```
wget https://zlib.net/zlib-1.3.2.tar.gz
```

## 22.4 — Compilar

```bash
cd /sources
tar xf zlib-1.3.2.tar.gz
cd zlib-1.3.2

./configure --prefix=/usr --libdir=/usr/lib --shared 
make -j$(nproc)
make install
```

Flags:
- `--shared`: compila solo la biblioteca compartida (`.so`)
- `--libdir=/usr/lib`: musl busca en `/usr/lib`, no `/usr/lib64`

## 22.5 — Verificar

```bash
ldconfig /usr/lib
file /usr/lib/libz.so.1.3.2
# → ELF ... shared object

ldd /usr/lib/libz.so.1.3.2
# → libc.so => /lib/ld-musl-aarch64.so.1

echo -n | gzip | zcat > /dev/null
# → sin errores
```

---

<< [Anterior](21-m4.md) | [Índice](README.md) | [Siguiente](23-sed.md) >>
