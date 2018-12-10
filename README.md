# DeadLockTree
死锁技术研究


<pre>
死锁：
      死锁是指两个或两个以上的进程在执行过程中，因争夺资源而造成的互相等待的现象，若无外力
   作用，他们都将无法推进下去，此时称系统处于死锁状态，这些永远互相等待的进程称为死锁进程。
</pre>

<pre>
死锁产生的4个必要条件
      1）互斥条件
         指进程对所分配到的资源进行排他性使用，即在一段时间内某资源只有一个进程使用。如果此
      时还有其它进程使用，则请求者只能等待。
      
      2）请求和保持条件
         指进程已经保持至少一个资源，但又提出新的资源请求，而该资源已被其他进程占有，此时请
      求进程阻塞，但又对自己活得的其他资源保持不放。

      3）不剥夺条件
         指进程已获得的资源，在未使用完之前，不能被剥夺，只能在使用完时自己释放。
 
      5）环路等待
         指在发生死锁时，必然存在一个进程----资源的环形链，即进程集合{P0,P1,P2,...,Pn}
         中的P0正在等待一个P1占有的资源；P1正在等待P2占用的资源，...,Pn正在等待P0占用的
         资源。
</pre>

<pre>
在数据库中存在两种基本类型的锁：
      1）排它锁（Exclusive Locks，即X锁）
      2）共享锁(Share Locks，即S锁)

      当数据对象被加上排他锁时，其他的事务不能对它读取和修改。加了共享锁的数据对象可以被其他
      事务读取，但不能修改，数据库利用这两种基本的锁类型来对数据库的事务进行并发控制。

Mysql死锁技术研究

      1）用户A锁住了T1，然后又访问表T2,另一个表B锁住了表T2，同时企图访问表T1，这时用户A
        由于B已经锁住了表T2,用户A它必须等待用户B释放表T2的锁才能继续，同样用户B要等待用户A释放T1的锁才能继续，于是产生死锁。

         解决方法：
                 1）一般是代码bug引起，调整程序业务逻辑，预先规定一个加锁顺序，所有的事务
                    都必须按照这个顺序对数据执行加锁。       

      2）A查询一条记录，然后修改该记录，用户B修改该条记录，用户A事务里的性质由查询的共享锁
         编程修改的独占锁，而用户B的独占锁由于A有共享锁，所以必须等待A释放共享锁，而由于A
         的共享锁无法上升到独占锁所以不能释放共享锁，于是出现了死锁。

         解决方法：
                 1：使用乐观锁解决方案。乐观锁大多数是基于数据版本Version记录机制实现，
                    即为数据增加一个版本标识，在基于数据表的版本方案中，一般是通过位数据
                    库表增加"version"字段来实现。
                 2：谨慎使用悲观锁控制
                    如 select *** from update ***

      3）如果在事务中执行了一条不满足条件的update语句，则执行全表扫描，把行级锁上升为表级
         锁，多个这样的事务执行后，就很容易产生死锁和阻塞。类似的情况，还有当表中的数据量
         非常庞大而索引建的过少或不合适的情况，使得经常发生全表扫描，最终应用系统会越来越慢
         ，最终发生阻塞或死锁。

         解决方法：
                 1：每个事务执行的时间不允许太长;SQL语句中不要使用太复杂的关联多表查
                    询，使用执行计划对SQL语句进行分析，对于有全表扫描的SQL语句，建立相应的索引进行优化。
                 2：使用尽可能低的隔离级别
                 3: 数据空间离散法：将逻辑上在一个表中的数据分散到若干个离散的空间上去，
                    主要通过将大表进行行或列的分解为若干个小表，或者按照不同的用户群两种
                    方法实现。
                 5：每一个事务必须一次就将所使用到的数据全部加锁，否则不允许使用。
</pre>