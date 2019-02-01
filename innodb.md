多版本并发控制（mvcc）获取高并发<br>
使用next-key locking策略避免幻读<br>

重做日志缓冲区（redo log buffer）<br>
1 master thread 每秒刷新一次<br>
2 每个事务提交时刷新一次<br>
3 剩余空间小于1/2时<br>

事务提交时先写重做日志，再修改缓冲页<br>

insert buffer<br>
对于非聚集索引的插入，先放到insert buffer中，再以一定的频率和辅助索引叶子节点合并<br>
使用insert buffer必须满足1.索引是辅助索引，2索引不唯一<br>

delete buffer，将记录标记为已删除<br>
purge buffer，真正将记录删除<br>

double write<br>
对缓冲池中的脏页进行刷新时：<br>
1. 将脏页复制到double write buffer中<br>
2. 将double write buffer写入磁盘<br>
2. 调用fsync同步脏页到磁盘中<br>
如果发生崩溃，先用磁盘中double write buffer中的数据恢复磁盘数据，然后应用重做日志<br>

二进制日志记录了所有表变更记录，重做日志文件记录innodb事务日志<br>
二进制日志记录具体操作内容，重做日志文件记录每个页的改动<br>

VARCHAR(N)N是字符的长度，所有varchar列长度总和最大65532字节<br>

innodb行记录格式<br>
compact：变长字段长度列表 + null标志位 + 记录头信息(是否删除(1b)+记录类型(3b)+记录数(4b)+下一条位置(16b)) + [rowid] + 事务id(Transaction ID) + 回滚指针(Roll Pointer) + 列1 + 列2 + 。。。<br>

对于多字节编码的char，innodb视其为变长字符类型<br>

innodb数据页格式<br>
File Header：checksum + 表空间页的偏移量 + 上一个页 + 下一个页 + 最后被修改的LSN + 页类型 + 所属表空间<br>
Page Header：页目录（Page Directory）中的槽数 + 堆中第一个记录的指针 + 堆中记录数 + 指向可用空间的首指针 + delete flag为1的记录数 + 最后插入记录的位置 + 最后插入记录的方向 + 一个方向连续插入记录的数量 + 该页记录数量 + 修改当前页的最大事务ID + 当前页在索引树中的位置 + 索引ID + 数据页非叶节点所在段 + 数据页所在段<br>
Page Directory: 目录槽，稀疏目录，可以利用二分查找迅速找到记录指针（粗略结果，需要通过遍历查找具体结果）<br>
File Trailer：完整写入检查，4字节checksum，4字节lsn，通过与Filer Header的checksum和lsn比较是否一致来保证页的完整性<br>

分区可以提高扫描性能，对于查询并没有性能提升<br>

innodb支持B+树索引、全文索引、哈希索引<br>

B+树索引，B代表balance，所有记录节点按键值的大小顺序存放在同一层的叶子节点上，B+树找到的是被查找数据行所在的页，然后把页数据读入内存，再在内存中查找
堆表：数据按照插入顺序存储，索引按照“文件号：页号：槽号”格式定位。（更新困难）<br>

全文检索通常使用倒排索引实现，它在辅助表中存储单词与其自身在一个或多个文档中所处位置之间的映射，有两种表现形式：<br>
1. inverted file index，{单词，单词所在文档id}<br>
2. full inverted index，{单词，（单词所在文档id，文档中位置）}<br>

一致性非锁定读：innodb通过行多版本控制的方法来读取当前执行时间数据库中行的数据，如果有更新操作，这个读不会等待排它锁释放，而是去读取行快照数据，快照数据通过undo段完成。<br>
快照数据是当前行数据之前的历史版本，有多个版本，称为行多版本技术（MVCC）<br>
一致性非锁定读，在事务隔离级别read committed下，使用最新的快照数据，在事务隔离级别repeatable read下，使用事务开始时的行数据版本。<br>

一致性锁定读：读加排他锁，for update，lock in share mode<br>

自增长使用特殊的表锁，锁在插入完成后释放，而不是事务结束时释放<br>

innodb有3种行锁算法<br>
1. record lock：单个行记录上的锁<br>
2. gap lock：间隙锁，锁定一个范围，不包含记录本身<br>
3. next-key lock：record lock + gap lock，锁定一个范围，且锁定记录本身，解决Phantom problem（幻象问题）<br>

read uncommitted：脏读，读到未提交的数据<br>
read commited：不可重复读，读到已提交的数据。（幻象问题，使用next key lock避免这个问题）<br>
丢失更新，一个事务的更新操作会被另一个事务的更新操作覆盖<br>

死锁处理<br>
1. 设置事务超时时间<br>
2. 使用wait-for graph（等待图）的方式进行死锁检测<br>

事务：<br>
隔离性通过锁实现<br>
原子性、一致性、持久性通过redo log和undo log实现<br>

事务提交时，先将事务日志写入到重做日志文件，可以设置fsync的周期<br>
事务在undo log segment分配页并写入undo log的过程同样要写入重做日志<br>

事务提交时，将undo log放入列表中，由purge线程删除，因为此时undo log可能由其他事务使用<br>

undo log格式：<br>
insert undo log，对应insert，只对事务本身可见，提交后可直接删除<br>
update undo log，对应delete和update，对需要提供mvcc机制<br>
