# Capítulo 5: GCC 16.1.0 Completo (C++, shared, threads)

## 5.1 — Objetivo

Recompilar GCC ahora con soporte completo: C, C++, LTO, shared libraries,
y threads POSIX. Este GCC compila contra el sysroot musl y produce binarios
musl nativos para el target, aunque el propio GCC siga siendo glibc-linked
(host == build machine). Es temporal: stage 3 rebuild nativo dentro del chroot.

## 5.2 — Dependencias

- musl 1.2.6 en `$LFS/tools` (capítulo anterior)
- Binutils 2.46.1 en `$LFS/tools`
- GCC pass 1 en `$LFS/tools` (sigue siendo el compilador activo)

---

## 5.3 — Copiar headers de musl donde GCC los espera

GCC con `--target=aarch64-linux-musl --with-sysroot=$LFS/tools` busca los
headers del sistema en `$LFS/tools/aarch64-linux-musl/include/`. musl instaló
los suyos en `$LFS/tools/include/`. Hay que copiarlos:

```bash
cp -r $LFS/tools/include/* $LFS/tools/aarch64-linux-musl/include/
```

## 5.4 — Compilar GCC completo

```bash
cd $LFS/build
rm -rf gcc-16.1.0  # empezar limpio
tar -xf $LFS/sources/gcc-16.1.0.tar.xz
cd gcc-16.1.0

# GMP, MPFR, MPC dentro del árbol (igual que pass 1)
tar -xf ../gmp-6.3.0.tar.xz && mv gmp-*/ gmp
tar -xf ../mpfr-4.2.1.tar.xz && mv mpfr-*/ mpfr
tar -xf ../mpc-1.3.1.tar.gz && mv mpc-*/ mpc

mkdir build && cd build

../configure \
  --prefix=$LFS/tools \
  --build=$(../config.guess) \
  --host=aarch64-linux-musl \
  --target=aarch64-linux-musl \
  --with-sysroot=$LFS/tools \
  --libdir=/usr/lib \
  --disable-nls \
  --enable-languages=c,c++ \
  --enable-shared \
  --enable-threads=posix \
  --enable-lto \
  --enable-__cxa_atexit \
  --disable-libsanitizer \
  --disable-libvtv \
  --disable-multilib \
  --disable-bootstrap

make -j$(nproc) 2>&1 | tee build.log

make install
```

> **Nota**: usamos `--host=aarch64-linux-musl` (glibc del build host) porque
`--host=aarch64-linux-musl` requiere que configure ejecute test programs
musl-linked en el host glibc, lo cual no es posible. El GCC resultante es
glibc-linked pero produce binarios musl vía `--with-sysroot`.
`--disable-bootstrap` evita que GCC se compile 3 veces (solo necesitamos
1 pase para stage 1).

### Explicación de flags nuevos

| Flag | Significado |
|------|-------------|
| `--enable-languages=c,c++` | C y C++ completos |
| `--enable-shared` | GCC genera y enlaza librerías compartidas |
| `--enable-threads=posix` | Soporte de hilos POSIX (usando musl) |
| `--enable-lto` | Link Time Optimization |
| `--enable-__cxa_atexit` | Manejo correcto de destructores C++ |
| `--disable-libsanitizer` | Sanitizers no disponibles en musl |
| `--disable-libvtv` | Vtable verification (no soportado en musl) |
| `--disable-bootstrap` | Un solo pase de compilación (stage 1) |
| `--host=aarch64-linux-musl` | El GCC corre en el host musl aarch64 (por ahora) |

### Prerrequisito: symlink del linker dinámico musl

Antes de compilar, asegurarse de que el linker dinámico musl exista en el
sysroot (lo necesita el build system de GCC para linkear los binarios):

```bash
ln -sf libc.so $LFS/tools/lib/ld-musl-aarch64.so.1
```

### Limpiar build anterior (si existe)

```bash
rm -rf $LFS/sources/gcc-16.1.0/build
```

### Verificar compilación simple

```bash
echo 'int main(){ return 0; }' > /tmp/test.c
$LFS/tools/bin/aarch64-linux-musl-gcc -o /tmp/test /tmp/test.c
file /tmp/test
# → ELF ... interpreter /lib/ld-musl-aarch64.so.1, musl
```

---

## 5.5 — Verificar resultado

El GCC y binutils en `$LFS/tools/bin/` son glibc-linked (temporales):

```bash
readelf -l /tools/bin/aarch64-linux-musl-gcc | grep interpreter
# → /lib64/ld-linux-aarch64.so.2  (glibc — esperado)
```

Pero los binarios que produce son musl nativos:

```bash
$LFS/tools/bin/aarch64-linux-musl-gcc hello.c -o hello
file hello
# → ... interpreter /lib/ld-musl-aarch64.so.1, musl
```

---

## 5.6 — Preparar archivos del sistema

Antes del chroot, crear los archivos mínimos que necesita el sistema.

### 5.6.1 — /etc/passwd

```bash
cat > $LFS/etc/passwd << "EOF"
root:x:0:0:root:/root:/bin/bash
bin:x:1:1:bin:/dev/null:/usr/bin/false
daemon:x:6:6:Daemon User:/dev/null:/usr/bin/false
messagebus:x:18:18:D-Bus Message Daemon User:/run/dbus:/usr/bin/false
nobody:x:65534:65534:Unprivileged User:/dev/null:/usr/bin/false
EOF
```

### 5.6.2 — /etc/group

```bash
cat > $LFS/etc/group << "EOF"
root:x:0:
bin:x:1:daemon
sys:x:2:
kmem:x:3:
tty:x:5:
daemon:x:6:
disk:x:8:
lp:x:9:
dialout:x:10:
audio:x:11:
video:x:12:
utmp:x:13:
cdrom:x:15:
adm:x:16:
messagebus:x:18:
input:x:24:
kvm:x:61:
mail:x:34:
wheel:x:97:
users:x:999:
nogroup:x:65534:
EOF
```

### 5.6.3 — /etc/passwd y /etc/shadow root

```bash
cat > $LFS/etc/shadow << "EOF"
root:::::::: 
EOF

cat > $LFS/etc/shells << "EOF"
/bin/bash
/bin/sh
EOF
```

> **Nota**: el campo de password vacío en shadow permite login sin
contraseña. Se cambia después con `passwd` dentro del chroot.

### 5.6.4 — /etc/fstab (mínimo)

```bash
cat > $LFS/etc/fstab << "EOF"
# Sistema de archivos virtuales
proc  /proc  proc  defaults  0 0
sysfs /sys   sysfs defaults  0 0
devpts /dev/pts devpts gid=5,mode=620  0 0
tmpfs /run   tmpfs defaults  0 0
EOF
```

### 5.6.5 — /etc/resolv.conf

```bash
cat > $LFS/etc/resolv.conf << "EOF"
nameserver 1.1.1.1
nameserver 8.8.8.8
EOF
```

### 5.6.6 — /etc/hosts

```bash
cat > $LFS/etc/hosts << "EOF"
127.0.0.1  localhost
::1        localhost
EOF
```

### 5.6.7 — /etc/profile

```bash
cat > $LFS/etc/profile << "EOF"
export PATH=/tools/bin:/usr/bin:/bin
export LC_ALL=POSIX

if [ "$EUID" = 0 ]; then
  export PS1='\u [ \w ]# '
else
  export PS1='\u [ \w ]\$ '
fi

alias ls='ls --color=auto'
EOF
```

## 5.7 — Estado del toolchain stage 1

| Componente | Linkado contra | Nota |
|------------|----------------|------|
| `aarch64-linux-musl-gcc` | glibc (host) | Temporal; produce binarios musl |
| `aarch64-linux-musl-as`, `ld`, `ar` | glibc (host) | Temporal |
| `/tools/lib/libc.so` | musl | Primer shared library musl del sistema |

## 5.8 — Plan para stage 2

Sin chroot: usar `aarch64-linux-musl-gcc` como cross-compiler desde el host
glibc para compilar binutils, musl, GCC, busybox, bash directamente en
`$LFS/usr`. Luego chroot limpio con sistema musl nativo.

---

## 5.9 — Backup de la toolchain stage 1

Opcional pero recomendado. Este backup permite volver a este punto sin tener
que recompilar todo desde cero.

```bash
# Desde el host (afuera del chroot)
cd $LFS
tar -cJpf $LFS/sources/tools-stage1-$(date +%Y%m%d).tar.xz tools/
```

Para restaurar:

```bash
cd $LFS
rm -rf tools
tar -xpf $LFS/sources/tools-stage1-*.tar.xz
```

Tamaño estimado: ~500-600 MB comprimido (incluye binutils + GCC completos).

---

<< [Anterior](04-musl-dinamico.md) | [Índice](README.md) | [Siguiente](06-linux-headers.md) >>
