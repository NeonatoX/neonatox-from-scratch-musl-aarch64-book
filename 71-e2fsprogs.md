# Capítulo 71: e2fsprogs 1.47.4

## 71.1 — Objetivo

Compilar e2fsprogs 1.47.4 en `/usr`. Proporciona `mkfs.ext2/3/4`,
`fsck.ext2/3/4`, `e2fsck`, `tune2fs`, `resize2fs`, `debugfs`,
`dumpe2fs`, `lsattr`, `chattr`.

## 71.2 — Dependencias

- gcc, make (ya en el sistema)

## 71.3 — Descargar

```bash
wget https://www.kernel.org/pub/linux/kernel/people/tytso/e2fsprogs/v1.47.4/e2fsprogs-1.47.4.tar.xz
```

## 71.4 — Compilar

```bash
cd /sources
tar xf e2fsprogs-1.47.4.tar.xz
cd e2fsprogs-1.47.4

mkdir -v build
cd build

../configure --prefix=/usr           \
             --sysconfdir=/etc       \
             --enable-elf-shlibs     \
             --disable-fsck          \
             --disable-libblkid      \
             --disable-libuuid       \
             --disable-uuidd         \
             --disable-static        \
             --docdir=/usr/share/doc/e2fsprogs

make
make install
```

## 71.5 — Verificar

```bash
file /usr/sbin/mkfs.ext4
# → ELF 64-bit LSB executable, ARM aarch64, ... dynamically linked

/usr/sbin/mkfs.ext4 -V
# → mke2fs 1.47.4
```

## 71.6 — Notas

- `--enable-elf-shlibs`: buildea shared libraries `libext2fs.so`, `libcom_err.so`, etc.
- `--disable-libblkid` y `--disable-libuuid`: util-linux provee bibliotecas más completas
- `--disable-fsck`: busybox provee `fsck`
- `--disable-uuidd`: no necesario en sistema monousuario
- e2fsprogs instala también `blkid` y `uuidgen`; los de util-linux son más completos
- En musl, e2fsprogs compila sin problemas (usa syscalls directas)


---

<< [Anterior](70-texinfo.md) | [Índice](README.md) | [Siguiente](72-btrfs-progs.md) >>
