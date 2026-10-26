# Capítulo 1: Preparación del Entorno

## 1.1 — Objetivo

Tener un directorio `$LFS=/mnt` con la estructura base y el cross-compiler
semilla (`musl-cross`) listo para empezar a compilar.

## 1.2 — Concepto clave

musl-cross es un compilador que **corre con glibc** (el idioma de tu PC) pero
**produce binarios musl**. Lo usamos desde el host para compilar los primeros
componentes de la toolchain. Cuando tengamos un compilador nativo musl, haremos
chroot y seguiremos desde ahí.

## 1.3 — Requisitos

- Tu sistema normal (el que tiene `/opt/cross-aarch64-linux-musl/`)
- Un disco extra (10 GB mínimo)
- Conexión a internet (para descargar tarballs)
- `gcc`, `make`, `wget`, `tar` — herramientas básicas del host

---

## 1.4 — Preparar el disco destino

Identificar el disco (`lsblk`) y particionar:

```bash
# fdisk /dev/sdX
```

Crear una partición primaria tipo Linux.

Formatear como ext4:

```bash
# mkfs.ext4 /dev/sdX1
```

Montar:

```bash
# mount /dev/sdX1 /mnt
```

---

## 1.5 — Crear estructura LFS

Estructura completa del sistema (NeonatoX / LFS). Usamos la misma que
`01-base-neonatox.sh` del proyecto neonatox-bootstrap.

```bash
export LFS=/mnt

# Directorios de toolchain (temporales)
mkdir -pv $LFS/{tools,sources}

# usr merge
mkdir -pv $LFS/usr/{bin,lib,lib32}
ln -sfv usr/bin   $LFS/bin
ln -sfv usr/bin   $LFS/sbin
ln -sfv usr/lib   $LFS/lib
ln -sfv bin       $LFS/usr/sbin

# Sistemas de archivos virtuales
mkdir -pv $LFS/{dev,proc,sys,run}

# Directorios del sistema
mkdir -pv $LFS/{boot,home,mnt,opt,srv}

# Configuración
mkdir -pv $LFS/etc/{opt,sysconfig,skel,profile.d}
mkdir -pv $LFS/etc/polkit-1/rules.d

# Librerías de kernel
mkdir -pv $LFS/lib/firmware

# Media
mkdir -pv $LFS/media/{floppy,cdrom}

# usr/share completo
mkdir -pv $LFS/usr/{,local/}{include,src}
mkdir -pv $LFS/usr/lib/locale
mkdir -pv $LFS/usr/local/{bin,lib,sbin}
mkdir -pv $LFS/usr/{,local/}share/{color,dict,doc,info,locale,man}
mkdir -pv $LFS/usr/{,local/}share/{misc,terminfo,zoneinfo}
mkdir -pv $LFS/usr/{,local/}share/man/man{1..8}

# var
mkdir -pv $LFS/var/{cache,local,log,mail,opt,spool}
mkdir -pv $LFS/var/lib/{color,misc,locate}
ln -sfv /run     $LFS/var/run
ln -sfv /run/lock $LFS/var/lock

# root y tmp
install -dv -m 0750 $LFS/root
install -dv -m 1777 $LFS/tmp $LFS/var/tmp
```

**Nota**: para la toolchain stage 1 (capítulos 2-5) solo se necesita la
estructura mínima. Pero tener todo desde el principio no molesta y deja el
sistema listo cuando completemos la instalación.

**Importante — musl y el directorio lib64**: musl no busca bibliotecas en
`/usr/lib64`. A diferencia de glibc, el loader de musl solo examina
`/lib`, `/usr/lib` y las rutas listadas en `/etc/ld-musl-aarch64.path`.
GCC por defecto instala bibliotecas de 64 bits en `/usr/lib64`, lo que
rompe todos los binarios C++ dinámicos dentro del chroot. Por eso:
- NO se crea `/usr/lib64` en la estructura (solo `/lib64 -> usr/lib`
  para mantener compatibilidad con binarios que usan ruta absoluta `/lib64`).
- **Toda compilación de GCC** debe incluir `--libdir=/usr/lib`.
- Si ya compilaste GCC sin esa opción, migra las bibliotecas de
  `/usr/lib64` a `/usr/lib` con:
  ```
  cd /usr/lib64
  for f in *; do [ ! -e "/usr/lib/$f" ] && mv "$f" "/usr/lib/$f"; done
  cd /usr && rmdir lib64 && ln -sf lib lib64
  ```
  O crea `/etc/ld-musl-aarch64.path` con el contenido `/usr/lib64`.
  > **Cuidado**: ese archivo **reemplaza** las rutas por defecto de musl,
  > no las extiende. Si solo pones `/usr/lib64`, musl deja de buscar en
  > `/lib` y `/usr/lib`, rompiendo todo el sistema.

---

## 1.6 — Copiar musl-cross

```bash
cp -a /opt/cross-aarch64-linux-musl/* $LFS/cross/
```

Verificar:

```bash
$LFS/cross/bin/aarch64-linux-musl-gcc --version
$LFS/cross/bin/aarch64-linux-musl-gcc -dumpmachine
# → aarch64-linux-musl


Este compilador depende del glibc del host para funcionar. Eso es normal.
Lo usamos solo para la etapa inicial (compilar musl dinámico y GCC nativo).

---

## 1.7 — Variables de entorno


export PATH="$LFS/cross/bin:$PATH"
export MAKEFLAGS="-j$(nproc)"
export LC_ALL=POSIX


Agregar opcionalmente a `~/.bashrc` para no tener que escribirlas cada vez.

---

Verificar integridad (sha256):

```bash
sha256sum musl-1.2.5.tar.gz
```

---

## 1.9 — Estructura final

```bash
/mnt/
├── cross/             # musl-cross semilla (glibc-host dependiente)
│   ├── bin/
│   ├── lib/
│   ├── libexec/
│   └── ...
├── tools/             # Toolchain temporal (stage 1) — vacío por ahora
├── sources/           # Tarballs descargados
├── build/             # Directorios de compilación
└── usr/               # Sistema final (stage 2)
```

---

## 1.10 — Verificación final

```bash
echo $LFS       # → /mnt
echo $PATH      # → .../cross/bin:...
aarch64-linux-musl-gcc --version
ls $LFS/sources/*.tar.*   # tarballs listos
```

---


**Nota**: musl-cross se compila en el host (con glibc). El binario resultante
se copia a `$LFS/cross/`. No es un paquete que se instale con nhopkg dentro
del sistema musl, sino una herramienta externa de bootstrap.


---

<< Anterior | [Índice](README.md) | [Siguiente](02-binutils.md) >>
