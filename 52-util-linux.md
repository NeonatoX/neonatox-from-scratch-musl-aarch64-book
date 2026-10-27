# Capítulo 52: Util-linux 2.52.1

## 52.1 — Objetivo

Compilar util-linux 2.52.1 en `/usr`. Proporciona herramientas
esenciales del sistema que busybox no tiene (o tiene limitadas):
`lsblk`, `findmnt`, `fstrim`, `wipefs`, `sfdisk`, `flock`,
`fallocate`, `blkdiscard`, `uuidgen`, `whereis`, `rename`,
`blockdev`, `lsns`, `nsenter`, `unshare`, etc.

También reemplaza selectivamente applets de busybox con versiones
más completas: `mount`, `umount`, `fdisk`, `blkid`, `losetup`,
`dmesg`, `agetty`, `hwclock`, `swapon`.

## 52.2 — Relación con busybox

No es necesario deshabilitar nada en busybox — los applets que
util-linux reemplaza simplemente no se usan.

## 52.3 — Dependencias

- gcc, make (ya en el sistema)
- Python 3.14.6 (capítulo 37)

## 52.4 — Descargar

```bash
wget https://www.kernel.org/pub/linux/utils/util-linux/v2.52/util-linux-2.52.1.tar.xz
```

## 52.5 — Compilar

```bash
cd /sources
rm -rf util-linux-2.52.1
tar xf util-linux-2.52.1.tar.xz
cd util-linux-2.52.1

./configure --prefix=/usr              \
            --runstatedir=/run         \
            --disable-login            \
            --disable-nologin          \
            --disable-su               \
            --disable-static           \
            --disable-makeinstall-chown \
            --without-systemd          \
            --without-systemdsystemunitdir \
            --docdir=/usr/share/doc/util-linux \
            ADJTIME_PATH=/var/lib/hwclock/adjtime

make -j$(nproc)
make install
```

## 52.6 — Verificar

```bash
ldd /usr/bin/mount | grep musl
# → /lib/ld-musl-x86_64.so.1

/usr/bin/lsblk --version
# → lsblk from util-linux 2.52.1

/usr/bin/findmnt --help > /dev/null && echo OK
# → OK
```

## 52.7 — Notas

### Herramientas que reemplazan applets de busybox

| Applet busybox | util-linux | Motivo del reemplazo |
|----------------|------------|----------------------|
| `mount` | `mount` | x-mount options, bind recursion, `--make-shared` |
| `umount` | `umount` | `--lazy`, `--force`, `--recursive` |
| `fdisk` | `fdisk` | GPT, MBR, JSON output, `--wipe` |
| `blkid` | `blkid` | `-o export`, `-p` probing, superblock details |
| `losetup` | `losetup` | `--direct-io`, `--sector-size`, `--json` |
| `swapon`/`swapoff` | `swapon`/`swapoff` | `--show`, discard policy, priority |
| `dmesg` | `dmesg` | `-l` level filter, `--human`, `--json` |
| `hwclock` | `hwclock` | `--delay`, `--update-drift`, parametric modes |
| `agetty` | `agetty` | `--autologin`, `--remote` (requerido por systemd) |

### Herramientas que busybox no tiene (indispensables)

`lsblk`, `findmnt`, `fstrim`, `blkdiscard`, `wipefs`, `sfdisk`,
`cfdisk`, `partx`, `blockdev`, `zramctl`, `flock`, `fallocate`,
`lsns`, `nsenter`, `unshare`, `setpriv`, `rfkill`, `wdctl`,
`lscpu`, `lsmem`, `uuidgen`, `uuidd`, `rename`, `whereis`,
`column`, `fsfreeze`, `swaplabel`, `setterm`, `lsfd`, `lslocks`,
`choom`, `prlimit`, `fincore`, `fadvise`, `lsirq`, `pipesz`,
`coresched`, `exch`, `copyfilerange`, `blkpr`, `blkzone`,
`scriptlive`, `lsipc`, `lslogins`, `getino`

### Otras notas

- `--disable-login`, `--disable-nologin`, `--disable-su`: shadow
  ya provee estas herramientas (capítulo 67).
- Sistema usr-merged: `--prefix=/usr` es suficiente — los binarios
  van a `/usr/bin/` y `/usr/sbin/`, accesibles vía `/bin -> usr/bin`
  y `/sbin -> usr/sbin`.
- `--without-systemd`: systemd no está presente durante stage 2.
- Python ya está disponible; util-linux lo usa para `lsfd` y
  herramientas de scripting.
- `libblkid`, `libmount`, `libuuid` se instalan como shared
  libraries en `/usr/lib/`.
- `switch_root` es crítico para initramfs → transición al sistema
  real.
- En musl, todas las herramientas de util-linux compilan y funcionan
  sin problemas.


---

<< [Anterior](51-nhopkg.md) | [Índice](README.md) | [Siguiente](53-gperf.md) >>
