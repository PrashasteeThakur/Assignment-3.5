# Assignment-3.5

Q1. High Availability of NameNode

The NameNode was a single point of failure (SPOF) in a HDFS cluster in Hadoop 1.x..Each cluster had a single NameNode, and if that machine or process became unavailable, the cluster as a whole would be unavailable until the NameNode was either restarted or brought up on a separate machine.
This impacted the total availability of the HDFS cluster in two major ways:
•	In the case of an unplanned event such as a machine crash, the cluster would be unavailable until an operator restarted the NameNode.
•	Planned maintenance events such as software or hardware upgrades on the NameNode machine would result in windows of cluster downtime.
The HDFS High Availability feature addresses the above problems by providing the option of running two redundant NameNodes (Active and Standby namenodes in the same cluster. 

HDFS HA can be configured by two ways
•	Using Shared NFS Directory
•	Using Quorum Journal Manager

Using Shared NFS Directory
This implementation requires that the two nodes both have access to a directory on a shared storage device (for example, an NFS mount from a NAS) for the standby NameNode to keep its state synchronized with the active NameNode.
When any namespace modification is performed by the active NameNode, it durably logs a record of the modification to an edit log file stored in the shared directory. The standby NameNode constantly watches this directory for edits, and when edits occur, the standby NameNode applies them to its own namespace. In the event of a failover, the standby will ensure that it has read all of the edits from the shared storage before promoting itself to the active state. This ensures that the namespace state is fully synchronized before a failover occurs.

Using Quorum Journal Manager
In this implementation, both nodes communicate with a group of separate daemons called JournalNodes,for the standby NameNode to keep its state synchronized with the active NameNode . When any namespace modification is performed by the active NameNode, it durably logs a record of the modification to a majority of the JournalNodes. The standby NameNode is capable of reading the edits from the JournalNodes, and is constantly watching them for changes to the edit log. As the standby Node sees the edits, it applies them to its own namespace. In the event of a failover, the standby ensures that it has read all of the edits from the JournalNodes before promoting itself to the active state. This ensures that the namespace state is fully synchronized before a failover occurs.
To provide a fast failover, it is also necessary that the standby NameNode has up-to-date information regarding the location of blocks in the cluster. To achieve this, DataNodes are configured with the location of both NameNodes, and they send block location information and heartbeats to both.



Q2.Checkpointing is an essential part of maintaining and persisting filesystem metadata in HDFS.It is an important indicator of overall cluster functioning and is crucial for efficient NameNode recovery and restart.
The NameNode’s primary responsibility is storing the HDFS namespace. This means things like the directory tree, file permissions, and the mapping of files to block IDs. It’s important that this metadata (and all changes to it) are safely persisted to stable storage for fault tolerance.
This filesystem metadata is stored in two different constructs: the fsimage and the edit log. 
The fsimage is a file that represents a point-in-time snapshot of the filesystem’s metadata. However, while the fsimage file format is very efficient to read, it’s unsuitable for making small incremental updates like renaming a single file. Thus, rather than writing a new fsimage every time the namespace is modified, the NameNode instead records the modifying operation in the edit log for durability. If the NameNode crashes, it can restore its state by first loading the fsimage then replaying all the operations (also called edits or transactions) in the edit log to catch up to the most recent state of the namesystem. The edit log comprises a series of files, called edit log segments, that together represent all the namesystem modifications made since the creation of the fsimage. 
Importance of checkpointing
A typical edit ranges from 10s to 100s of bytes, but over time enough edits can accumulate to become unwieldy. A couple of problems can arise from these large edit logs. In extreme cases, it can fill up all the available disk capacity on a node, but more subtly, a large edit log can substantially delay NameNode startup as the NameNode reapplies all the edits. This is where checkpointing comes in.
Checkpointing is a process that takes an fsimage and edit log and compacts them into a new fsimage. This way, instead of replaying a potentially unbounded edit log, the NameNode can load the final in-memory state directly from the fsimage. This is a far more efficient operation and reduces NameNode startup time.



Q3.HDFS federation separates the namespace layer and storage layer. It enables the block storage layer. It also expands the architecture of an existing HDFS cluster to allow new implementations and use cases. The current HDFS architecture has two layers –
•	Namespace – This layer manages files, directories and blocks. This layer supports the basic file system operations e.g. listing of files, creation of files, modification of files and deletion of files and folders.
•	Block Storage – This layer has two parts –
•	Block Management This manages the datanodes in the cluster and provides operations like creation, deletion, modification and search.     It also takes care of the replication management.
•	Physical Storage This stores the blocks and provides access for read or write operations.

•	HDFS federation allows scaling the name service horizontally. It uses several namenodes or namespaces which are independent of each other. These independent namenodes are federated i.e. they don’t require inter coordination. These datanodes are used as common storage by all the namenodes. Each datanode is registered with all the namenodes in the cluster. These datanodes send periodic reports and responds to the commands from the name nodes. We have a block pool which is a set of blocks that belong to a single namespace. In a cluster, the datanodes stores blocks for all the block pools. Each block pool is managed independently. This enables the name space to generate block ids for new blocks without informing other namespaces. If one namenode fails for any reason, the datanode keeps on serving from other namenodes.
•	One namespace and its block are collectively called Namespace Volume. When a namespace or a namenode is deleted the corresponding block pool at the datanode is deleted automatically. In the process of cluster up-gradation, each namespace volume is upgraded as a unit.


4.Configuration files that are to be edited for sure while installing a hadoop cluster
The four files that need to be configured explicitly while setting up a single node hadoop cluster are:

Core-site.xml
Settings that need to be done in Core-site.xml
Some of the important properties are:

Configuring the name node address
Configuring the rack awareness factor
Selecting the type of security

HDFS-site.xml

The properties inside this xml file deals with storage procedure inside HDFS of Hadoop. Some of the important properties are:
Configure port access
Manages ssl client authentication
Controls Network interface
Changes file permission

YARN-site.xml
In Hadoop v1.x TaskTraker and JobTracker were present to handle the job of allocating resources to processes.
YARN has ResourceManager settings which effects resource allocation with node manager and application manager. Some of the important properties are:
WebAppProxy Configuration
MapReduce Configuration
NodeManager Configuration
ResourceManager Configuration
IPC Configuration

mapred-site.xml
When Hadoop runs for any analysis of dataset, the framework at runtime for MapReduce jobs is a vast set of rules for assigning jobs to slave and maintain the jobs records.In Hadoop2.x YARN is introduced to help this framework to work efficiently and take the workload for job related assignments. It is again a large unit of Hadoop ecosystem which helps running the map and reduce the collaboration with YARN.Some of the important features it handles are:

Node health script variables
Proxy Configuration
Job Notification Configuration
