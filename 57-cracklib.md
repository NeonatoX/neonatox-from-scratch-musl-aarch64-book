# Capítulo 57: Cracklib 2.10.3

# 57.1 - Objetivo

Password Checking Library.

## 57.3 — Descargar

```bash
wget --no-check-cert https://github.com/cracklib/cracklib/releases/download/v2.10.3/cracklib-2.10.3.tar.xz
wget --no-check-cert https://github.com/cracklib/cracklib/releases/download/v2.10.3/cracklib-words-2.10.3.xz
```

## 57.4 — Compilar

```bash
cd /sources
rm -rf cracklib-2.10.3
tar -xf cracklib-2.10.3.tar.xz
cd cracklib-2.10.3

./configure --prefix=/usr --disable-static --with-default-dict=/usr/lib/cracklib/pw_dict
make -j$(nproc)

make install
xzcat ../cracklib-words-2.10.3.xz  > /usr/share/dict/cracklib-words      
ln -v -sf cracklib-words /usr/share/dict/words               
echo $(hostname) >>      /usr/share/dict/cracklib-extra-words
install -v -m755 -d      /usr/lib/cracklib                   
create-cracklib-dict     /usr/share/dict/cracklib-words  /usr/share/dict/cracklib-extra-words

```

## 57.5 — Verificar

```bash
file /usr/lib/libcrack.so.2.9.0 
/usr/lib/libcrack.so.2.9.0: ELF 64-bit LSB shared object, ARM aarch64, version 1 (SYSV), dynamically linked, with debug_info, not stripped
```

---

<< [Anterior](56-cpio.md) | [Índice](README.md) | [Siguiente](58-linux-pam.md) >>
