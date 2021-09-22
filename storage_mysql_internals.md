# Storage\_MySQL\_Internals

* [MySQL](storage_mysql_internals.md#mysql)
  * [Internals](storage_mysql_internals.md#internals)
    * [Overview](storage_mysql_internals.md#overview)
    * [Pluggable engines](storage_mysql_internals.md#pluggable-engines)
      * [Selection criteria](storage_mysql_internals.md#selection-criteria)
      * [Redo logs](storage_mysql_internals.md#redo-logs)
      * [Undo logs](storage_mysql_internals.md#undo-logs)
    * [Server layer](storage_mysql_internals.md#server-layer)
      * [\[TODO:::\] Binlog](storage_mysql_internals.md#todo-binlog)
      * [Slow query log](storage_mysql_internals.md#slow-query-log)
      * [General purpose log](storage_mysql_internals.md#general-purpose-log)
      * [Relay log](storage_mysql_internals.md#relay-log)
  * [InnoDB engine](storage_mysql_internals.md#innodb-engine)
    * [Components](storage_mysql_internals.md#components)
    * [Index](storage_mysql_internals.md#index)
      * [Types](storage_mysql_internals.md#types)
        * [Clustered vs unclustered index](storage_mysql_internals.md#clustered-vs-unclustered-index)
        * [Primary vs secondary index \(same as above\)](storage_mysql_internals.md#primary-vs-secondary-index-same-as-above)
        * [B+ tree vs hash index](storage_mysql_internals.md#b-tree-vs-hash-index)
      * [Adaptive hash index](storage_mysql_internals.md#adaptive-hash-index)
  * [Transaction model](storage_mysql_internals.md#transaction-model)
    * [ACID and InnoDB](storage_mysql_internals.md#acid-and-innodb)
    * [Three problems](storage_mysql_internals.md#three-problems)
      * [Dirty read](storage_mysql_internals.md#dirty-read)
      * [Non-repeatable read](storage_mysql_internals.md#non-repeatable-read)
      * [Phantam read](storage_mysql_internals.md#phantam-read)
    * [Four isolation solution options](storage_mysql_internals.md#four-isolation-solution-options)
      * [Read uncommitted](storage_mysql_internals.md#read-uncommitted)
      * [Read committed](storage_mysql_internals.md#read-committed)
      * [Repeatable read](storage_mysql_internals.md#repeatable-read)
      * [Serializable](storage_mysql_internals.md#serializable)
    * [MVCC \(multi-version concurrency control\)](storage_mysql_internals.md#mvcc-multi-version-concurrency-control)
      * [Motivation](storage_mysql_internals.md#motivation)
      * [InnoDB MVCC Interals](storage_mysql_internals.md#innodb-mvcc-interals)
        * [Example](storage_mysql_internals.md#example)
    * [Lock](storage_mysql_internals.md#lock)
      * [Shared vs exclusive locks](storage_mysql_internals.md#shared-vs-exclusive-locks)
        * [Shared lock](storage_mysql_internals.md#shared-lock)
        * [Exclusive lock](storage_mysql_internals.md#exclusive-lock)
        * [Intentional shared/exclusive lock](storage_mysql_internals.md#intentional-sharedexclusive-lock)
      * [Row vs table locks](storage_mysql_internals.md#row-vs-table-locks)
        * [Table locks](storage_mysql_internals.md#table-locks)
          * [Add/Release Table lock:](storage_mysql_internals.md#addrelease-table-lock)
          * [AUTO\_INC lock](storage_mysql_internals.md#auto_inc-lock)
        * [Some Row locks \(exclusive lock\)](storage_mysql_internals.md#some-row-locks-exclusive-lock)
          * [Record lock](storage_mysql_internals.md#record-lock)
          * [Gap lock](storage_mysql_internals.md#gap-lock)
          * [Next-key lock](storage_mysql_internals.md#next-key-lock)
  * [TODO](storage_mysql_internals.md#todo)

## MySQL

### Internals

#### Overview

#### Pluggable engines

* Theoretically, different tables could be configured with different engines. 
* There are a list of innoDB engines such as Innodb
  * InnoDB: support transaction, support row level lock 
  * MyISAM: not support transaction, only table level lock
  * Archive
  * Memory
  * CSV
  * Federated
  * TokuDB: 
* InnoDB vs MyISAM: 

**Selection criteria**

* Need to support transaction? 
* Need to support hot online backup?
  * mysqldump
  * Innodb is the only engine supports online backup
* Need to support crush recovery?

**Redo logs**

**Undo logs**

#### Server layer

**\[TODO:::\] Binlog**

* Reference: [https://coding.imooc.com/lesson/49.html\#mid=486](https://coding.imooc.com/lesson/49.html#mid=486)

**Slow query log**

**General purpose log**

**Relay log**

### InnoDB engine

#### Components

![](.gitbook/assets/mysql_internal_innodb_arch.png)

#### Index

* Pros:
  * Change random to sequential IO
  * Reduce the amount of data to scan
  * Sort data to avoid using temporary table
* Cons: 
  * Slow down writing speed
  * Increase query optimizer process time

**Types**

**Clustered vs unclustered index**

* Def: If within the index, the leaf node stores the entire data record, then it is a clustered index. "Clustered" literrally means whether the index and data are clustered together. 
* Limitations: 
  * Since a clustered index impacts the physical organization of the data, there can be only one clustered index per table.
  * Since an unclustered index only points to data location, at least two operations need to be performed for accessing data. 
* For example, 
  * mySQL innoDB primary key index is a clustered index \(both index and data inside \*.idb file\)
  * mySQL myISAM index is an unclustered index \(index inside _.myi file and data inside \_.myd file\). 
  * Oracle uses unclustered index
* Comparison: 
  * For read queries: Clustered index will typically perform a bit faster because only needs to read disk once \(data and index stored together\)
  * For write updates/deletes: Unclustered index will typically perform a bit faster because for unclustered index approach, the data part could be written in an append-only fashion and index part could be inserted. 

![](.gitbook/assets/mysql_internal_clusteredIndex.png)

![](.gitbook/assets/mysql_internal_unclusteredindex.png)

**Primary vs secondary index \(same as above\)**

* Def: Primary index points to data and secondary index points to primary key. Primary index will be a clustered index and secondary index will be an unclustered index.  
* Why secondary index only points to primary key: 
  * Save space: Avoid storing copies of data. 
  * Keep data consistency: When there are updates on primary key, all other secondary indexes need to be updated at the same time. 

![Index B tree secondary index](.gitbook/assets/mysql_index_secondaryIndex.png)

**B+ tree vs hash index**

* B+ tree index
  * Use case: Used in 99.99% case because it supports different types of queries
* Hash index
  * Use case: Only applicable for == and IN type of query, does not support range query

**Adaptive hash index**

![Index B tree secondary index](.gitbook/assets/mysql_index_adaptiveHashIndex.png)

### Transaction model

* There are three problems related to 

![InnoDB read](.gitbook/assets/mysql_innodb_isolationlevel.png)

#### ACID and InnoDB

* InnoDB implements ACID by using undo, redo log and locks
  * Atomic: Undo log is used to record the state before transaction. 
  * Consistency: Redo log is used to record the state after transaction.
  * Isolation: Locks are used for resource isolation. 
  * Durability: Redo log and undo log combined to realize this. 

#### Three problems

**Dirty read**

* Def: SQL-transaction T1 modifies a row. SQL-transaction T2 then reads that row before T1 performs a COMMIT. If T1 then performs a ROLLBACK, T2 will have read a row that was never committed and that may thus be considered to have never existed.

![Dirty read](.gitbook/assets/databasetransaction_dirtyread.png)

**Non-repeatable read**

* Def: P2 \("Non-repeatable read"\): SQL-transaction T1 reads a row. SQL-transaction T2 then modifies or deletes that row and performs a COMMIT. If T1 then attempts to reread the row, it may receive the modified value or discover that the row has been deleted. It only applies to UPDATE / DELETE operation. 

![Non-repeatable read](.gitbook/assets/databasetransaction_nonrepeatableread.png)

**Phantam read**

* Def: SQL-transaction T1 reads the set of rows N that satisfy some . SQL-transaction T2 then executes SQL-statements that generate one or more rows that satisfy the  used by SQL-transaction T1. If SQL-transaction T1 then repeats the initial read with the same , it obtains a different collection of rows.

![Phantam read](.gitbook/assets/databasetransaction_phantamread.png)

#### Four isolation solution options

**Read uncommitted**

* Def: Not solving any concurrent transaction problems.

**Read committed**

* Def: When a transaction starts, could only see the modifications by the transaction itself. 

**Repeatable read**

* Def: Within a transaction, it always read the same data. 

**Serializable**

* Def: Everything is conducted in an exlusive way with lock. 

#### MVCC \(multi-version concurrency control\)

**Motivation**

* A traditional approach to resolve concurrency control problem is using locks. MVCC eliminates locking so that the processes can run concurrently without blocking each other.
* MVCC has different implementation for different database such as MySQL and PostgreSQL. This section focuses on MySQL's MVCC implemenation. 
* MySQL implements MVCC mechanism for both Read Committed and Repeatable Read isolation level. By default, MySQL uses Repeatable read. 
* MySQL MVCC consists of two components: Data versioning and read view. 

**InnoDB MVCC Interals**

* Undo log uses transaction id for data versioning
* Read view consists of:
  * An array of all uncommitted transaction ids. 
  * Already created max transaction id. 

**Example**

* Repeatable read
  * Read view will only be executed once in a transaction when the first statement executes. This is why \#select 2 reads a different value when compared with \#select 1. 
  * MySQL will go through the undo log from the latest to the older ones, and use the first log record bigger than its read view as true value. 

![](.gitbook/assets/mysql_innodb_mvcc_example.png)

![](.gitbook/assets/mysql_innodb_mvcc_undologchain.png)

* Read committed
  * Read view will be generated each time when a statement is executed. 
  * The rest will stay same as repeatable read. 

#### Lock

**Shared vs exclusive locks**

**Shared lock**

* Def: If transaction T1 holds a shared \(S\) lock on row r, then requests from some distinct transaction T2 for a lock on row r are handled as follows:
  * A request by T2 for an S lock can be granted immediately. As a result, both T1 and T2 hold an S lock on r.
  * A request by T2 for an X lock cannot be granted immediately.
* Add lock:   
  1. select ...from XXX where YYY lock in share mode
  2. insert ... into select ... 
* Release lock:  commit / rollback

**Exclusive lock**

* Def: If a transaction T1 holds an exclusive \(X\) lock on row r, a request from some distinct transaction T2 for a lock of either type on r cannot be granted immediately. Instead, transaction T2 has to wait for transaction T1 to release its lock on row r.
* Add lock: Automatically by default
  1. update
  2. delete
  3. insert
  4. select ... from XXX where YYY from update
     * If there is no index on YYY, then it will lock the entire table. 
* Release lock: commit / rollback

**Intentional shared/exclusive lock**

* Goal: Improve the efficiency of adding table wise lock. Divide the operation for adding lock into multiple phases. This is especially useful in cases of table locks. 
* Operation: Automatically added by database. If a shared lock needs to be acquired, then an intentional shared lock needs to be acquired first; If an exclusive lock needs to be acquired, then an intentional exclusive lock needs to be acquired first. 

**Row vs table locks**

* There are locks at different granularity and their conflicting status is documented below. 
* References: [https://www.javatpoint.com/dbms-multiple-granularity](https://www.javatpoint.com/dbms-multiple-granularity)

![](.gitbook/assets/dbms-multiple-granularity2.png)

**Table locks**

**Add/Release Table lock:**

* Add:
  1. Lock Table tableName READ
  2. Lock Table tableName WRITE
  3. discard table
  4. import  
* Release:
  * Commit / Rollback

**AUTO\_INC lock**

* Be triggered automatically when insert ... into Table xxx happens

**Some Row locks \(exclusive lock\)**

**Record lock**

* Prerequistes: Both needs to be met:
  * Where condition uses exact match \(==\) and the record exists. 
  * Where condition uses unique index. 

![Record lock](.gitbook/assets/mysql_lock_recordLock.png)

**Gap lock**

* Prerequistes: Both needs to be met:
  * Database isolation level is repeatable read. 
  * One of the following:
    * Where condition uses exact match \(==\) on a unique index and the record does not exist.
    * Where condition uses range match on a unique index.
    * Where condition doesn't have a unique index. \(table lock will be used\)
    * Where condition has index but is not unique index.

![Gap lock](.gitbook/assets/mysql_lock_gaplock.png)

**Next-key lock**

* Prerequistes:
  * If the where condition covers both gap lock and record lock, then next-key lock will be used. 
* Relationship with other locks:

![Interval keys](.gitbook/assets/mysql_index_interval.png)

* Next key = record lock + gap lock + record on the right border

![Next-key lock](.gitbook/assets/mysql_lock_nextkeylock.png)

### TODO

* [MySQL index deep dive](https://medium.com/free-code-camp/database-indexing-at-a-glance-bb50809d48bd)
* [MVCC](https://kousiknath.medium.com/how-mvcc-databases-work-internally-84a27a380283)
