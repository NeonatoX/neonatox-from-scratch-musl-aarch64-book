# Capítulo 48: cmake 4.3.4

## 48.1 — Objetivo

Cross platform Make.

## 48.2 — Descargar

```bash
wget --no-check-cert https://cmake.org/files/v4.3/cmake-4.3.4.tar.gz
# MD5:	971d5af514d207cd383e47499d2c6f79
```

## 48.3 — Compilar

```bash
cd /sources
tar xvf cmake-4.3.4.tar.bz2 
cd cmake-4.3.4

sed -i '/"lib64"/s/64//' Modules/GNUInstallDirs.cmake

./bootstrap --prefix=/usr \
  --system-libs \
  --mandir=/share/man \
  --no-system-jsoncpp \
  --no-system-cppdap \
  --no-system-librhash \
  --docdir=/share/doc/cmake \
  --parallel=12 \
  -- \
  -DCMAKE_USE_SYSTEM_FORM=OFF \
  -DCMAKE_USE_SYSTEM_CURSES=OFF \
  -DBUILD_CursesDialog=OFF
  
make -j$(nproc)

make install
```

## 48.4 — Verificar

```bash
file /usr/bin/cmake 
/usr/bin/cmake: ELF 64-bit LSB pie executable, ARM aarch64, version 1 (SYSV), dynamically linked, interpreter /lib/ld-musl-aarch64.so.1, with debug_info, not stripped
```

# Limpia
rm -rf Bootstrap.cmk CMakeFiles cmake_bootstrap.log

# Bootstrap desactivando ccmake (más rápido, no necesita libform)
./bootstrap --prefix=/usr -- \
  -DCMAKE_USE_SYSTEM_FORM=OFF \
  -DCMAKE_USE_SYSTEM_CURSES=OFF


---

<< [Anterior](48-libuv1.md) | [Índice](README.md) | [Siguiente](50-plocate.md) >>
