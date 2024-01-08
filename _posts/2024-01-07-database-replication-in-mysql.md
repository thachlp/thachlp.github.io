---
layout: post
title: Database Replication in MySQL
tags: [Databases]
comments: true
---
### Replication
Replication means **keeping a copy of the same data** on multiple machines that are connected via a network. We assume that our dataset is small enough to hold a copy of the entire dataset. Benefits of replication include:

- Reducing latency by keeping data geographically closer to users.
- Increasing availability by allowing the system to continue working even if some parts fail.
- Scaling out the number of machines that can serve read queries and thus increase read throughput.

### Leader and follower
Each node that stores a copy of the database is call a replica, every write to the database needs to be processed by every replica. The most common solution for this is called leader-based replication (master-slave replication):
1. One of the replicas is designed the leader, when clients want to write to the database, they must send their requests to the leader, which first writes the new data to its local storage.
2. The other replicas are known as followers, whenever the leader writes new data to its local storage, it also sends the data change to all of its followers as part of a replication log.
3. When a client wants to read from the database, it can query either the leader or any of the followers, however, writes are only accepted on the leader.
4. There can be a delay, or replication lag, between when the leader writes data and when the followers receive and apply these changes. This can lead to a state of eventual consistency, where followers may temporarily serve stale data until they catch up with the leader.

Here is the diagram to illustrate it:

```
                            +------------------+       
                            |                  |       
                            |User (Read/Write) |       
                            |                  |       
                            +---------+--------+       
                                      |
                                      v
                            +---------+--------+       
                            |                  |       
                            |    Master DB     |       
                            |                  |       
                            +---------+--------+       
                                      |
               +----------------------+------------------------+ 
               |                                               |
               v                                               v
  +------------+----------+                        +-----------+------------+
  |                       |                        |                        |
  |     Slave DB 1        |                        |     Slave DB 2         |
  | (Read Requests        |                        | (Read Requests         |
  |  Handled)             |                        |  Handled)              |
  |                       |                        |                        |
  +-----------------------+                        +------------------------+
              ^                                                ^
              |                                                |
    +---------+--------+                              +--------+---------+
    |                  |                              |                  |
    |   User (Read)    |                              |   User (Read)    |
    |                  |                              |                  |
    +------------------+                              +--------+---------+
```

### Setup MySQL master-slave using Docker

```bash
#1 Create a network
docker network create mysql-network

#2 Create a mysql-master
docker run --name mysql-master \
  --network mysql-network \
  -e MYSQL_ROOT_PASSWORD=123456 \
  -p 3307:3306 \
  -d mysql:latest \
  --server-id=1 \ # it is important for uniquely identifying each server in the cluster
  --log-bin=mysql-bin
 
#Config the master database
docker exec -it mysql-master mysql -uroot -p123456
mysql> CREATE USER 'replicator'@'%' IDENTIFIED WITH 'mysql_native_password' BY '123456';
mysql> GRANT REPLICATION SLAVE ON *.* TO 'replicator'@'%';
mysql> FLUSH PRIVILEGES; 

# Verify setup
mysql> SHOW MASTER STATUS;
+------------------+----------+--------------+------------------+-------------------+
| File             | Position | Binlog_Do_DB | Binlog_Ignore_DB | Executed_Gtid_Set |
+------------------+----------+--------------+------------------+-------------------+
| mysql-bin.000007 |      157 |              |                  |                   |
+------------------+----------+--------------+------------------+-------------------+
1 row in set (0.00 sec)

#4 Add slave
docker run --name mysql-slave1 \
  --network mysql-network \
  -e MYSQL_ROOT_PASSWORD=123456 \
  -p 3308:3306 \
  -d mysql:latest \
  --server-id=2

#Config the slave database
docker exec -it mysql-slave1 mysql -uroot -p123456
mysql> CHANGE MASTER TO
  MASTER_HOST='mysql-master',
  MASTER_USER='replicator',
  MASTER_PASSWORD='123456',
  MASTER_LOG_FILE='mysql-bin.000007', -- Use log file name from master
  MASTER_LOG_POS=157; -- Use log position from master

# Verify SLAVE setup
mysql> SHOW SLAVE STATUS \G
*************************** 1. row ***************************
               Slave_IO_State: 
                  Master_Host: mysql-master
                  Master_User: replicator
                  Master_Port: 3306
                Connect_Retry: 60
              Master_Log_File: mysql-bin.000007
          Read_Master_Log_Pos: 157
               Relay_Log_File: 8cabcceaf6a7-relay-bin.000001
                Relay_Log_Pos: 4
        Relay_Master_Log_File: mysql-bin.000007

# Start slave
mysql> START SLAVE;
Query OK, 0 rows affected, 1 warning (0.02 sec)

#5 Repeat the above steps for setting up additional slaves
```

Now we have mysql-cluster with 1 master and muti slaves. We can test the replication by create a table on master and see the data has replicated to slaves.

### Proxy
From the application's perspective, it is typically necessary to manually handle the routing of read and write requests to the master or slave databases. In the event of a master failure and the subsequent election of a new master, the application may need to update its configuration to point to the new master, which can lead to downtime.

In a practical environment, a proxy is often used to manage requests. Tools such as Orchestrator, MySQL Router, ProxySQL, and Percona Replication Manager can automate the process of redirecting requests, detecting master failures, and promoting a slave to master. These tools help to minimize downtime by providing a seamless transition during failover events.

The diagram below illustrates this setup:

```bash
                            +------------------+       
                            |                  |       
                            |   Application    |       
                            | (Reads & Writes) |       
                            |                  |       
                            +---------+--------+       
                                      |
                                      v
                            +---------+--------+       
                            |                  |       
                            |    ProxySQL      | <---- Handles read/write split
                            | (Load Balancer)  |       and load balancing
                            +---------+--------+       
                                      |
               +----------------------+------------------------+ 
               |                      |                        |
               v                      v                        v
  +------------+----------+ +---------+--------+  +------------+----------+
  |                       | |                  |  |                       |
  |    Master DB          | |     Slave DB 1   |  |     Slave DB 2        |
  | (Write Requests       | | (Read Requests   |  | (Read Requests        |
  |  Handled)             | |  Handled)        |  |  Handled)             |
  |                       | |                  |  |                       |
  +-----------------------+ +------------------+  +-----------------------+
```

In an upcoming post, I plan to install ProxySQL and conduct some tests to verify this setup.

**References:** 
- *Leaders and Followers, Chapter 5: Replication, in “Designing Data-Intensive Applications”* by **Martin Kleppman**
