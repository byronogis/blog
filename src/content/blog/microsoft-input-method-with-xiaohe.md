---
title: 给微软拼音添加小鹤双拼方案
description: 给微软拼音添加小鹤双拼方案
publishedTime: 2022-10-17
modifiedTime: 2023-04-02
tags: [windows]
keywords: windows,微软拼音,小鹤双拼
---
update: 2023-03-19 12:33:17


- 首先打开注册表，找到这个路径

`计算机\HKEY_CURRENT_USER\Software\Microsoft\InputMethod\Settings\CHS`

- 然后新建一个名为 `UserDefinedDoublePinyinScheme0` 的字符串值，数值数据为

`小鹤双拼*2*^*iuvdjhcwfg^xmlnpbksqszxkrltvyovt`
