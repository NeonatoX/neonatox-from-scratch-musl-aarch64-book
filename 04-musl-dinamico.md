# Capítulo 4: musl 1.2.6 (dinámico)

## 4.1 — Objetivo

Compilar musl como librería compartida (`libc.so`) usando el GCC pass 1.
musl es la primera pieza del sistema que es completamente nativa (no cruzada).

## 4.2 — Dependencias

- GCC pass 1 en `$LFS/tools` (capítulo anterior)
- Binutils 2.46.1 en `$LFS/tools`
- Linux API Headers en `$LFS/tools`

---

## 4.3 — Compilar musl

```bash
cd $LFS/build
rm -rf musl-1.2.6  # limpiar por si existe
tar -xf $LFS/sources/musl-1.2.6.tar.gz
cd musl-1.2.6

# Configuramos para compilar musl usando el GCC Pass 1 que acabas de instalar en /tools
./configure \
  CROSS_COMPILE=aarch64-linux-musl- \
  --prefix=$LFS/tools \
  --target=aarch64-linux-musl \
  --enable-shared \
  --disable-static

make -j$(nproc)
make install

```

### Explicación de flags

| Flag | Significado |
|------|-------------|
| `CROSS_COMPILE=aarch64-linux-musl-` | Usa el GCC pass 1 como compilador |
| `--prefix=$LFS/tools` | Toolchain temporal |
| `--enable-shared` | `libc.so` dinámica |
| `--disable-static` | No compila `libc.a` (innecesario) |

### Verificar


```bash
echo 'int main(){}' > t.c

$LFS/tools/bin/aarch64-linux-musl-gcc t.c -o t

file t

readelf -l t | grep interpreter

```

### Resultado esperado

```bash
ELF 64-bit LSB executable, aarch64, version 1 (SYSV), dynamically linked, interpreter /lib/ld-musl-aarch64.so.1, not stripped
      [Requesting program interpreter: /lib/ld-musl-aarch64.so.1]
```

---

## 4.4 Actualizar el enlace del sysroot

Los Linux API Headers solo proporcionan las interfaces del kernel (linux/, asm/, etc.). Tras instalar musl,
los headers estándar de C (stdio.h, stdlib.h, string.h, etc.) pasan a residir en $LFS/tools/include. 
Se actualiza el enlace del sysroot para que el compilador del target utilice el conjunto completo de headers de musl.

```bash
rm -rf "$LFS/tools/aarch64-linux-musl/include"

ln -snf ../include \
    "$LFS/tools/aarch64-linux-musl/include"
```

---

## 4.5 — ¿Qué cambió?

Ahora tenemos `$LFS/tools/lib/libc.so` — la primera librería 100% musl.
El GCC pass 1 que usamos para compilarla es estático (no necesita libc).
Pero musl SÍ es dinámica: cualquier programa que enlace contra ella será
un binario musl.

---

<< [Anterior](03-gcc-minimal.md) | [Índice](README.md) | [Siguiente](05-gcc-completo.md) >>
