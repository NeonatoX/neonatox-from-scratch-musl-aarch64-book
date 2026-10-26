# Capítulo 13: Chroot con Sistema musl Nativo

## 13.1 — Objetivo

Por primera vez, hacer chroot a `$LFS` usando exclusivamente binarios
musl nativos. No hay glibc bind-mounteado, no hay nada del host.

## 13.2 — Prerrequisitos

Antes del chroot, verificar:

```
# Linker dinámico musl en /lib
ls -la $LFS/lib/ld-musl-aarch64.so.1 $LFS/usr/lib/ld-musl-aarch64.so.1
# → /lib/ld-musl-aarch64.so.1 → ../usr/lib/ld-musl-aarch64.so.1
# → /usr/lib/ld-musl-aarch64.so.1 → libc.so  (¡NO circular!)

# Shell estático musl
file $LFS/bin/bash
# → ... statically linked, musl

file $LFS/bin/busybox
# → ... statically linked, musl
```

### Archivos del sistema (ya creados en stage 1)

```
ls $LFS/etc/passwd $LFS/etc/group $LFS/etc/shadow $LFS/etc/fstab
```

---

## 13.3 — Montar sistemas de archivos virtuales

```
mount -v --bind /dev $LFS/dev
mount -vt devpts devpts $LFS/dev/pts -o gid=5,mode=620
mount -vt proc proc $LFS/proc
mount -vt sysfs sysfs $LFS/sys
mount -vt tmpfs tmpfs $LFS/run
```

---

## 13.4 — Chroot

```
chroot "$LFS" /usr/bin/busybox env -i \
  HOME=/root \
  TERM="$TERM" \
  PS1='(musl chroot) \u [ \w ]# ' \
  PATH=/usr/local/bin:/usr/bin:/bin:/sbin \
  LC_ALL=POSIX \
  /bin/busybox sh -l
```

Si todo funciona, deberías ver:

```
(musl chroot) root [ / ]#
```

---

## 13.5 — Verificación post-chroot

Dentro del chroot, verificar que el compilador y el linker dinámico
funcionan correctamente:

```
gcc --version
# → gcc (GCC) 16.1.0  (musl nativo)

gcc -dumpmachine
# → aarch64-linux-musl

echo 'int main(){}' | gcc -x c - -o /tmp/test
# → debe compilar sin errores

file /tmp/test
# → ELF ... dynamically linked, interpreter /lib/ld-musl-aarch64.so.1
```

### Si el linker dinámico falla

Si `gcc` falla con:

```
bash: /usr/bin/gcc: cannot execute: required file not found
```

La causa es que `ld-musl-aarch64.so.1` es un symlink circular (apunta
a sí mismo). Corregir desde FUERA del chroot:

```
rm -f $LFS/usr/lib/ld-musl-aarch64.so.1
ln -sf libc.so $LFS/usr/lib/ld-musl-aarch64.so.1
```

La cadena correcta es:

- `$LFS/lib/ld-musl-aarch64.so.1 → ../usr/lib/ld-musl-aarch64.so.1`
- `$LFS/usr/lib/ld-musl-aarch64.so.1 → libc.so` (archivo real)

Verificar con:

```
ls -la $LFS/lib/ld-musl-aarch64.so.1 $LFS/usr/lib/ld-musl-aarch64.so.1
readelf -l $LFS/usr/bin/gcc | grep interpreter
# → Requesting program interpreter: /lib/ld-musl-aarch64.so.1
```

### Si chroot falla

Error típico: `chroot: failed to run command '/bin/bash': No such file or directory`

Causas posibles:
- `$LFS/bin/bash` no es ejecutable o no existe
- `$LFS/bin/busybox` no existe (alternativa: `chroot ... /bin/busybox sh`)
- El linker `$LFS/lib/ld-musl-aarch64.so.1` no está bien configurado

Solución de emergencia: entrar con busybox ash:

```
chroot "$LFS" /bin/busybox ash
```

---

<< [Anterior](12-archivos-sistema.md) | [Índice](README.md) | [Siguiente](13-ncurses.md) >>
