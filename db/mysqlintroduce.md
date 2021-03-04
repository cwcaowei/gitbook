#### Buffer Pool

InnoDB设计了一个内存的缓冲区，用来提高读写效率，缓冲区内容和磁盘不一致的时候称之为脏页，有专门的线程每隔一段时间就一次性的把Buffer Pool中的修改写入磁盘，称之为刷脏。

#### redo log

内存的修改操作的持久化文件，记录的是做了什么修改，而不是修改后的数据，是物理日志，大小固定，覆盖机制。只有InnoDB有，是顺序IO，效率更高，而刷脏是随机IO。redo log保证了事务的持久性。

#### undo log

分为insert undo log和update undo log，记录了事务发生之前的数据状态，便于回滚，它记录的是反向的操作，比如insert记为delete，update记为update原来的值，所以它是逻辑日志。undo log保证了事务的原子性。与redo log统称为事务日志。

#### bin log

以事件的形式记录了所有的DDL和DML语句，是逻辑日志，大小不固定，无限追加，用来做数据恢复或者主从复制。

#### 主从复制

- master将数据改变记录到bin log中，slave将master的bin log拷贝到它的中继日志relay log，

- master在每个事务更新数据完成之前将事务串行的写入bin log，即使事务中的语句都是交叉执行的。写入完成后，master通知存储引擎提交事务。

- slave开始一个工作线程——I/O线程。I/O线程在master上打开一个普通的连接，然后开始binlog dump process。Binlog dump process从master的bin log中读取事件，如果已经跟上master，它会睡眠并等待master产生新的事件。I/O线程将这些事件写入中继日志relay log。

- slave的SQL线程从relay log读取事件，并重放其中的事件而更新slave的数据，使其与master中的数据一致。只要该线程与I/O线程保持一致，relay log通常会位于OS的缓存中，所以relay log的开销很小。

![](/assets/db/copy.png)
