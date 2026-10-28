# Capítulo 41: Linux-PAM 1.7.0

## 41.1 — Objetivo

Compilar Linux-PAM 1.7.0 en `/usr`. Pluggable Authentication Modules —
sistema de autenticación modular. Necesario para shadow (login, su,
passwd con PAM).

## 41.2 — Dependencias

- gcc, make, ninja, meson (ya en el sistema)
- flex 2.6.4 (capítulo 35)
- bison 3.8.2 (capítulo 36)

## 41.3 — Descargar

```bash
wget --no-check-cert https://github.com/linux-pam/linux-pam/releases/download/v1.7.0/Linux-PAM-1.7.0.tar.xz
```

## 41.4 — Compilar

```bash
cd /sources
tar xf Linux-PAM-1.7.0.tar.xz
cd Linux-PAM-1.7.0

meson setup build \
      --prefix=/usr \
      --libdir=/usr/lib 
ninja -C build

ninja -C build install

# Configuración básica
mkdir -pv /etc/pam.d


install -vdm755 /etc/pam.d &&
cat > /etc/pam.d/system-account << "EOF" &&
# Begin /etc/pam.d/system-account

account   required    pam_unix.so

# End /etc/pam.d/system-account
EOF

cat > /etc/pam.d/system-auth << "EOF" &&
# Begin /etc/pam.d/system-auth

auth      required    pam_unix.so

# End /etc/pam.d/system-auth
EOF

cat > /etc/pam.d/system-session << "EOF" &&
# Begin /etc/pam.d/system-session

session   required    pam_unix.so

# End /etc/pam.d/system-session
EOF

cat > /etc/pam.d/system-password << "EOF"
# Begin /etc/pam.d/system-password

# use yescrypt hash for encryption, use shadow, and try to use any
# previously defined authentication token (chosen password) set by any
# prior module.
# yescrypt is not compatible with musl
#password  required    pam_unix.so       yescrypt shadow try_first_pass


password  required    pam_unix.so        sha512 shadow

# End /etc/pam.d/system-password
EOF

cat > /etc/pam.d/other << "EOF"
# Begin /etc/pam.d/other

auth        required        pam_warn.so
auth        required        pam_deny.so
account     required        pam_warn.so
account     required        pam_deny.so
password    required        pam_warn.so
password    required        pam_deny.so
session     required        pam_warn.so
session     required        pam_deny.so

# End /etc/pam.d/other
EOF
```

## 41.5 — Verificar

```bash
file /usr/lib/libpam.so.0.85.1 
/usr/lib/libpam.so.0.85.1: ELF 64-bit LSB shared object, ARM aarch64, version 1 (SYSV), dynamically linked, with debug_info, not stripped

```

## 41.6 — Notas

- PAM 1.7.0 usa meson como build system (no autotools)
- `-Ddisable-nls=true`: desactiva traducciones
- `-Dregenerate-docu=false`: evita dependencias de docbook
- Se crea `/etc/pam.d/other` con política denegada por defecto
- En musl, PAM compila sin problemas
- `libpam.so`, `libpam_misc.so`, `libpamc.so` se instalan en `/usr/lib/`


---

<< [Anterior](57-cracklib.md) | [Índice](README.md) | [Siguiente](59-flit-core.md) >>
