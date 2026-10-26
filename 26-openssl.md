# Capítulo 26: OpenSSL 4.0.1

## 26.1 — Objetivo

Compilar OpenSSL 4.0.1 en `/usr`. Necesario para Python (ssl), wget,
cURL, git y conexiones HTTPS en general.

## 26.2 — Dependencias

- gcc, make (ya en el sistema)
- Perl 5.42.2 (capítulo 31) — para el `Configure`

## 26.3 — Descargar

```bash
wget https://github.com/openssl/openssl/releases/download/openssl-4.0.1/openssl-4.0.1.tar.gz
```

## 26.4 — Compilar

```bash
cd /sources
tar xf openssl-4.0.1.tar.gz
cd openssl-4.0.1

./Configure linux-$(uname -m) \
            --prefix=/usr \
            --openssldir=/etc/ssl \
            --libdir=/lib \
            enable-shared

make -j$(nproc)

make install
```

## 26.5 — Verificar

```bash
/usr/bin/openssl version
# → OpenSSL 4.0.1

ldd /usr/bin/openssl
# → libssl.so.4, libcrypto.so.4, libc.so => /lib/ld-musl-aarch64.so.1
```

## 26.6 — Notas

- `--libdir=/lib` instala las shared libs en `/lib`, donde el linker
  las encuentra sin `LD_LIBRARY_PATH`
- `no-ssl3` desactiva SSL 3.0 (obsoleto e inseguro)
- Para musl, OpenSSL compila sin problemas
- OpenSSL 4.0.x usa la API 4.0; los paquetes que usen `openssl` deben
  enlazar contra `libssl.so.4` y `libcrypto.so.4`
- El build puede tomar varios minutos
- No se necesita hacer `make test` (tarda mucho y no es crítico)


---

<< [Anterior](25-perl.md) | [Índice](README.md) | [Siguiente](31-pkgconf.md) >>
