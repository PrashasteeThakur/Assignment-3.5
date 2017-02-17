# Assignment-3.5

Q1. High Availability of NameNode

In Hadoop 1.x, the NameNode was a single point of failure (SPOF) in an HDFS cluster. Each cluster had a single NameNode, and if that machine or process became unavailable, the cluster as a whole would be unavailable until the NameNode was either restarted or brought up on a separate machine.
This impacted the total availability of the HDFS cluster in two major ways:
•	In the case of an unplanned event such as a machine crash, the cluster would be unavailable until an operator restarted the NameNode.
•	Planned maintenance events such as software or hardware upgrades on the NameNode machine would result in windows of cluster downtime.
The HDFS High Availability feature addresses the above problems by providing the option of running two redundant NameNodes (Active and Standby namenodes in the same cluster. 

HDFS HA can be configured by two ways
•	Using Shared NFS Directory
•	Using Quorum Journal Manager

Using Shared NFS Directory
For the standby NameNode to keep its state synchronized with the active NameNode, this implementation requires that the two nodes both have access to a directory on a shared storage device (for example, an NFS mount from a NAS).
When any namespace modification is performed by the active NameNode, it durably logs a record of the modification to an edit log file stored in the shared directory. The standby NameNode constantly watches this directory for edits, and when edits occur, the standby NameNode applies them to its own namespace. In the event of a failover, the standby will ensure that it has read all of the edits from the shared storage before promoting itself to the active state. This ensures that the namespace state is fully synchronized before a failover occurs.

Using Quorum Journal Manager
For the standby NameNode to keep its state synchronized with the active NameNode in this implementation, both nodes communicate with a group of separate daemons called JournalNodes. When any namespace modification is performed by the active NameNode, it durably logs a record of the modification to a majority of the JournalNodes. The standby NameNode is capable of reading the edits from the JournalNodes, and is constantly watching them for changes to the edit log. As the standby Node sees the edits, it applies them to its own namespace. In the event of a failover, the standby ensures that it has read all of the edits from the JournalNodes before promoting itself to the active state. This ensures that the namespace state is fully synchronized before a failover occurs.
To provide a fast failover, it is also necessary that the standby NameNode has up-to-date information regarding the location of blocks in the cluster. To achieve this, DataNodes are configured with the location of both NameNodes, and they send block location information and heartbeats to both.


Q2.Checkpointing is an essential part of maintaining and persisting filesystem metadata in HDFS. It’s crucial for efficient NameNode recovery and restart, and is an important indicator of overall cluster health.
At a high level, the NameNode’s primary responsibility is storing the HDFS namespace. This means things like the directory tree, file permissions, and the mapping of files to block IDs. It’s important that this metadata (and all changes to it) are safely persisted to stable storage for fault tolerance.
This filesystem metadata is stored in two different constructs: the fsimage and the edit log. 
The fsimage is a file that represents a point-in-time snapshot of the filesystem’s metadata. However, while the fsimage file format is very efficient to read, it’s unsuitable for making small incremental updates like renaming a single file. Thus, rather than writing a new fsimage every time the namespace is modified, the NameNode instead records the modifying operation in the edit log for durability. This way, if the NameNode crashes, it can restore its state by first loading the fsimage then replaying all the operations (also called edits or transactions) in the edit log to catch up to the most recent state of the namesystem. The edit log comprises a series of files, called edit log segments, that together represent all the namesystem modifications made since the creation of the fsimage. 
Importance of checkpointing
A typical edit ranges from 10s to 100s of bytes, but over time enough edits can accumulate to become unwieldy. A couple of problems can arise from these large edit logs. In extreme cases, it can fill up all the available disk capacity on a node, but more subtly, a large edit log can substantially delay NameNode startup as the NameNode reapplies all the edits. This is where checkpointing comes in.
Checkpointing is a process that takes an fsimage and edit log and compacts them into a new fsimage. This way, instead of replaying a potentially unbounded edit log, the NameNode can load the final in-memory state directly from the fsimage. This is a far more efficient operation and reduces NameNode startup time.


Q3.
