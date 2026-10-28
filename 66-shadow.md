# Capítulo 66: shadow 4.19.4

## 66.1 — Objetivo

Compilar shadow 4.19.4 en `/usr`. Proporciona `useradd`, `groupadd`,
`passwd`, `login`, `su`, `chpasswd`, `chsh`, `chfn`, `newgrp` y otras
herramientas de gestión de usuarios y grupos.

## 66.2 — Dependencias

- gcc, make (ya en el sistema)
- Linux-PAM 1.7.0 (capítulo 40)
- **No depende** de cracklib (no se compila)

## 66.3 — Descargar

```bash
wget https://github.com/shadow-maint/shadow/releases/download/4.19.4/shadow-4.19.4.tar.xz
```

## 66.4 — Compilar

```bash
cd /sources
tar xf shadow-4.19.4.tar.xz
cd shadow-4.19.4

sed -i 's/groups$(EXEEXT) //' src/Makefile.in         
find man -name Makefile.in -exec sed -i 's/groups\.1 / /'   {} \;
find man -name Makefile.in -exec sed -i 's/getspnam\.3 / /' {} \;
find man -name Makefile.in -exec sed -i 's/passwd\.5 / /'   {} \;
sed -e 's@#ENCRYPT_METHOD DES@ENCRYPT_METHOD YESCRYPT@' \
    -e 's@/var/spool/mail@/var/mail@'                   \
    -e '/PATH=/{s@/sbin:@@;s@/bin:@@}'                  \
    -i etc/login.defs                                  
./configure --sysconfdir=/etc   \
            --disable-static    \
            --without-libbsd    \
            --with-libpam       \
            --disable-logind    \
            --with-{b,yes}crypt
make -j$(nproc)

make install
```

## 66.5 — Configuración

```bash
pwconv
grpconv

sed -i 's/^#SUB_GID_COUNT.*/SUB_GID_COUNT 100000/' /etc/login.defs
sed -i 's/^#SUB_UID_COUNT.*/SUB_UID_COUNT 100000/' /etc/login.defs

# Configuración PAM para shadow
cat > /etc/pam.d/login << "EOF"
auth     required       pam_unix.so
account  required       pam_unix.so
password required       pam_unix.so
session  required       pam_unix.so
EOF

cat > /etc/pam.d/passwd << "EOF"
auth     required       pam_unix.so
account  required       pam_unix.so
password required       pam_unix.so
EOF

cat > /etc/pam.d/su << "EOF"
auth     sufficient     pam_rootok.so
auth     required       pam_unix.so
account  required       pam_unix.so
session  required       pam_unix.so
EOF

cat > /etc/pam.d/system-password << "EOF"
password  required    pam_unix.so        sha512 shadow
EOF
```

## 66.6 — Notas

### Reemplazo de applets de busybox

Shadow **reemplaza selectivamente** estos applets de busybox por
versiones más completas con soporte PAM:

| Herramienta | Shadow | Busybox | Motivo |
|-------------|--------|---------|--------|
| `login`     | reemplaza | fallback | login con PAM |
| `su`        | reemplaza | fallback | su con PAM |
| `passwd`    | reemplaza | fallback | passwd con PAM |
| `useradd`   | reemplaza | no tiene | shadow es indispensable |
| `groupadd`  | reemplaza | no tiene | shadow es indispensable |
| `chsh`      | reemplaza | applet | versión PAM |
| `chfn`      | reemplaza | applet | versión PAM |
| `newgrp`    | reemplaza | applet | versión PAM |
| `groups`    | **no se instala** | applet | se elimina del build |
| `vipw`      | reemplaza | applet | editor protegido |

El resto de applets de busybox (sed, grep, awk, vi, ls, cp, mv,
tar, gzip, etc.) **no se tocan** — busybox sigue proveyéndolos.

### Otras notas

- `--enable-pam` habilita autenticación vía Linux-PAM
- `--disable-account-tools-setuid` evita setuid en useradd/groupadd
- `SUB_GID_COUNT` y `SUB_UID_COUNT` para contenedores (subordinate IDs)
- Los archivos PAM en `/etc/pam.d/` son configuración mínima; deben
  refinarse después según necesidades del sistema (sudo, sshd, etc.)
- `pam_unix.so` con `sha512` es el hash por defecto. No usar `yescrypt`
  porque `pam_unix.so` compilado contra musl no lo soporta.
- No incluir `pam_pwquality.so` a menos que se compile cracklib
  previamente.


---

<< [Anterior](65-libcap.md) | [Índice](README.md) | [Siguiente](67-markupsafe.md) >>
