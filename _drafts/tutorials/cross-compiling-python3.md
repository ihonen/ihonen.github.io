#---
layout: post
title:  "WIP: Cross-compiling Python3 for embedded Linux"
author: "ihonen"
tags: ["python", "python3", "embedded", "linux"]
---

Recently at work I had to compile and install Python 3 for an embedded Linux system running on an ARM-based SoC. It turns out the cross-compilation process isn't exactly well documented, so I thought I would post my findings in the hopes that they might be of help to someone else.

This post assumes you're familiar with the basics of cross-compilation.

Furthermore, the following is assumed:

* `$TOOLCHAIN` is the path of your GCC-based cross-compilation toolchain
* `$SYSROOT` is the path of your toolchain's sysroot

