#mysql 数据库性能优化总结

#在工作中遇到了数据库执行时,相同逻辑的sql执行效率不同，因此本片通过简单实例介绍一下数据库性能优化的情况。

##(1)尽量不要在数据库中做运算

###所有sql语句中,尽量不要执行运算

###这里说的运算主要包括如四则运算符，如+，-等。

##(2)避免负向查询和%前缀模糊查询,否则也无法使用索引

###对于模糊查询相关业务场景,可以使用索引的形式通过solr等工具进行查询。
##(3)不在索引列做运算或者使用函数(废弃索引)

###当在where条件中对于索引字段使用运算或者使用函数.

###For example : select empno, ename, sal, sal * 12 from emp1 where sal * 12 > 20000;

###其中sal为索引，当使用这样的sql的时候,sal最终没有作为sql使用

###这样的sql可以优化为select empno, ename, sal, sal * 12 from emp1 where sal > 20000/12;

###这样的sql使用可以作为索引。

##(4)不要在生产环境程序中使用select * from 的形式查询数据。只查询需要使用的列。

###有人执行sql使用select * from这种形式，在对指定字段进行处理,这种方式不是很合理，考虑业务的发展，这个表的数据量和数据结构会越来越大，会影响的sql的执行时间。

##(5)数据查询时尽可能使用limit减少返回的行数，减少数据传输时间和带宽浪费。

###因为limit能够避免全表查找,同时减少sql存储mysql的缓存数据。数据量少能够减少网络带宽的消耗。

##(6)应尽量避免在 where 子句中对字段进行 null 值判断，否则将导致引擎放弃使用索引而进行全表扫描(废弃索引)

###For example 选出没有成绩的同学 select name from table where score == null; 这个sql使用是有其问题。若

###score设置索引 但是本sql却无法合理使用索引。

###可以推荐字段default值为-1 这样sql就变成了 select name from table where score == -1.

##(7)应尽量避免在 where 子句中使用 or 来连接条件，否则将导致引擎放弃使用索引而进行全表扫描(废弃索引)

##(8)in 和 not in 也要慎用，因为IN会使系统无法使用索引,而只能直接搜索表中的数据(废弃索引)

##(9)10.在使用索引字段作为条件时，如果该索引是复合索引，那么必须使用到该索引中的第一个字段作为条件时才能保证系统使用该索引，否则该索引将不会被使用，并且应尽可能的让字段顺序与索引顺序相一致

###for example 

###若select name from table where math > 90 and chinese >90 当索引为idx_math_chinese时生效 

###若select name from table where chinese >90 and math > 90时索引idx_math_chinese 不生效
