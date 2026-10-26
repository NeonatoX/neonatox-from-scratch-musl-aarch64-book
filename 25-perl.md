# Capítulo 29: Perl 5.42.2

## 29.1 — Objetivo

Compilar Perl 5.42.2 en `/usr`. Es necesario para el `Configure` de OpenSSL
y para muchos scripts del sistema.

## 29.2 — Dependencias

- gcc, make (ya en el sistema)
- zlib (opcional para Compress::Zlib, se desactiva para usar la del sistema)

## 29.4 — Descargar

```bash
wget https://www.cpan.org/src/5.0/perl-5.42.2.tar.xz
```

## 29.5 — Compilar

```bash
cd /sources
tar xf perl-5.42.2.tar.xz
cd perl-5.42.2

export BUILD_ZLIB=False

sh Configure -des \
             -D cc=gcc \
             -D prefix=/usr \
             -D vendorprefix=/usr \
             -D privlib=/usr/lib/perl5/5.42/core_perl \
             -D archlib=/usr/lib/perl5/5.42/core_perl \
             -D sitelib=/usr/lib/perl5/5.42/site_perl \
             -D sitearch=/usr/lib/perl5/5.42/site_perl \
             -D vendorlib=/usr/lib/perl5/5.42/vendor_perl \
             -D vendorarch=/usr/lib/perl5/5.42/vendor_perl \
             -D man1dir=/usr/share/man/man1 \
             -D man3dir=/usr/share/man/man3 \
             -D pager="/usr/bin/less -isR" \
             -D useshrplib \
             -D usethreads \
             -Dcf_by='NeonatoX' \
             -D ldflags='-s'

make -j$(nproc)
make install

```

## 29.6 — Verificar

```bash
perl -V
# → Summary of my perl5 (revision 5 version 42 subversion 2)

perl -e 'print "Hello from Perl\n"'
# → Hello from Perl
```



---

<< [Anterior](24-zstd.md) | [Índice](README.md) | [Siguiente](26-openssl.md) >>
