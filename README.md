# Hadoop

### Q: What are the main subprojects of Hadoop?
A: The main subprojects of Hadoop include:

1. **Core Hadoop Components**:
   - **Core**: Set of components and interfaces for distributed filesystem and I/O
   - **HDFS**: Distributed filesystem that runs on large clusters
   - **MapReduce**: Distributed data processing model and execution environment
   - **YARN**: Resource management and job scheduling system

2. **Data Processing and Analysis**:
   - **Pig**: Data flow language and execution environment for exploring large datasets
   - **Hive**: Distributed data warehouse supporting SQL-like queries (HQL)
   - **Spark**: High-performance data processing engine (100x faster than traditional Hadoop)

3. **Storage and Database**:
   - **HBase**: Distributed, column-oriented database
   - **Avro**: Data serialization system for efficient RPC and persistent storage

4. **Coordination and Management**:
   - **ZooKeeper**: Distributed coordination service
   - **Oozie**: Workflow scheduling system

5. **Data Ingestion and Transfer**:
   - **Flume**: Service for collecting and moving large amounts of streaming data
   - **Chukwa**: Distributed data collection and analysis system
   - **Kafka**: Event streaming platform

6. **Search and Indexing**:
   - **Solr/Lucene**: Enterprise search and indexing capabilities

### Q: Why is Hadoop popular?
A: Hadoop is popular for several key reasons:

1. **Data Handling Capabilities**:
   - Processes massive amounts of varied data types
   - Handles both structured and unstructured data
   - Supports flexible storage decisions

2. **Cost-Effectiveness**:
   - Open-source framework
   - Runs on commodity hardware
   - Reduces storage and processing costs

3. **Performance and Scalability**:
   - Distributed processing capabilities
   - Linear scalability by adding nodes
   - High computing power for large datasets

4. **Data Protection**:
   - Built-in data replication
   - Fault tolerance
   - Protection against hardware failures

### Q: What is HDFS and how does it work?
A: HDFS (Hadoop Distributed File System) is a distributed file system designed for:

1. **Key Characteristics**:
   - Runs on commodity hardware
   - Highly fault-tolerant
   - Optimized for large datasets
   - Follows "write-once, read-many" pattern

2. **Architecture Components**:
   - **NameNode** (Master):
     - Manages filesystem namespace
     - Stores metadata
     - Controls block mapping
   - **DataNodes** (Slaves):
     - Store actual data blocks
     - Handle read/write requests
     - Manage block operations

3. **Block Management**:
   - Default block size: 64MB
   - Each block replicated multiple times (default: 3)
   - Replicas distributed across different nodes

### Q: How does HDFS handle file reading?
A: The file reading process follows these steps:

1) The client opens the file it needs by calling open() on the FileSystem object.
2) DFS calls the namenode using RPC, to determine the location of blocks for the first few blocks in the file.
-For each block the namenode returns the addresses of the datanodes that have a copy of that block
-If the client itself is a datanode then it will read from local DaataNode
3) The client then calls read() on stream.(DFSInputStream which has stored the datanode addresses for the first few blocks in the file, connects to the first datanode for the first block in the file.
4) Data is streamed from datanode back to client which calls read() repeatedly.
5) When the end of data block is reached ,DFSInputStream will close the connection to the datanode,then find the best datanode for the next block
-Blocks are read in order with DFSInputStream opening new connections to datanodes as the client reads through the stream.
6) When client has finished reading - calls close() on the FSDataInputStream


### How does Hadoop Cluster File Writing Happen?
1. **Client Creates the File**:
   - The client calls the `create()` method on the DistributedFileSystem (DFS) to create a new file.

2. **NameNode Namespace Operations**:
   - The DistributedFileSystem makes Remote Procedure Call (RPC) requests to the 
NameNode to create the new file in the Hadoop Distributed File System (HDFS) namespace.
   - The NameNode performs various checks:
     - It verifies that the file doesn't already exist in the namespace.
     - It checks that the client has the necessary permissions to create the file.
   - If all the checks pass, the NameNode makes a record of the new file in the namespace.
 If any check fails, the NameNode throws an IOException.

3. **FSDataOutputStream Creation**:
   - After the file is created in the namespace, the DistributedFileSystem returns an `FSDataOutputStream`
 to the client.
   - The `FSDataOutputStream` is a wrapper around a `DFSOutputStream`, which handles the
 communication with the DataNodes and NameNode.
 **Data Packet Handling**:
   - The `FSDataOutputStream` splits the data being written by the client into smaller packets.
   - These packets are written to an internal queue called the data queue.

 **Block Allocation by the NameNode**:
   - The Data Streamer is responsible for consuming the data queue and asking the NameNode
 to allocate new blocks for the file.
   - The NameNode selects a list of suitable DataNodes to store the replicas of the new blocks.
   - This list of DataNodes forms a pipeline for the data transfer.

4. **Data Transfer to DataNodes**:
   - The Data Streamer streams the data packets to the first DataNode in the pipeline.
   - The first DataNode stores the packet and forwards it to the second DataNode in the pipeline.
   - This process continues until the packet reaches the last DataNode in the pipeline.

5. **Acknowledgment Handling**:
   - The `DFSOutputStream` maintains another internal queue called the ack queue, which stores
 the packets waiting to be acknowledged by the DataNodes.
   - A packet is removed from the ack queue only when it has been successfully acknowledged by 
all the DataNodes in the pipeline.

6. **File Closure**:
   - When the client has finished writing data, it calls the `close()` method on the `FSDataOutputStream`.
   - This action flushes any remaining packets to the DataNode pipeline and waits for acknowledgments.
   - After all the packets have been acknowledged, the client contacts the NameNode to signal that the file
 is complete.

7. **NameNode Finalization**:
   - The NameNode already knows which blocks the file is made of, as it was involved in the block 
allocation process.
   - The NameNode only waits for the blocks to be minimally replicated (based on the configured
 replication factor) before returning successfully to the client.


### Q: How does HDFS handle failures during writing?
A: HDFS handles writing failures through:

1. **Pipeline Recovery**:
   - Failed DataNode removed from pipeline
   - Unacknowledged packets requeued
   - Pipeline reconstructed with remaining nodes

2. **Block Management**:
   - New block identity assigned
   - Partial blocks on failed node deleted
   - Additional replicas created as needed

### Q: What is HDFS's replica placement strategy?
A: HDFS uses a sophisticated replica placement strategy:

1. **First Replica**:
   - Placed on client's node (if client is on cluster)
   - Random node chosen for external clients

2. **Additional Replicas**:
   - Second replica on different rack
   - Third replica on same rack as second
   - Further replicas distributed randomly

3. **Considerations**:
   - Network topology awareness
   - Rack diversity for redundancy
   - Balance between reliability and bandwidth

### Q: What is Hive and its components?
A: Hive is a data warehouse system for Hadoop:

1. **Key Components**:
   - Command-line interface
   - JDBC/ODBC connectivity

2. **Features**:
   - Highly scalable
   - Supports batch and real-time processing
   - Compatible with various data stores
   - SQL-like query language (HQL)

### Q: What is Apache Kafka?
A: Kafka is an event streaming platform that provides:

1. End-to-end event streaming
2. Publish-subscribe capability for event streams
3. Durable storage of event streams
4. Real-time event processing

### Q: What is YARN's role in Hadoop?
A: YARN (Yet Another Resource Negotiator) serves as:

1. **Resource Management**:
   - Cluster resource allocation
   - Task scheduling
   - Processing activity management

2. **Components**:
   - Resource Manager (master)
   - Node Managers (on each DataNode)
   - Application Master (per-application)
  
## HDFS Deep Dive

### Q: What is Apache Flume and what are its capabilities?
A: Apache Flume is a distributed data ingestion service that:

1. **Core Functionality**:
   - Ingests unstructured and semi-structured data into HDFS
   - Collects, aggregates, and moves large datasets
   - Handles real-time streaming data

2. **Data Sources**:
   - Network traffic
   - Social media feeds
   - Email messages
   - Log files
   - Various streaming sources

### Q: What are the key characteristics of HDFS?
A: HDFS (Hadoop Distributed File System) has several distinctive features:

1. **Core Design Principles**:
   - Runs on commodity hardware
   - Highly fault-tolerant architecture
   - Optimized for large datasets
   - Supports streaming data access
   - Relaxes POSIX requirements for better performance

2. **Data Management**:
   - "Write-once, read-many" access pattern
   - Block-based data distribution
   - Automatic data replication
   - High throughput access

3. **Reliability Features**:
   - Handles hardware failures gracefully
   - Maintains data integrity through replication
   - Continues operation during interruptions

### Q: How does HDFS organize and manage data?
A: HDFS uses a hierarchical block structure:

1. **Block Management**:
   - Default block size: 64MB
   - Each file divided into block-sized chunks
   - Blocks stored as independent units
   - Minimum read/write unit for disk operations

2. **Data Distribution**:
   - Blocks spread across multiple DataNodes
   - Each block replicated (default: 3 copies)
   - Replicas stored on different nodes
   - Ensures reliability and availability

3. **Metadata Management**:
   - NameNode tracks block locations
   - Maintains filesystem namespace
   - Stores all metadata information

### Q: What are the main components of HDFS architecture?
A: HDFS uses a master-slave architecture with these components:

1. **NameNode (Master)**:
   - Single master server per cluster
   - Manages filesystem namespace
   - Handles metadata operations
   - Controls client access
   - Tracks block locations
   - Functions:
     - Opening/closing files
     - Renaming files/directories
     - Block-to-DataNode mapping

2. **DataNodes (Slaves)**:
   - One per node in cluster
   - Stores actual data blocks
   - Handles read/write requests
   - Performs block operations:
     - Creation
     - Deletion
     - Replication
   - Regular heartbeat communication with NameNode

### Q: How does HDFS handle data reading operations?
A: The read operation follows a specific workflow:

1. **Initialization**:
   - Client calls `open()` on FileSystem object
   - DFS contacts NameNode via RPC
   - NameNode returns block locations

2. **Read Process**:
   - Client establishes connection with nearest DataNode
   - Reads data through DFSInputStream
   - Streams data block by block
   - Closes connections after each block

3. **Optimization**:
   - Reads from local DataNode when possible
   - Maintains multiple active streams
   - Sequential block reading

### Q: How does HDFS handle errors during read operations?
A: HDFS has built-in error handling mechanisms:

1. **Node Failures**:
   - Automatically tries next closest replica
   - Maintains failed node blacklist
   - Avoids retrying failed nodes

2. **Data Corruption**:
   - Verifies checksums for all data transfers
   - Reports corrupted blocks to NameNode
   - Attempts to read from alternative replicas
  
   
### How does Hadoop Cluster File Writing Happen?
1. **Client Creates the File**:
   - The client calls the `create()` method on the DistributedFileSystem (DFS) to create a new file.

2. **NameNode Namespace Operations**:
   - The DistributedFileSystem makes Remote Procedure Call (RPC) requests to the 
NameNode to create the new file in the Hadoop Distributed File System (HDFS) namespace.
   - The NameNode performs various checks:
     - It verifies that the file doesn't already exist in the namespace.
     - It checks that the client has the necessary permissions to create the file.
   - If all the checks pass, the NameNode makes a record of the new file in the namespace.
 If any check fails, the NameNode throws an IOException.

3. **FSDataOutputStream Creation**:
   - After the file is created in the namespace, the DistributedFileSystem returns an `FSDataOutputStream`
 to the client.
   - The `FSDataOutputStream` is a wrapper around a `DFSOutputStream`, which handles the
 communication with the DataNodes and NameNode.
 **Data Packet Handling**:
   - The `FSDataOutputStream` splits the data being written by the client into smaller packets.
   - These packets are written to an internal queue called the data queue.

 **Block Allocation by the NameNode**:
   - The Data Streamer is responsible for consuming the data queue and asking the NameNode
 to allocate new blocks for the file.
   - The NameNode selects a list of suitable DataNodes to store the replicas of the new blocks.
   - This list of DataNodes forms a pipeline for the data transfer.

4. **Data Transfer to DataNodes**:
   - The Data Streamer streams the data packets to the first DataNode in the pipeline.
   - The first DataNode stores the packet and forwards it to the second DataNode in the pipeline.
   - This process continues until the packet reaches the last DataNode in the pipeline.

5. **Acknowledgment Handling**:
   - The `DFSOutputStream` maintains another internal queue called the ack queue, which stores
 the packets waiting to be acknowledged by the DataNodes.
   - A packet is removed from the ack queue only when it has been successfully acknowledged by 
all the DataNodes in the pipeline.

6. **File Closure**:
   - When the client has finished writing data, it calls the `close()` method on the `FSDataOutputStream`.
   - This action flushes any remaining packets to the DataNode pipeline and waits for acknowledgments.
   - After all the packets have been acknowledged, the client contacts the NameNode to signal that the file
 is complete.

7. **NameNode Finalization**:
   - The NameNode already knows which blocks the file is made of, as it was involved in the block 
allocation process.
   - The NameNode only waits for the blocks to be minimally replicated (based on the configured
 replication factor) before returning successfully to the client.

# HDFS Fault Tolerance and Replica Management

### Q: What happens when a DataNode fails during a write operation?
A: HDFS has a sophisticated failure handling mechanism that involves several steps:

1. **Pipeline Management**:
   - Immediate closure of the write pipeline
   - Packets in acknowledgment queue moved to front of data queue
   - Prevents data loss for downstream DataNodes

2. **Block Management**:
   - New identity assigned to blocks on surviving DataNodes
   - NameNode updated with new block identities
   - Partial blocks on failed node marked for deletion upon recovery

3. **Pipeline Reconstruction**:
   - Failed DataNode removed from pipeline
   - Pipeline reconstructed with remaining healthy nodes
   - Write operation continues with adjusted pipeline

4. **Recovery Actions**:
   - Under-replication detected by NameNode
   - New replica created on different DataNode
   - Subsequent blocks written to new pipeline

## Replica Placement Strategy

### Q: How does HDFS determine where to place data replicas?
A: HDFS uses a sophisticated replica placement strategy that balances several factors:

1. **Key Considerations**:
   - Reliability requirements
   - Write bandwidth optimization
   - Read bandwidth optimization
   - Network topology awareness

2. **Placement Rules**:
   - **First Replica**:
     - Placed on client's node (if client is in cluster)
     - Random node selection for external clients
     - Avoids overloaded/full nodes

   - **Second Replica**:
     - Different rack from first replica
     - Random node selection within chosen rack

   - **Third Replica**:
     - Same rack as second replica
     - Different node from second replica
     - Random selection within rack

   - **Additional Replicas**:
     - Distributed randomly across cluster
     - Avoids rack concentration

3. **Trade-offs**:
   - Single-node placement: Fastest writes, no redundancy
   - Cross-datacenter placement: Maximum redundancy, higher bandwidth cost
   - Rack-aware placement: Balance of reliability and performance

### Q: How does HDFS handle network topology?
A: HDFS uses a network-aware architecture:

1. **Network Structure**:
   - Two-level topology implementation
   - 30-40 servers per rack
   - 1GB switch per rack
   - Uplink router connection

2. **Performance Optimization**:
   - Topology-aware data placement
   - Network location mapping for multi-rack setups
   - Rack-aware replica placement

3. **Operational Considerations**:
   - NameNode uses network topology for block placement
   - JobTracker utilizes location data for task scheduling
   - Optimizes data locality for MapReduce tasks

### Q: How is the write pipeline constructed?
A: The pipeline construction follows network topology awareness:

1. **Pipeline Structure** (for replication factor 3):
   ```
   Client → First Replica (same node) 
        → Second Replica (different rack) 
        → Third Replica (same rack as second)
   ```

2. **Optimization Factors**:
   - Minimizes network hops
   - Balances load across racks
   - Considers network bandwidth
   - Optimizes for fault tolerance

3. **Implementation Details**:
   - Pipeline built after replica locations chosen
   - Network topology taken into account
   - Ensures efficient data transfer
   - Maintains data consistency
