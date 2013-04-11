## Project Rhino

As Hadoop extends into new markets and sees new use cases with security and compliance challenges, the benefits of processing sensitive and legally protected data with all Hadoop projects and HBase must be coupled with protection for private information that limits performance impact. [Project Rhino]( https://github.com/intel-hadoop/project-rhino/) is our open source effort to enhance the existing data protection capabilities of the Hadoop ecosystem to address these challenges, and contribute the code back to Apache.

The core of the Apache Hadoop ecosystem as it is commonly understood is:

-  Core: A set of shared libraries
-	HDFS: The Hadoop filesystem
-	MapReduce: Parallel computation framework
-	ZooKeeper: Configuration management and coordination
-	HBase: Column-oriented database on HDFS
-	Hive: Data warehouse on HDFS with SQL-like access
-	Pig: Higher-level programming language for Hadoop computations
-	Oozie: Orchestration and workflow management
-	Mahout: A library of machine learning and data mining algorithms
-	Flume: Collection and import of log and event data
-	Sqoop: Imports data from relational databases

These components are all separate projects and therefore cross cutting concerns like authN, authZ, a consistent security policy framework, consistent authorization model and audit coverage loosely coordinated. Some security features expected by our customers, such as encryption, are simply missing. Our aim is to take a full stack view and work with the individual projects toward consistent concepts and capabilities, filling gaps as we go.

#### Our initial goals are:

##### 1) Framework support for encryption and key management

There is currently no framework support for encryption or key management. We will add this support into Hadoop Core and integrate it across the ecosystem. 

##### 2) A common authorization framework for the Hadoop ecosystem

Each component currently has its own authorization engine. We will abstract the common functions into a reusable authorization framework with a consistent interface. Where appropriate we will either modify an existing engine to work within this framework, or we will plug in a common default engine. Therefore we also must normalize how security policy is expressed and applied by each component. Core, HDFS, ZooKeeper, and HBase currently support simple access control lists (ACLs) composed of users and groups. We see this as a good starting point. Where necessary we will modify components so they each offer equivalent functionality, and build support into others.

##### 3) Token based authentication and single sign on

Core, HDFS, ZooKeeper, and HBase currently support Kerberos authentication at the RPC layer, via SASL. However this does not provide valuable attributes such as group membership, classification level, organizational identity, or support for user defined attributes. Hadoop components must interrogate external resources for discovering these attributes and at scale this is problematic. There is also no consistent delegation model. HDFS has a simple delegation capability, and only Oozie can take limited advantage of it. We will implement a common token based authentication framework to decouple internal user and service authentication from external mechanisms used to support it (like Kerberos). 

##### 4) Extend HBase support for ACLs to the cell level

Currently HBase supports setting access controls at the table or column family level. However, many use cases would benefit from the additional capability to do this on a per cell basis. In fact for many users dealing with sensitive information the ability to do this is crucial.

##### 5) Improve audit logging

Audit messages from various Hadoop components do not use a unified or even consistently formatted format. This makes analysis of logs for verifying compliance or taking corrective action difficult. We will build a common audit logging facility as part of the common authorization framework work. We will also build a set of common audit log processing tools for transforming them to different industry standard formats, for supporting compliance verification, and for triggering responses to policy violations.

#### Current JIRAs:

As part of this ongoing effort we are contributing our work to-date against the jiras listed below. As you may appreciate, the goals for Project Rhino covers a number of different Apache projects, the scope of work is significant and likely to only increase as we get additional community input. We also appreciate that there may be others in the Apache community that may be working on some of this or are interested in contributing to it. If so, we look forward to partnering with you in Apache to accelerate this effort so the Apache community can see the benefits from our collective efforts sooner.

Please feel free to reach out to us by commenting on the JIRAs below:

[HADOOP-9331: Hadoop crypto codec framework and crypto codec implementations](https://issues.apache.org/jira/browse/hadoop-9331) and related sub-tasks

We extend the compression codec to a crypto codec framework to support encryption and decryption files stored on HDFS. By defining these interfaces and facilities, we are targeting to establish a common abstraction on the API level that can be shared by all crypto codec implementations as well as users that use the API. It also provides a foundation for other components in Hadoop such as Map Reduce or HBase to support encryption features. Our framework borrows from and maintains the type hierarchy and API conventions of the existing Hadoop compression codec framework. Encryptors and decryptors share the interfaces of compressors and decompressors, respectively. However, because, unlike compression algorithms, crypto algorithms require some additional context specific state at runtime (key material, initialization vectors, etc.), we provide a few additional extra methods for management of this context.
 
We also provide a splittable AES codec implementation that supports encryption and decryption of data with Advanced Encryption Standard (AES), a specification for the encryption of electronic data used worldwide.  Further more, the performance of AES codec was optimized by using AES-NI instructions.  For AESCodec, we extend the SplittableCryptoCodec to support encryption. Our goal is adding a splittable crypto codec framework and a fast encryption algorithm through AES-NI, with minimal changes to core code. Compression can be done before encryption. The encrypted file is splittable and the blocks can be stored in different data nodes. For SimpleAESCodec, we extend the CryptoCodec to support encryption, compared to AESCodec, SimpleAESCodec doesn't support splittable and can't be used together with compression, and its encryption block header is much smaller.

[MAPREDUCE-5025: Key Distribution and Management for supporting crypto codec in Map Reduce](https://issues.apache.org/jira/browse/mapreduce-5025) and related JIRAs

The work here enables MapReduce to utilize the Crypto Codec framework to support encryption and decryption of data during MapReduce jobs. [HADOOP-9331: Hadoop crypto codec framework and crypto codec implementations](https://issues.apache.org/jira/browse/hadoop-9331) introduces crypto codec support in the framework level for pluggable crypto codec implementations and provides basic facilities of key context resolving.  While encryption and decryption of files in a MapReduce job usually have a lot of requirements, we collect and implement the common requirements according to the real use cases and discussions from the community on [MAPREDUCE-4491: Encryption and Key Protection](https://issues.apache.org/jira/browse/MAPREDUCE-4491) and other list discussions. These requirements include:
- Support that different stages (input, output, intermediate output) should have the flexibility to choose whether encrypt or not, as well as which crypto codec to use
-	Support that different stages may have different scheme of providing the keys
-	Support that different files (for example, different input files) may have or use different keys
-	Support a flexible way of retrieving keys for encryption or decryption

We also provide a DistCrypto tool for helping encryption, decryption and rotating multiple files by using a MapReduce job.

[HADOOP-9392: Token based authentication and Single Sign On] (https://issues.apache.org/jira/browse/HADOOP-9392)

We propose a new common facility for token based authentication and single sign on that is not tightly coupled to and dependent on Kerberos. We will extend the existing (Kerberos based) authentication framework with a delegation model of user authentication, a pluggable token provider interface, a pluggable token verification protocol and interface, and a new common secure mechanism for distributing secrets in cluster nodes capable of replacing the HDFS block delegation token facility.

[HADOOP-9466: Unified authorization framework](https://issues.apache.org/jira/browse/HADOOP-9466)

This JIRA proposes a unified common authorization framework, based on token-based authentication (HADOOP-9392), that can support unified authorization policy and configuration, provide a common pluggable authorization enforcement engine, and manage trust transferrence throughout the Hadoop ecosystem. Based on this framework we will implement a default service level authorization facility backwards compatible with the existing one, extend HDFS with file ACLs, and provide a full stack role-based access control policy.

[HBASE-6222: Add per-KeyValue Security](https://issues.apache.org/jira/browse/hbase-6222)
 
We extend the AccessController with mechanism to support authorization on a per cell basis. Our goal is to add per-cell ("per-KeyValue") security through as simple and straightforward extensions of the existing AccessController implementation as possible, with minimal changes to core code. The current proposal employs a "shadow" column family for storing ACL data. This column family is automatically added to new and existing tables and protected from all user actions. As before we maintain the AccessController check for a user’s access rights at the table and column family level, but now also we evaluate the union of table and column family level permissions with any permissions stored at the cell scope (if any) covered by the pending operation. Where possible we take advantage of these union-of-ACL semantics to avoid any additional I/O or performance impact.

[HBASE-7544: Transparent table/CF encryption](https://issues.apache.org/jira/browse/hbase-7544)

We introduce transparent encryption for HFiles and the WAL, work that builds on a crypto codec framework submitted to the Hadoop Common and MapReduce projects above. The goal is to protect sensitive data against accidental leakage and to facilitate auditable compliance. Under normal circumstances HBase files in HDFS are protected from other users and cluster services. However this does not guarantee leakage isn't possible if HDFS permissions are improperly applied or if a server is decommissioned from the cluster and mishandled.  Encryption plugs in almost exactly like compression codecs, and can be configured along with existing compression options, or alone. Our design facilities low impact incremental encryption and key management.

[ZOOKEEPER-1688: Transparent encryption of on-disk files](https://issues.apache.org/jira/browse/ZOOKEEPER-1688)

We introduce optional transparent encryption of snapshots and commit logs on disk. The goal is to protect against the leakage of sensitive information from files at rest, due to accidental misconfiguration of filesystem permissions, improper decommissioning, or improper disk disposal. This change would introduce a new ServerConfig option that allows the administrator to select the desired persistence implementation by classname, and new persistence classes extending the File* classes that wrap current formats in encrypted containers. Otherwise and by default the current File* classes will be used without change. If enabled, transparent encryption of all on disk structures will be accomplished with a shared cluster key made available to the quorum peers via the Java Keystore (supporting various store options, including hardware security module integration). A new utility for offline key rotation will also be provided.
