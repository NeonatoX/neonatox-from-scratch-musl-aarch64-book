# Capítulo 70: Systemd 260.2

## 69.1 — Objetivo

The systemd package contains programs for controlling the startup, running, and shutdown of the system. 
   
## 69.2 — Descargar

```bash
wget https://github.com/systemd/systemd/archive/v260.2/systemd-260.2.tar.gz

```

## 69.3 — Compilar

```bash
cd /sources
tar xvf systemd-260.2.tar.gz
cd systemd-260.2

sed -i 's/SYSTEMD_TEST_DATA=%q\\nSYSTEMD_CATALOG_DIR=%q\\n/SYSTEMD_TEST_DATA=%s\\nSYSTEMD_CATALOG_DIR=%s\\n/g' meson.build
mkdir -p build && cd build

meson setup ..                \
      --prefix=/usr           \
      --buildtype=release     \
      -D libc=musl            \
      -D nss-systemd=false    \
      -D nss-myhostname=false \
      -D nss-resolve=disabled    \
      -D nss-mymachines=disabled \
      -D userdb=false         \
      -D default-dnssec=no    \
      -D firstboot=false      \
      -D install-tests=false  \
      -D ldconfig=false       \
      -D sysusers=false       \
      -D rpmmacrosdir=no      \
      -D homed=disabled       \
      -D man=disabled         \
      -D mode=release         \
      -D pam=enabled          \
      -D pamconfdir=/etc/pam.d \
      -D dev-kvm-mode=0700    \
      -D nobody-group=nogroup \
      -D sysupdate=disabled   \
      -D ukify=disabled       \
      -D tests=false          \
      -D docdir=/usr/share/doc/systemd
ninja

ninja install
```

## 69.4 — Verificar

```bash
file /usr/lib/systemd/systemd
# → ELF ... pie executable ... interpreter /lib/ld-musl-aarch64.so.1

/usr/lib/systemd/systemd --version
# → systemd 260.2

```


---

<< [Anterior](68-jinja2.md) | [Índice](README.md) | [Siguiente](70-texinfo.md) >>
