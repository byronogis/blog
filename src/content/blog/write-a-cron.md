---
title: 写一个定时任务
description: 写一个定时任务 cron
publishedTime: 2023-04-08
modifiedTime: 2023-04-08
tags: [linux]
keywords: linux,cron
---

## 目录

## cron

> [在线工具](https://qqe2.com/cron)

- `* * * * * * command` 分别代表: 秒, 分钟, 小时, 日期, 月份, 星期, 命令
- `* * * * * command` 分别代表: 分钟, 小时, 日期, 月份, 星期, 命令
- `*` 代表任意值
- `*/2` 代表每2个单位执行一次
- `1-10` 代表1到10
- `1,2,3` 代表1,2,3
- `?` 只能在日和星期（周）中指定使用，其作用为不指定

## 例子

- 每小时零分零秒执行一次命令 `0 0 * * * * command`

