---
title: MySQL Full Unicode Support
date: 2020-08-11 16:13:11
categories: [Database]
tags: [MySQL, Moodle]
---
Firstly, to check that which tables need to change the char set.

``` sql
SELECT * FROM INFORMATION_SCHEMA.TABLES WHERE TABLE_SCHEMA= "moodle" AND TABLE_TYPE="BASE TABLE" AND `TABLE_COLLATION` <> "utf8mb4_unicode_ci"
```
And then, generate the SQL which need to be executed.

``` sql
SELECT CONCAT('ALTER TABLE `', TABLE_NAME,'` CONVERT TO CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;') AS mySQL FROM INFORMATION_SCHEMA.TABLES WHERE TABLE_SCHEMA= "moodle" AND TABLE_TYPE="BASE TABLE" AND `TABLE_COLLATION` <> "utf8mb4_unicode_ci"
```

<!--more-->

Some useful SQL

### Alter the Database char set

``` sql
ALTER DATABASE moodle CHARACTER SET = utf8mb4 COLLATE = utf8mb4_unicode_ci;
```
### Alter the Specific Table char set

``` sql
ALTER TABLE mdl_zoom_meeting_participants CONVERT TO CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
```

### You May Encounter with errors

``` sql
ALTER TABLE `mdl_user_devices` CHANGE `pushid` `pushid` VARCHAR(191) CHARACTER SET utf8 COLLATE utf8_unicode_ci NOT NULL DEFAULT '';

```

[Moodle MySQL](https://docs.moodle.org/39/en/MySQL)

[Moodle MySQL Full Unicode Support](https://docs.moodle.org/39/en/MySQL_full_unicode_support)



