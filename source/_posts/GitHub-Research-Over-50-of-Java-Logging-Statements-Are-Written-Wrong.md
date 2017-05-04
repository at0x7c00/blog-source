---
title: 'GitHub调查: 超过50%的Java日志语句写法错误'
date: 2017-03-01 10:43:34
tags:
- Java
- Log
categories: 
- 翻译
---

# 为什么你无法利用生产系统的日志找到错误的根本原因?
要问你是否用日志文件去监视你的系统就好像在问你是不是喝水一样。我们都用日志，但是我们怎么用的日志其实很难回答。
在下面的文章中，我们将深入到日志文件中看看他们是怎么写的,都写了些什么。
咱们开始吧。
> 特别感谢（Big shout out to）研发团队（R&D team）的Aviv Danziger的大力帮助，帮我们对数据进行分析处理。   

# 基本原理(Groundwork)
我们的研究需要大量的数据，因此我们使用了Google BigQuery，几个月之前，我们第一次用它来统计Github上排名前列的Java项目是怎么使用日志的。
截止到这篇文章发表，我们统计了Github上排名前400,000的Java仓库（根据2016年项目获取到的星数排名）。我们从中排除了Android、样例程序和简单的测试程序。留下了17,797个仓库。

然后，我们提取了包含100条以上日志语句的仓库，最后剩下了1,463个仓库。现在，是时候为让我们彻夜不眠的问题找到答案了。

# [TL;DR(Too long;Don't Read)](https://www.zhihu.com/question/20187408): 主要结论
如果你没心思去看统计图，想跳过本文直奔主题，这里给出我们总结的重要的五点：

1. 日志没有我们想象的那样包含很多信息，即使日志可能每天以上百GB的速度增长。50%以上的日志语句不包含任何关于程序变量状态的信息。
2. 在生成系统中，64%的日志语句都是不活动的。
3. The logging statements that do reach production have 35% less variables than the average development level logging statement。
4. 经常出现“This should never happen” 。
5. 应该有更好的办法来解决生成系统中的错误。


现在我们用数据来说明每一点。

# 1. 实际上有多少日志语句包含变量?
首先我们要检查的是每条语句中有多少变量。在每个仓库中,我们选择将语句划分成从0个变量到5个及以上的几组。然后获取到总数，并且获取到在所有研究的仓库中的平均数值。

![Average Java Project by Number of Variables](https://www.javacodegeeks.com/wp-content/uploads/2017/02/totalrepo-300x253@2x.png)
Java项目日志中平均包含变量个数


看到了吗, 日志中包含变量平均个数为0的占到了50%以上。 另外可见仅有0.95%的项目的日志语句中包含5个及以上变量。
这意味着日志文件捕捉到的系统信息其实是非常有限的。要从日志文件中分析出到底发生了什么犹如大海捞针一样困难。


# 2. 在生产系统中有多少日志语句是活动的?
多种原因会造成开发环境和生产环境的差异，不同日志的方式其中之一。在开发环境中，所有级别都是活动的。但是，在生成系统中仅有ERROR和WARN的是活动的。
看起来就想这样：
![](https://www.javacodegeeks.com/wp-content/uploads/2017/02/prodvslog-300x190@2x.png)
开发环境和生产环境日志的差异

统计显示，Java应用中有35.5%的日志语句在生产系统中有可能被执行（ERROR, WARN），另外的65.5%的语句仅在开发环境执行（TRACE, INFO, DEBUG）。
哎呀，大部分信息都丢失了！ 

# 3. 一条日志中平均包含多少个变量?
所以，不仅开发人员在编程语句中省略了变量，平均上看Java系统在日志系统中根本没有多少语句能执行。
现在，我们决定来看看每个级别的日志平均包含多少个变量信息。

![Average Number of Variables per Logging Statement](https://www.javacodegeeks.com/wp-content/uploads/2017/02/logging_level-300x273@2x.png)
每条日志语句平均包含变量数量



从平均值上看到，TRACE、DEBUG和INFO的日志比WARN和ERROR的包含更多的变量信息。“更多”时相对的，前三种日志包含的变量数量平均为0.78个，后两个的平均值为0.5（作者的意思是都很低）。
这意味着生产系统的日志语句包含的变量数量比开发系统要少35%。另外，前面提过，他们全部的数量也很少。

如果你在从日志中分析系统出了什么故障，但一无所获。不用担心，其实还有别的好办法。

用[OverOps](https://en.wikipedia.org/wiki/OverOps) 可以帮你在不依赖日志文件的内容下，看到任何异常、错误日志或和警告的变量信息。你可以通过事件调用栈看到完整的源码和变量状态。即使并没有打印任何信息到日志文件。OverOps还能展示出错误之前的250条DEBUG、TRACE 和INFO级别的语句。 
[点击这里查看Demo](https://youtu.be/xb8eP08b2iQ)


# 4. This Should Never Happen
我们已经了解了这些日志语句的所有信息, 来开个玩笑. 我们发现了58处 “This should never happen”。
我们想说的是，如果那真的应该不会发生，那你至少应该体面的打印一两个变量出来，这样你才能知道它到底是怎么发生的。


# 我们怎么实现的这些统计?
如上文提到，为了得到这些数据，我们第一步是过滤到没用的Java项目，关注那些超过100条日志语句的项目。最后得到1463个项目。
然后通过正则表达式取出所有日志行。

```bash
SELECT *
FROM [java-log-levels-usage:java_log_level_usage.top_repos_java_contents_lines_no_android_no_arduino]
WHERE REGEXP_MATCH(line, r'.*((LOGGER|Logger|logger|LOG|Log|log)[.](trace|info|debug|warn|warning|error|fatal|severe|config|fine|finer|finest)).*')
OR REGEXP_MATCH(line, r'.*((Level|Priority)[.](TRACE|TRACE_INT|X_TRACE_INT|INFO|INFO_INT|DEBUG|DEBUG_INT|WARN|WARN_INT|WARNING|WARNING_INT|ERROR|ERROR_INT)).*')
OR REGEXP_MATCH(line, r'.*((Level|Priority)[.](FATAL|FATAL_INT|SEVERE|SEVERE_INT|CONFIG|CONFIG_INT|FINE|FINE_INT|FINER|FINER_INT|FINEST|FINEST_INT|ALL|OFF)).*')

```
现在我们拿到了数据，现在开始进行切分。首先，过滤出每一个日志级别包含的变量数量。
```bash
SELECT sample_repo_name
      ,log_level
      ,CASE WHEN parametersCount + concatenationCount = 0  THEN "0"
            WHEN parametersCount + concatenationCount = 1  THEN "1"
            WHEN parametersCount + concatenationCount = 2  THEN "2"
            WHEN parametersCount + concatenationCount = 3  THEN "3"
            WHEN parametersCount + concatenationCount = 4  THEN "4"
            WHEN parametersCount + concatenationCount >= 5 THEN "5+"
        END total_params_tier
      ,SUM(parametersCount + concatenationCount) total_params
      ,SUM(CASE WHEN parametersCount > 0 THEN 1 ELSE 0 END) has_parameters
      ,SUM(CASE WHEN concatenationCount > 0 THEN 1 ELSE 0 END) has_concatenation
      ,SUM(CASE WHEN parametersCount = 0 AND concatenationCount = 0 THEN 1 ELSE 0 END) has_none
      ,SUM(CASE WHEN parametersCount > 0 AND concatenationCount > 0 THEN 1 ELSE 0 END) has_both
      ,COUNT(1) logging_statements
      ,SUM(parametersCount) parameters_count
      ,SUM(concatenationCount) concatenation_count
      ,SUM(CASE WHEN isComment = true THEN 1 ELSE 0 END) comment_count
      ,SUM(CASE WHEN shouldNeverHappen = true THEN 1 ELSE 0 END) should_never_happen_count
  FROM [java-log-levels-usage:java_log_level_usage.top_repos_java_log_lines_no_android_no_arduino_attributes]  
 GROUP BY sample_repo_name
         ,log_level
         ,total_params_tier
```
然后计算每个级别的平均值。如下显示了计算所有仓库语句平均半分比的方法：
```bash
SELECT total_params_tier
      ,AVG(logging_statements / total_repo_logging_statements) percent_out_of_total_repo_statements
      ,SUM(total_params) total_params
      ,SUM(logging_statements) logging_statements
      ,SUM(has_parameters) has_parameters
      ,SUM(has_concatenation) has_concatenation
      ,SUM(has_none) has_none
      ,SUM(has_both) has_both
      ,SUM(parameters_count) parameters_count
      ,SUM(concatenation_count) concatenation_count
      ,SUM(comment_count) comment_count
      ,SUM(should_never_happen_count) should_never_happen_count
  FROM (SELECT sample_repo_name
              ,total_params_tier
              ,SUM(total_params) total_params
              ,SUM(logging_statements) logging_statements
              ,SUM(logging_statements) OVER (PARTITION BY sample_repo_name) total_repo_logging_statements
              ,SUM(has_parameters) has_parameters
              ,SUM(has_concatenation) has_concatenation
              ,SUM(has_none) has_none
              ,SUM(has_both) has_both
              ,SUM(parameters_count) parameters_count
              ,SUM(concatenation_count) concatenation_count
              ,SUM(comment_count) comment_count
              ,SUM(should_never_happen_count) should_never_happen_count
          FROM [java-log-levels-usage:java_log_level_usage.top_repos_java_log_lines_no_android_no_arduino_attributes_counters_with_params_count]
         GROUP BY sample_repo_name
                 ,total_params_tier)
 WHERE total_repo_logging_statements >= 100
 GROUP BY total_params_tier
 ORDER BY 1,2
```

你可以从我们的[原始数据文件](http://384uqqh5pka2ma24ild282mv.wpengine.netdna-cdn.com/wp-content/uploads/2017/02/Logging-Scrapes-from-GitHub.xlsx)看看计算过程.

# 反思
我们都使用日志文件，但好像大多数都认为这是理所当然（take them for granted）的。大量的日志管理工具让我们忘了去优化自己的代码，让代码可以易读，方便我们理解、调试和修复。


> 原文：[https://www.javacodegeeks.com/2017/02/github-research-50-java-logging-statements-written-wrong.html](https://www.javacodegeeks.com/2017/02/github-research-50-java-logging-statements-written-wrong.html)