---
layout: post
title: "在 MySQL 中存储 emoji 表情"
date: 2014-11-9 17:40:33
---
这周在派的测试过程中，发现一旦发起的内容中有 Emoji 表情，就会导致服务器返回500错误。后来在查了相关资料之后，发现这是由于 MySQL 引起的。

### 原因
Emoji 表情的特殊之处是，在存储的时候，需要用到 4 个字节。而 MySQL 中常见的 utf8 字符集的 `utf8_general_ci` 这个 collate 最大只支持 3 个字节。所以为了存储 Emoji，你需要改用 `utf8mb4` 字符集。

### 具体解决
在创建表的时候，用类似的语句

```sql
CREATE TABLE table_name ... ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE utf8mb4_general_ci;
```

问题又来了，对 `utf8mb4` 字符集的支持是 __MySQL 5.5 的新功能__，而咱们数据库的版本比这个要老，所以上面的解决办法救没法应用了。

那怎么办呢？目前的解决办法是对内容进行 `Base64` 转码，读取的时候在转回来即可。然后通过缓存来降低对来回转码对效率的影响，基本解决这个问题。

### one more thing
考虑到如果自己开发一些东西玩的时候，肯定会直接装新的数据库，不会有上面的历史包袱。再来说说使用 `utf8mb4` 字符集的一些注意的事项。

在启用了 `utf8mb4` 字符集后，备份和导入就不能再用默认参数了。

用 `mysqldump` 备份时，需要加入: `mysqldump --default-character-set=utf8mb4`

而在回复备份或通过程序连接时，需要在每次链接打开之后发送下面这条 `SQL` 指令: `SET CHARSET utf8mb4`。