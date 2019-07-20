#drop、truncate、delete

drop直接删掉表；

truncate删除的是表中的数据，再插入数据时自增长的数据id又重新从1开始；

delete删除表中数据，可以在后面添加where字句。



delete:      (1):可以带条件删除 where   （2):只能删除表的数据，不能删除表的约束   （3）：使用delete from删除的数据可以回滚（事务）

truncate:  (1):不可以带条件删除              (2):即可以删除表的数据也可以删除表的约束  （3):使用truncate table删除的数据不能回滚 



