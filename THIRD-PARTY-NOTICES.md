# Third-party notices

## Mesa vc4 driver: `FileSets/VersionIndependent/vc4_dri.so`

Built from [Mesa](https://mesa3d.org) 24.0.7 with the patches Venus OS applies
(see `build/README.md`). The vc4 driver and the core Mesa library are under the
MIT license:

```
Copyright © 2014 Broadcom
Copyright (C) 2012 Rob Clark <robclark@freedesktop.org>
Copyright (C) 1999-2007 Brian Paul   All Rights Reserved.

Permission is hereby granted, free of charge, to any person obtaining a
copy of this software and associated documentation files (the "Software"),
to deal in the Software without restriction, including without limitation
the rights to use, copy, modify, merge, publish, distribute, sublicense,
and/or sell copies of the Software, and to permit persons to whom the
Software is furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice (including the next
paragraph) shall be included in all copies or substantial portions of the
Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT.  IN NO EVENT SHALL
THE AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING
FROM, OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS
IN THE SOFTWARE.
```

Mesa consists of several components with their own copyrights; see
[Mesa's license page](https://docs.mesa3d.org/license.html). Source:
<https://archive.mesa3d.org/mesa-24.0.7.tar.xz>.

## Device tree overlays: `vc4-kms-v3d.dtbo`, `vc4-kms-dsi-7inch.dtbo`

Compiled device tree overlays from the Raspberry Pi Linux kernel, licensed under
GPL-2.0. Taken from the Raspberry Pi firmware release
[1.20220308](https://github.com/raspberrypi/firmware/tree/1.20220308/boot/overlays),
built from [raspberrypi/linux dc6771425e96](https://github.com/raspberrypi/linux/tree/dc6771425e9604650d1d57f7c69948be405f59a5/arch/arm/boot/dts/overlays).
These overlay sources are identical to the ones in the Venus OS 5.10.110 kernel
([victronenergy/linux 3859262148951](https://github.com/victronenergy/linux/tree/3859262148951d08d011c96d637383883c451739/arch/arm/boot/dts/overlays)).

## Not included

gui-v2 and the Qt libraries are **not** part of this package. They are
installed from the Victron Energy package feed. gui-v2 is licensed under the
[Victron Energy OS license v1](https://github.com/victronenergy/gui-v2/blob/main/LICENSE.txt),
which only allows use in systems built around Victron Energy products.
