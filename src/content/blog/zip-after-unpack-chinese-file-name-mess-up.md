---
title: linux-zip解压后中文乱码
description: 由windows下生成的zip格式的压缩包，在linux下解压后中文乱码
publishedTime: 2022-06-12
modifiedTime: 2023-04-02
tags: [linux,windows]
keywords: linux,arch,ark,unzip,zip
---

> [zip compressed package is garbled](https://wiki.archlinux.org/title/Localization/Simplified_Chinese#zip_compressed_package_is_garbled)

> Under non-utf8 coding environments (generally the Chinese environment under Windows), do not use zip for compressions (7z is recommended).

由windows下生成的zip格式的压缩包，在linux下解压后中文乱码，使用额外命令行参数-O来解决。

```bash
unzip -O gbk file.zip
```
