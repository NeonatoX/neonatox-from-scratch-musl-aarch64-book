# Capítulo 8: musl 1.2.6 (Stage 2 — $LFS/usr)

## 8.1 — Objetivo

Compilar musl como shared library + static para el sistema final en
`$LFS/usr`. Esta será la libc primaria del sistema.

## 8.2 — Dependencias

- `aarch64-linux-musl-gcc` en `$LFS/tools/bin/`
- Linux API headers en `$LFS/usr/include`

---

## 8.3 — Compilar

```bash
cd $LFS/sources
rm -rf musl-1.2.6
tar -xf musl-1.2.6.tar.gz
cd musl-1.2.6

CC=aarch64-linux-musl-gcc \
./configure \
  --prefix=/usr \
  --target=aarch64-linux-musl \
  --enable-shared \
  --enable-static

make -j$(nproc) 2>&1 | tee build.log

make DESTDIR=$LFS install
```

### Post-instalación — symlinks del linker dinámico

Primero copiar el binario real (musl install no siempre copia libc.so):

```bash
cp $LFS/sources/musl-1.2.6/lib/libc.so $LFS/usr/lib/libc.so
```

Luego crear los symlinks:

```bash
ln -sf libc.so $LFS/usr/lib/ld-musl-aarch64.so.1

# GCC busca el linker dinámico en /lib, no en /usr/lib
mkdir -p $LFS/lib
ln -sf ../usr/lib/libc.so $LFS/lib/libc.so
ln -sf ../usr/lib/ld-musl-aarch64.so.1 $LFS/lib/ld-musl-aarch64.so.1
```

> **Advertencia**: no usar `ln -sf` **después** del `cp` en el mismo
directorio (`$LFS/usr/lib/`), porque sobrescribe `libc.so` con un
symlink roto. Primero `cp`, luego los symlinks en orden.

---

## 8.4 — Verificar

```bash
file $LFS/usr/lib/libc.so
# → ELF 64-bit LSB shared object, ... dynamically linked

ls -la $LFS/usr/lib/libc.a
# → static library (2.7 MB aprox)
```

### Probar que el cross-compiler enlaza dinámico

```bash
echo 'int main(){return 0;}' | $LFS/tools/bin/aarch64-linux-musl-gcc -x c - -o /tmp/test2
file /tmp/test2
# → ELF ... interpreter /lib/ld-musl-aarch64.so.1, dynamically linked
```

---

<< [Anterior](07-binutils-stage2.md) | [Índice](README.md) | [Siguiente](09-gcc-stage2.md) >>
