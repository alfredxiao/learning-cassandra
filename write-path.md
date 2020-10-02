Write Path
* Writes can go to any node (called coordinator, it can be different per request)

* Written to
  1. Commit Log
  2. Memtable
  3. From memtable to sstable (on disk) periodically

Acknowledgement to Coordinator
* A Write is considered successful once data is written to commitlog and memtable

Relationship with Tables
* Memtables and SSTables are maintained per table. The commit log is shared among tables.

Commit Log
* Commit Log is for not losing data should power outage occur
* To reduce the commit log replay time, DataStax recommends flushing the memtable before you restart the nodes.
* If a node stops working, replaying the commit log restores the writes to the memtable that were there before the node stopped.
* append only on commit log
* The database uses the commit log to rebuild memtables.
* The commit log is divided into segments.
* The database purges commit log segments only after all the data in a segment has been flushed to disk from the memtable.

Memtable
* Order
  * data sorted on memtable
* Write-Back Cache
  * Meaning batch sync later time, whereas a write-through cache sync each writes right away
* Upon a configured limit is reach, data is flushed to SStable

Flushing (from Memtable to SStable)
* Just flush data within this Memtable to a new file, not merged with existing SStable files?

SSTable
* Immutable, not written to again after the memtable is flushed
  * Consequently, a partition is typically stored across multiple SSTable files.

Delete
* Deletes are treated as tagging with 'tombstone'
* Rows marked as tombstone are not purged straight away in next compaction run(s)
  * There is a grace period (default to 10 days?), only expired tombstone rows are really purged
  * Reason not to purge them in next available compaction is
    * This node has responsibility to propagate data to replicas, if the tombstone rows are purged, how does other nodes know those rows are deleted?
* TTL can generate tombstone rows too

Update
* NOTE: Cassandra does NOT do update or delete in place, it always APPEND/INSERT a newer version of the row with a timestamp, which can later be used for compaction
* Overtime, the same row identified by a particular primary key has more than one versions in one or multiple sstables, compaction would workout how to merge them into one sstable and throw away unneeded sstables.
  * Some database operations may only write partial updates of a row, so some versions of a row may include some columns, but not all. During a compaction or write, the database assembles a complete version of each row from the partial updates, using the most recent version of each column.

Disk
* Recommended to use separate disks for commit logs and SSTables
