+++
date = "2024-03-07" 
title = "Hello World" 
description = "my blog website change to github" 
slug = "hola-mundo" 
draft = false

[taxonomies] 
  categories = ["news"] 
  tags = ["website"]

[extra] 
  page_identifier = "blog-2022-hola-mundo"
+++

🎉🎉🎉🎉🎉 Hello my web site

##### POSIX中定义的部分文件元数据

| 文件元数据 | 说明                                 |
| ---------- | ------------------------------------ |
| mode       | 文件模式，其中包括文件类型和文件权限 |
| nlink      | 指向此文件的链接个数                 |
| uid        | 文件所属用户的id                     |
| gid        | 文件所属用户组的id                   |
| size       | 文件的大小                           |
| atime      | 文件数据最近访问时间                 |
| ctime      | 文件元数据最近修改时间               |
| mtime      | 文件数据最近修改时间                 |

##### Linux中支持的文件类型

| 文件类型     | 文件用途                         |
| ------------ | -------------------------------- |
| 常规文件     | 保存数据                         |
| 目录文件     | 表示和组织一组文件               |
| 符号链接文件 | 保存符号链接(指向目标文件的路径) |
| FIFO文件     | 以队列形式传递数据,又称命名管道  |
| 套接字文件   | 用于传递数据,比FIFO文件更加灵活  |
| 字符设备文件 | 表示和访问字符设备               |
| 块设备文件   | 表示和访问块设备                 |


## Command

```shell
# 列出当前目录下所有文件和子目录的大小，包括隐藏文件。 -r表示逆序。
du -ah | sort -rh
# --max-depth=1 限制输出的深度为 1
du -ah --max-depth=1

# 以人类可读的格式（例如 KB、MB、GB）显示磁盘使用情况。
df -h
# 显示每个文件系统的 inode 使用情况，包括总 inode 数量、已用 inode 数量和可用 inode 数量。
df -i
```

### gdisk