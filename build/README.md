# Building `vc4_dri.so`

The driver must match the mesa version of the firmware (Venus OS v3.80 and
later: mesa 24.0.7). It is a cross build of mesa with only the vc4 Gallium
driver, using the target sysroot of the official Venus OS SDK.

## Inputs

- Mesa 24.0.7: <https://archive.mesa3d.org/mesa-24.0.7.tar.xz>
  (sha256 `7454425f1ed4a6f1b5b107e1672b30c88b22ea0efea000ae2c7d96db93f6c26a`)
- The patches Venus OS applies, in this order, from
  [openembedded-core b3.80](https://github.com/victronenergy/openembedded-core/tree/b3.80/meta/recipes-graphics/mesa/files):
  `0001-meson-misdetects-64bit-atomics-on-mips-clang.patch`,
  `0001-drisw-fix-build-without-dri3.patch`,
  `0002-glxext-don-t-try-zink-if-not-enabled-in-mesa.patch`,
  `0001-Revert-meson-do-not-pull-in-clc-for-clover.patch`,
  `0001-c11-threads-fix-build-on-c23.patch`,
  `CVE-2026-40393-01.patch` … `CVE-2026-40393-04.patch`
- Then `patches/0001-mesa-align-the-matrix-stack.patch` from this directory.
  Mesa allocates the GL matrix stack with `calloc()`, which on 32-bit ARM is
  only 8-byte aligned, while `GLmatrix` asks for 16. Clang then uses 16-byte
  aligned NEON stores and the driver dies with SIGBUS (alignment exception)
  as soon as gui-v2 creates its GL context. GCC builds don't hit this, but the
  patch is harmless there too.
- Sysroot `sysroots/cortexa8hf-neon-ve-linux-gnueabi` from
  [venus-scarthgap-x86_64-arm-cortexa8hf-neon-toolchain-v3.85.sh](https://updates.victronenergy.com/feeds/venus/release/sdk/)
  (the archive after the `MARKER:` line is a tar.xz; the sysroot has gcc 13.5,
  libdrm, expat, zlib and the mesa 24.0.7 headers)

## Build

On Linux, `source` the SDK environment and use its gcc and meson cross file.
The included binary was built with clang and lld instead, using
`cross-venus-armhf.ini.in` (replace the `@...@` placeholders). With clang the
libstdc++ headers must be passed explicitly, as in that file.

```sh
meson setup build-vc4 --cross-file cross-venus-armhf.ini --prefix=/usr --libdir=lib \
  --buildtype=plain -Dplatforms= -Dgallium-drivers=vc4 -Dvulkan-drivers= \
  -Dglx=disabled -Degl=enabled -Dgbm=enabled -Dgles1=disabled -Dgles2=enabled \
  -Dopengl=true -Dshared-glapi=enabled -Dglvnd=false -Dllvm=disabled \
  -Dxmlconfig=enabled -Dexpat=enabled -Dzlib=enabled -Dzstd=disabled \
  -Dvalgrind=disabled -Dlibunwind=disabled -Dlmsensors=disabled -Dbuild-tests=false \
  -Dgallium-va=disabled -Dgallium-vdpau=disabled -Dgallium-xa=disabled \
  -Dgallium-nine=false -Dgallium-opencl=disabled -Dosmesa=false -Dshader-cache=enabled \
  -Dvideo-codecs= -Dandroid-libbacktrace=disabled -Dmicrosoft-clc=disabled \
  -Dintel-clc=disabled -Dperfetto=false
ninja -C build-vc4 src/gallium/targets/dri/libgallium_dri.so
cp build-vc4/src/gallium/targets/dri/libgallium_dri.so vc4_dri.so
```

`--buildtype=plain` with `-O2` matches the OpenEmbedded build (asserts stay
enabled). Link with `-Wl,-s` or strip the result.

## Check the result

It must be an ARM EABI5 hard-float shared object exporting
`__driDriverGetExtensions_vc4`, needing only libraries the firmware has:
`libglapi.so.0 libdrm.so.2 libexpat.so.1 libz.so.1 libstdc++.so.6 libm.so.6
libgcc_s.so.1 libc.so.6` (and `ld-linux-armhf.so.3`), with symbol versions up to
GLIBC_2.38 and GLIBCXX_3.4.29, like the firmware's own mesa-megadriver.

The included binary: 11136268 bytes, sha256
`95085330b09a62235262f992923af0328bea147a93cd36a93f3fc28c0f44bcfc`.
(The v1.0~1 build without the matrix stack patch, sha256 `2e512a63…8d99`,
crashes.)
