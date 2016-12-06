---
title: linux查看日志常用命令
date: 2016-12-05 16:50:21
tags: 
    - linux
    - 后台
categories: 后台开发
---
# 基本命令：
> tail -n 10 test.log   查询日志尾部最后10行的日志；
  tail -n +10 test.log  查询10行之后的所有日志；
  head -n 10 test.log   查询日志文件中的头10行日志；
  head -n -10 test.log  查询日志文件除了最后10行的其他所有日志；

<!-- more -->

# 场景1：按行号查找--过滤出关键字附近的日志
--因为通常时候我们用grep拿到的日志很少，我们需要查看附近的日志。
> cat -n test.log | grep "xxx"  得到关键日志的行号
  cat -n test.log | tail -n +90|head -n 20  查询100行前后10行的日志
  -- tail -n +90   查询90行之后的日志
  -- heand -n 20   则表示在前面的查询结果里再查前20条记录

# 场景2：查找指定时间端的日志
> sed -n '/2014-12-17 16:17:20/,/2014-12-17 16:17:36/p' test.log
  -- 特别说明：上面的两个日期必须是日志中打印出来的日志，否则无效。
  grep '2014-12-17 16:17:20' test.log   可以先确定日志中是否有该时间
  cat -n test.log | grep "xxx" | more  分页打印，通过空格翻页
  cat -n test.log | grep "xxx" > xx.txt 保存到xx文件中
