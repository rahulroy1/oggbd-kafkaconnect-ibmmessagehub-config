***************************************************************************************************************
This distribution only contains the config files required to connect IBM Message Hub using OGG BD Kafka Connect
***************************************************************************************************************

Oracle GoldenGate for Kafka Connect Handler to IBM Message Hub
Version 1.0

-------------
Functionality
-------------

The Kafka Connect Handler takes change data capture operations from a source trail file and generates data structs (org.apache.kafka.connect.data.Struct) as well as the associated schemas (org.apache.kafka.connect.data.Schema). The data structs are serialized via configured converters then enqueued onto Kafka topics. The topic name used corresponds to the fully qualified source table name as obtained from the GoldenGate trail file. Individual operations consist of inserts, updates, and delete operations executed on the source RDBMS. Insert and update operation data include the after change data. Delete operations include the before change data. A primary key update is a special case for an update where one or more of the primary key(s) is/are changed. The primary key update represents a special case in that without the before image data it is not possible to determine what row is actually changing when only in possession of the after change data. The default behavior of a primary key update is to ABEND in the Kafka Connect formatter. 

NOTE: In this exercise, we are going to use the sample trail file that is provided by OGG BD installation under AdapterExamples/trail/ which contains transactions for two sample two tables QASOURCE.TCUSTMER and QASOURCE.TCUSTORD and push those to IBM Message Hub.

------------------
Supported Versions
------------------

These configurations is tested with the following product versions.

- Oracle GoldenGate for Big Data 12.3.1.1.1
a. Linux: http://download.oracle.com/otn/goldengate/123111/123111_ggs_Adapters_Linux_x64.zip
b. Windows: http://download.oracle.com/otn/goldengate/123111/123111_ggs_Adapters_Windows_x64.zip
- Apache Kafka (kafka_2.12-0.10.2.1.tgz)

------------------
Assumptions
------------------

1. User has already created a IBM Message Hub instance  in IBM Cloud
2. User has created a new Service Credential and has the details from View Credentials link.

------------
Installation
------------

a. Extract OGG BD binaries (123111_ggs_Adapters_<OS Type>_x64.zip) to any folder of your choice. Let's name it as oggkafka
b. login to OGG-BD with "ggsci" in Windows or "./ggsci" in Linux
c. Run following commands from GGSCI prompt:

GGSCI (localhost.localdomain) 1> CREATE SUBDIRS
GGSCI (localhost.localdomain) 2> EDIT PARAM MGR

PORT 7801

d. Above should create following dirs under 'oggkafka':
Creating subdirectories under current directory <path>/oggkafka

Parameter files                <path>/oggkafka/dirprm: created

Report files                   <path>/oggkafka/dirrpt: created

Checkpoint files               <path>/oggkafka/dirchk: created

Process status files           <path>/oggkafka/dirpcs: created

SQL script files               <path>/oggkafka/dirsql: created

Database definitions files     <path>/oggkafka/dirdef: created

Extract data files             <path>/oggkafka/dirdat: created

Temporary files                <path>/oggkafka/dirtmp: created


--------------
Configuration
--------------

a. Extract ogg-kafkaconnect-ibmmessagehub-connector-v1.zip to any directory in Windows or Linux. Let's name it as messagehubconnector

b. Copy config files to actual OGG BD installation directory i.e. oggkafka

1. If you are on Windows, copy messagehubconnector/dirprm/windows/* to oggkafka/dirprm/
2. If you are on Linux, copy messagehubconnector/dirprm/linux/* to oggkafka/dirprm/

NOTE: 
messagehubconnector/dirprm/<OS TYPE>/ directory contains sample configuration file for IBM Message Hub

kafkaconnect-messagehub.properties – Message Hub kafkaconnect properties file
kcmsghb.prm – sample replicat config file
kcmessagehub.props – sample OGG properties file

3. copy oggkafka/AdapterExamples/trail/ to oggkafka/dirdat/

c. Now update oggkafka/dirprm/kafkaconnect-messagehub.properties with your IBM Message Hub specific details.
NOTE: Replace KAFKA_BROKERS_SASL, USER, and PASSWORD with the values from your Message Hub Service Credentials tab in the IBM Cloud console.

d. Create topics in IBM Message Hub having the same format as mentioned in kcmessagehub.props(gg.handler.kafkaconnect.topicMappingTemplate value).
For this example, topic names to be created as: QASOURCE.TCUSTMER and QASOURCE.TCUSTORD


------------
Execution
-------------

Post configuration, follow below steps for execution:
1. login to OGG-BD with "ggsci" in Windows or "./ggsci" in Linux
2. Create OGG replicat
GGSCI (localhost.localdomain) 3> add replicat kcmsghb exttrail dirdat/trail

GGSCI (localhost.localdomain) 3> start mgr

GGSCI (localhost.localdomain) 3> start kcmsghb

3. Check OGG log under oggkafka/dirrpt/ to see the records are read from trail file and pass through kafkaconnect handler

NOTE: Following log should indicate that it's working fine:

DEBUG 2018-05-17 21:56:09,113 [main] DEBUG (KafkaConnectProducer.java:107) - Sending a producer record to Kafka topic [gcssx_QASOURCE.TCUSTORD] using message key [null].  Message size [1055] bytes.
TRACE 2018-05-17 21:56:09,114 [main] TRACE (AbstractDataSource.java:787) - AbstractDataSource: getCurrentTx: activeTxns size is: 1
TRACE 2018-05-17 21:56:09,115 [main] TRACE (AbstractDataSource.java:709) - prune operation buffer: txSize was=1, new=0, buffer=0
TRACE 2018-05-17 21:56:09,115 [main] TRACE (AbstractDataSource.java:787) - AbstractDataSource: getCurrentTx: activeTxns size is: 1
TRACE 2018-05-17 21:56:09,116 [main] TRACE (AbstractDataSource.java:709) - prune operation buffer: txSize was=0, new=0, buffer=0
TRACE 2018-05-17 21:56:09,116 [main] TRACE (DataSourceStats.java:309) - processing events => adding=13ms; total=0:00:06.343 [total = 6 sec ]
DEBUG 2018-05-17 21:56:09,117 [main] DEBUG (UserExitDataSource.java:1349) - == JNI == getAsyncCheckpoint() => 0/3286
DEBUG 2018-05-17 21:56:09,134 [main] DEBUG (UserExitDataSource.java:1468) - == JNI == commitTx()
TRACE 2018-05-17 21:56:09,134 [main] TRACE (AbstractDataSource.java:787) - AbstractDataSource: getCurrentTx: activeTxns size is: 1
TRACE 2018-05-17 21:56:09,135 [main] TRACE (AbstractDataSource.java:787) - AbstractDataSource: getCurrentTx: activeTxns size is: 1
TRACE 2018-05-17 21:56:09,139 [main] TRACE (AbstractDataSource.java:509) - fireTransactionCommit: tx=00000000000000003286
DEBUG 2018-05-17 21:56:09,140 [main] DEBUG (AbstractHandler.java:656) - Event: handler=kafkaconnect, transactionCommit ( Commit transaction ) DsTransaction [ops=12, buffered=0, state=END, start=2015-11-05 18:45:39.000000, end=2015-11-05 18:45:39.000000]
DEBUG 2018-05-17 21:56:09,140 [main] DEBUG (KafkaConnectProducer.java:130) - Flushing the Kafka connection.
DEBUG 2018-05-17 21:56:09,355 [main] DEBUG (UserExitDataSource.java:589) - == JNI callback == setCheckpoint(0/5660)
TRACE 2018-05-17 21:56:09,355 [main] TRACE (DataSourceStats.java:309) - processing events => adding=216ms; total=0:00:06.559 [total = 6 sec ]

4. You can run local Kafka JAVA consumer to check IBM Message hub queues to see messages being consumed by Message Hub kafka