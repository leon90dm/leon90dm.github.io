---
layout:     post
title:      "Mysql性能调优"
subtitle:   "介绍Mysql性能调优的工具"
date:       2016-10-08
author:     "BilboDai"
header-img: "img/post-bg-2015.jpg"
catalog: true
tags:
    - Mysql
    - Database
---

MySQLTuner-perl
===
[MySQLTuner-perl](http://mysqltuner.com/)可以给Mysql一个运行状态概览，同时会给出一些推荐的配置以提供Mysql的性能。同时它也是一个开源的项目，维护在[GitHub](https://github.com/major/MySQLTuner-perl)上

安装
----
- 新建一个目录

```bash
mkdir mysqltuner
```

- 下载

```bash
$ wget http://mysqltuner.pl/ -O mysqltuner.pl
$ wget https://raw.githubusercontent.com/major/MySQLTuner-perl/master/basic_passwords.txt -O basic_passwords.txt
$ wget https://raw.githubusercontent.com/major/MySQLTuner-perl/master/vulnerabilities.csv -O vulnerabilities.csv
```

下载之后会在当前目录下面得到以下文件

```bash
$ ls
basic_passwords.txt mysqltuner.pl       vulnerabilities.csv
```

运行
---
把`mysqltuner.pl`加上可执行权限

```bash
chmod +x mysqltuner.pl
```

- 本地执行

```bash
$ ./mysqltuner.pl --user root --pass root
```

- 远程执行

```bash
$ ./mysqltuner.pl --host targetDNS_IP --user admin_user --pass admin_password --forcemem 100
```

*注意*，远程连接需要指定`--forcemem`参数，这个参数后面接一个数字表示分配的内存大小单位是M，`--forcemem 100`表示分配100M

输出
---
如果执行OK，将会得到以下主要输出：

```bash
-------- Recommendations ---------------------------------------------------------------------------
General recommendations:
    Run OPTIMIZE TABLE to defragment tables for better performance
      OPTIMIZE TABLE db.table; -- can free 184 MB
    Total freed space after theses OPTIMIZE TABLE : 184 Mb
    Set up a Password for user with the following SQL statement ( SET PASSWORD FOR 'user'@'SpecificDNSorIp' = PASSWORD('secure_password'); )
    Restrict Host for user@% to user@SpecificDNSorIp
    117 CVE(s) found for your MySQL release. Consider upgrading your version !
    Reduce your overall MySQL memory footprint for system stability
    Dedicate this server to your database for highest performance.
    Configure your accounts with ip or subnets only, then update your configuration with skip-name-resolve=1
    Adjust your join queries to always utilize indexes
    Increase table_open_cache gradually to avoid file descriptor limits
    Read this before increasing table_open_cache over 64: http://bit.ly/1mi7c4C
    Beware that open_files_limit (1000000) variable
    should be greater than table_open_cache ( 2000)
Variables to adjust:
  *** MySQL's maximum memory usage is dangerously high ***
  *** Add RAM before increasing MySQL buffer variables ***
    join_buffer_size (> 256.0K, or always use indexes with joins)
    table_open_cache (> 2000)
    innodb_buffer_pool_size (>= 11G) if possible.
    innodb_buffer_pool_instances (=1)
```