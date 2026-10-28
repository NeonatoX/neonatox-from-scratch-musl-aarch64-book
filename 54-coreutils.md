# Capítulo 54: Coreutils 9.11

## 54.1 — Objetivo

The Coreutils package contains the basic utility programs needed by every operating system. 

> Required to build systemd
    
## 54.2 — Descargar

```bash
wget https://www.linuxfromscratch.org/patches/lfs/development/coreutils-9.11-i18n-1.patch
wget https://ftpmirror.gnu.org/coreutils/coreutils-9.11.tar.xz
```

## 54.3 — Compilar

```bash
cd /sources
tar xvf coreutils-9.11.tar.xz
cd coreutils-9.11

patch -Np1 -i ../coreutils-9.11-i18n-1.patch
autoreconf -fv
automake -af
FORCE_UNSAFE_CONFIGURE=1 ./configure \
            --prefix=/usr
make -j$(nproc)

make install
mv -v /usr/share/man/man1/chroot.1 /usr/share/man/man8/chroot.8
sed -i 's/"1"/"8"/' /usr/share/man/man8/chroot.8

```

## 54.4 — Verificar

```bash
file /usr/bin/whoami 
/usr/bin/whoami: ELF 64-bit LSB pie executable, ARM aarch64, version 1 (SYSV), dynamically linked, interpreter /lib/ld-musl-aarch64.so.1, with debug_info, not stripped
```


---

<< [Anterior](53-gperf.md) | [Índice](README.md) | [Siguiente](55-lzo.md) >>
