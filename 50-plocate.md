# Capítulo 50: plocate 1.1.23

## 50.1 — Objetivo

Compilar plocate 1.1.23 en `/usr`. Alternativa moderna a `locate`/`updatedb`,
compatible con bases de datos mlocate. Reemplaza a GNU findutils para la
funcionalidad de búsqueda indexada.

Busybox o Findutils ya proveen `find` y `xargs` (suficiente para el sistema base);
plocate añade `locate`/`updatedb` que busybox no tiene y se descartan de findutils.

## 50.2 — Dependencias

- gcc, ninja, meson (ya en el sistema)
- zstd 1.5.7 (capítulo 29)
- liburing (opcional, mejora rendimiento con io_uring)

## 50.3 — Descargar

```bash
wget https://plocate.sesse.net/download/plocate-1.1.24.tar.gz

```

## 50.4 — Compilar

```bash
cd /sources
rm -rf plocate-1.1.23
tar xf plocate-1.1.23.tar.gz
cd plocate-1.1.23

sed -i '/#include <linux\/stat.h>/d' io_uring_engine.h

mkdir build && cd build

meson setup .. \
      --prefix=/usr \
      --libdir=lib \
      -Dinstall_cron=false \
      -Dinstall_systemd=true \
      -Dlocategroup=locate

ninja
ninja install
```

## 50.5 — Configuración

```bash
# Grupo para la base de datos de locate
groupadd -g 21 locate

# Permisos del binario
chgrp 21 /usr/bin/plocate
chmod 2755 /usr/bin/plocate

# Archivos de configuración del sistema
cat > /etc/updatedb.conf << "EOF"
PRUNE_BIND_MOUNTS = "yes"
PRUNEFS = "9p afs anon_inodefs auto autofs bdev binfmt_misc cgroup cifs coda configfs cpuset cramfs debugfs devpts devtmpfs ecryptfs exofs ftpfs fuse fuse.encfs fuse.s3fs fuse.sshfs fusectl gfs gfs2 hugetlbfs inotifyfs iso9660 jffs2 lustre mqueue ncpfs nfs nfs4 nfsd pipefs proc ramfs rootfs rpc_pipefs securityfs selinuxfs sfs shfs smbfs sockfs sshfs sysfs tmpfs ubifs udf usbfs vboxsf"
PRUNENAMES = ".git .hg .svn"
PRUNEPATHS = "/afs /media /mnt /net /sfs /tmp /udev /var/cache /var/lib/pacman/local /var/lock /var/run /var/spool /var/tmp"
EOF

# tmpfiles.d para crear la base de datos en /run
mkdir -p /usr/lib/tmpfiles.d/
cat > /usr/lib/tmpfiles.d/plocate.conf << "EOF"
f /var/lib/plocate/plocate.db   0750 root locate
EOF

# Enlaces para compatibilidad
ln -sf plocate /usr/bin/mlocate
ln -sf plocate /usr/bin/locate
```

## 50.6 — Verificar

```bash
updatedb
plocate bash | head -5
# → debe listar archivos con "bash" en el nombre

file /usr/bin/plocate
/usr/bin/plocate: setgid ELF 64-bit LSB pie executable, ARM aarch64, version 1 (SYSV), dynamically linked, interpreter /lib/ld-musl-aarch64.so.1, with debug_info, not stripped
```

## 50.7 — Notas

- Busybox tiene `find` y `xargs`, por lo que GNU findutils **es opcional**. 
plocate provee `locate`/`updatedb` que busybox no
  tiene.
- `-Dlocategroup=locate`: el binario plocate es setgid al grupo
  `locate` para que usuarios no-root puedan leer la base de datos.
- `-Dinstall_cron=false`: se usa systemd timer en vez de cron.
- `-Dinstall_systemd=true`: instala `plocate-updatedb.timer` y
  `.service` para actualizar la base de datos periódicamente.
- El parche `plocate-fix-updatedb.timer.patch` corrige el timer de
  systemd para que funcione correctamente.
- La base de datos se almacena en `/var/lib/plocate/plocate.db`.
- liburing es opcional: si no está disponible, plocate usa readdir
  tradicional.


---

<< [Anterior](49-cmake.md) | [Índice](README.md) | [Siguiente](51-nhopkg.md) >>
