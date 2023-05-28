### 1 redo log 和binlog有什么区别？

1. 首先binlog日志会记录所有与mysql数据库相关的日志记录，包括InnoDB、MyISAM、Heap等其他存储引擎的日志。而redo log只记录有关InnoDB本身的事务日志。
2. 其次记录的内容不同，无论用户将binlog文件记录的格式设为STATEMENT、ROW或者MIXED，其记录的都是关于一个事务的具体操作内容，即该日志是逻辑日志。而InnoDB的redo log记录的是关于每个页的更改的物理情况。
3. 写入的时间不同，binlog仅在事务事务提交前进行提交，即只写磁盘一次，不论该事务多大。而在事务进行的过程中，不断有重做日志条目被写入redo log。

### 2 Page Directory

Page directory中存放了记录的相对位置，这些记录指针也称为Slots(槽)或Directory Slots(目录槽)。由于在innoDB存储引擎中Page directory是稀疏目录，二叉查找的只是一个粗略的结果，因此InnoDB存储疫情必须通过recorder header中的next_record来继续查找相关记录。需要牢记的是，B+树索引本身并不能找到具体的一条记录，能找到的只是该记录所在的页。数据库把页载入到内存，然后通过Page directory再进行二叉查找。只不过二叉查找的时间复杂度很低，同时在内存中的查找很快，因此通常忽略这部分查找所用的时间。



