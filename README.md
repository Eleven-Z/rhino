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

These core components, as well as many additional ones available in the Apache ecosystem, are all separate projects and therefore cross cutting concerns like authN, authZ, a consistent security policy framework, consistent authorization model and audit coverage loosely coordinated. Some security features expected by our customers, such as encryption, are simply missing. Our aim is to take a full stack view and work with the individual projects toward consistent concepts and capabilities, filling gaps as we go.

#### Our goals are:

##### 1) Framework support for encryption and key management

There is currently no framework support for encryption or key management. We will add this support into Hadoop Core and integrate it across the ecosystem. 

##### 2) A common authorization framework for the Hadoop ecosystem

Each component currently has its own authorization engine. We abstract the common functions into a reusable authorization framework with a consistent interface. Where appropriate we either modify an existing engine to work within this framework, or we plug in a common default engine. Therefore we also must normalize how security policy is expressed and applied by each component. Core, HDFS, ZooKeeper, and HBase currently support access control lists (ACLs) composed of users and groups. Hive Server supports RBAC permissions set on the database, table or view level.  We see this as a good starting point; these different approaches need to be integrated so that a single normalized permissions policy can be in effect for the vast majority of data access paths.

##### 3) Token based authentication and single sign on

Core, HDFS, ZooKeeper, and HBase currently support Kerberos authentication at the RPC layer, via SASL. However this does not provide valuable attributes such as group membership, classification level, organizational identity, or support for user defined attributes. Hadoop components must interrogate external resources for discovering these attributes and at scale this is problematic. There is also no consistent delegation model. HDFS has a simple delegation capability, and only Oozie can take limited advantage of it. We will implement a common token based authentication framework to decouple internal user and service authentication from external mechanisms used to support it (like Kerberos). 

##### 4) Extend HBase support for ACLs to the cell level

Currently HBase supports setting access controls at the table or column family level. However, many use cases would benefit from the additional capability to do this on a per cell basis. In fact for many users dealing with sensitive information the ability to do this is crucial.

##### UPDATE: This goal has been achieved with the availability of HBase 0.98.

##### 5) Improve audit logging

Audit messages from various Hadoop components do not use a unified or even consistently formatted format. This makes analysis of logs for verifying compliance or taking corrective action difficult. We will build a common audit logging facility. We will also build a set of common audit log processing tools for transforming them to different industry standard formats, for supporting compliance verification, and for triggering responses to policy violations.

#### Apache Projects:

##### sentry.incubator.apache.org: Apache Sentry (Incubating)

Apache Sentry (incubating) is a modular system for providing fine-grained role based authorization to both data and metadata stored on an Apache Hadoop cluster. It currently works with Apache Hive.  In the future we plan to contribute changes to extend it, enforcing a common set of permissions over data accessed through additional components including HCatalog, MapReduce, Pig, YARN, Spark, Sqoop, and more.  Apache Sentry implements an authorization provider that can be plugged in to the unified authorization framework goal (above) and the associated jira (below).

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

[PIG-3289: Encryption aware load and store functions](https://issues.apache.org/jira/browse/PIG-3289)

With HADOOP-9331 and MAPREDUCE-5025 in place, MapReduce jobs have the ability to process and output the encrypted data. For pig users, to take advantage of this capability and process and output the encrypted data, pig should have capability to accept the key and pass it to the MapReduce , so that MapReduce can do the job on the behalf of pig. The scope of this Jira is limited to passing the key to MapReduce and takes the advantage of HADOOP-9331 and MAPREDUCE-5025 without breaking Pig.
To achieve that, file input formats or file output formats interface will be modified to handle CryptoCodec and set the context properly and provide key facilities.
The file [input/output] formats that does not support compression (by using CompressionCodec) can't be addressed by this work because the encryption feature (HADOOP-9331 and related) is based on CompressionCodec.
By making this change, pig can cover the following use case:
a. Pig user can run a query on an encrypted data
b. Pig users can store an encrypted data 
c. Outputting the encrypted data
Accessing of encrypted HBase storage/tables or any other encrypted storage format, who pig can query, should be addressed with separate Jiras, if needed because HBase | Other systems might have specific key management mechanisms or interfacing with Pig.
To handle versions of Hadoop that do not have crypto support, we can avoid compilation problems by segregating crypto API usage into separate files to be included only if a flag is defined on the Ant command line (something like –Dcrypto).

[PIG-3404: Improve Pig to ignore bad files or inaccessible files or folders](https://issues.apache.org/jira/browse/PIG-3404)

There are use cases in Pig:
A directory is used as the input of a load operation. It is possible that one or more files in that directory are bad files (for example, corrupted or bad data caused by compression).
A directory is used as the input of a load operation. The current user may not have permission to access any subdirectories or files of that directory.
The current Pig implementation will abort the whole Pig job for such cases. It would be useful to have option to allow the job to continue and ignore the bad files or inaccessible files/folders without abort the job, ideally, log or print a warning for such error or violations. This requirement is not trivial because for big data set for large analytics applications, this is not always possible to sort out the good data for processing; Ignore a few of bad files may be a better choice for such situations.
We propose to use "Ignore bad files" flag to address this problem. AvroStorage and related file format in Pig already has this flag but it is not complete to cover all the cases mentioned above. We would improve the PigStorage and related text format to support this new flag as well as improve AvroStorage and related facilities to completely support the concept.
The flag is Storage based (For example, PigStorage or AvroStorage) and can be set for each load operation respectively. The value of this flag will be false if it is not explicitly set. Ideally, we can provide a global pig parameter which forces the default value to true for all load functions even if it is not explicitly set in the LOAD statement.

[HIVE-5207: Support data encryption for Hive tables](https://issues.apache.org/jira/browse/HIVE-5207)

For sensitive and legally protected data such as personal information, it is a common practice that the data is stored encrypted in the file system. To enable Hive with the ability to store and query the encrypted data is very crucial for Hive data analysis in enterprise. 
When creating table, user can specify whether a table is an encrypted table or not by specify a property in TBLPROPERTIES. Once an encrypted table is created, query on the encrypted table is transparent as long as the corresponding key management facilities are set in the running environment of query. We can use hadoop crypto provided by HADOOP-9331 for underlying data encryption and decryption. 
As to key management, we would support several common key management use cases. First, the table key (data key) can be stored in the Hive metastore associated with the table in properties. The table key can be explicit specified or auto generated and will be encrypted with a master key. There are cases that the data being processed is generated by other applications, we need to support externally managed or imported table keys. Also, the data generated by Hive may be consumed by other applications in the system. We need to a tool or command for exporting the table key to a java keystore for using externally.
To handle versions of Hadoop that do not have crypto support, we can avoid compilation problems by segregating crypto API usage into separate files (shims) to be included only if a flag is defined on the Ant command line (something like -Dcrypto=true).

[AVRO-1371: Support of data encryption for Avro file](https://issues.apache.org/jira/browse/AVRO-1371)

Avro file format is widely used in Hadoop. As data security is getting more and more attention in Hadoop community, we propose to improve Avro file format to be able to handle data encryption and decryption.
Similar to compression and decompression, encryption and decryption can be implemented with Codecs, a concept that already exists in Avro. However, Avro Codec context handling needs to be extended to support per-codec contexts, such as encryption keys, for encryption and decryption.
Avro supports multiple language implementations. This is an umbrella JIRA for this work and the implementation work for each language will be addressed in sub tasks.

[HDFS-5143: Hadoop cryptographic file system](https://issues.apache.org/jira/browse/HDFS-5143)

There is an increasing need for securing data when Hadoop customers use various upper layer applications, such as Map-Reduce, Hive, Pig, HBase and so on.

HADOOP CFS (HADOOP Cryptographic File System) is used to secure data, based on HADOOP FilterFileSystem decorating DFS or other file systems, and transparent to upper layer applications. It is configurable, scalable and fast.

High level requirements:
1. Transparent to and no modification required for upper layer applications.
2. Seek, PositionedReadable are supported for input stream of CFS if the wrapped file system supports them.
3. Very high performance for encryption and decryption, they will not become bottleneck.
4. Can decorate HDFS and all other file systems in Hadoop, and will not modify existing structure of file system, such as namenode and datanode structure if the wrapped file system is HDFS.
5. Admin can configure encryption policies, such as which directory will be encrypted.
6. A robust key management framework.
7. Support Pread and append operations if the wrapped file system supports them.

[HADOOP-9134: Unified server side user groups mapping service](https://issues.apache.org/jira/browse/HADOOP-9134)

This proposes to provide/expose the server side user group mapping service in NameNode to clients so that user group mapping can be kept in the single place and thus unified in all nodes and clients.

[HADOOP-8943: Support multiple group mapping providers](https://issues.apache.org/jira/browse/HADOOP-8943)

Discussed with Natty about LdapGroupMapping, we need to improve it so that:
1. It's possible to do different group mapping for different users/principals. For example, AD user should go to LdapGroupMapping service for group, but service principals such as hdfs, mapred can still use the default one ShellBasedUnixGroupsMapping;
2. Multiple ADs can be supported to do LdapGroupMapping;
3. It's possible to configure what kind of users/principals (regarding domain/realm is an option) should use which group mapping service/mechanism.
4. It's possible to configure and combine multiple existing mapping providers without writing codes implementing new one.

[HADOOP-9477: posixGroups support for LDAP groups mapping service](https://issues.apache.org/jira/browse/HADOOP-9477)

It would be nice to support posixGroups for LdapGroupsMapping service. Below is from current description for the provider:
hadoop.security.group.mapping.ldap.search.filter.group:
An additional filter to use when searching for LDAP groups. This should be
changed when resolving groups against a non-Active Directory installation.
posixGroups are currently not a supported group class.

[ZOOKEEPER-1749: Login outside of Zookeeper client](https://issues.apache.org/jira/browse/ZOOKEEPER-1749)

This proposes to allow Zookeeper client to reuse login credentials and subject from outside, avoiding redundant logins and related configurations for services that utilizes Zookeeper.

[OOZIE-1376: Extending Oozie ACLs like admin groups and proxy users to support both groups and users](https://issues.apache.org/jira/browse/OOZIE-1376)

Currently Oozie relevant ACLs supports only users in some case or only groups in other case, which is not consistent with other components like Hadoop, HBase and Hive. Supporting both users and groups in ACLs can simplify the admin configuration work, and also help implement more advanced access control such as RBAC based on the ACLs scheme. For example RBAC can simply translate role privilege into corresponding ACLs with the users and groups assigned to the role.

[OOZIE-1232: GroupsService should be able to reference Hadoop configurations in Hadoop configuration folder](https://issues.apache.org/jira/browse/OOZIE-1232)

Oozie GroupsService wraps Hadoop user groups mapping to get groups for user, which requires to reference Hadoop configurations, especially the properties related to groups mapping provider (such as LdapGroupsMapping).
To avoid replication of such configurations into oozie-site.xml, mechanism is needed to configure the Hadoop configurations folder (often mentioned hadoop-conf) for the service, as HadoopAccessorService currently does.
Such work can be done per Service, as HadoopAccessorService, but would it be better to avoid code changes or similar work when other Service also needs to do that in future.

[HBASE-7524: hbase-policy.xml is improperly set thus all rules in it can be by-passed](https://issues.apache.org/jira/browse/HBASE-7524)

This resolved a service level authorization policy issue.

[HBASE-9570: With AccessDeniedException, HBase shell would be better to just display the error message to be user friendly](https://issues.apache.org/jira/browse/HBASE-9570)

This improved HBase shell about exception reporting when access denied due to ACL.


[HADOOP-9125: LdapGroupsMapping threw CommunicationException after some idle time](https://issues.apache.org/jira/browse/HADOOP-9125)

This resolved an issue found in LdapGrouspMapping provider.
