```
root@server12-prod:~# mysqlsh
MySQL Shell 8.4.10

Copyright (c) 2016, 2026, Oracle and/or its affiliates.
Oracle is a registered trademark of Oracle Corporation and/or its affiliates.
Other names may be trademarks of their respective owners.

Type '\help' or '\?' for help; '\quit' to exit.
 MySQL  SQL > \js
Switching to JavaScript mode...
 MySQL  JS > shell.connect('clusteradmin@server12-prod:3306');
Creating a session to 'clusteradmin@server12-prod:3306'
Fetching schema names for auto-completion... Press ^C to stop.
Your MySQL connection id is 246679
Server version: 8.0.46 MySQL Community Server - GPL
No default schema selected; type \use <schema> to set one.
<ClassicSession:clusteradmin@server12-prod:3306>
 MySQL  server12-prod:3306 ssl  JS > var cluster = dba.getCluster();
 MySQL  server12-prod:3306 ssl  JS > cluster.status();
{
    "clusterName": "myCluster", 
    "defaultReplicaSet": {
        "name": "default", 
        "primary": "server12-prod:3306", 
        "ssl": "REQUIRED", 
        "status": "OK_NO_TOLERANCE_PARTIAL", 
        "statusText": "Cluster is NOT tolerant to any failures. 1 member is not active.", 
        "topology": {
            "server10-prod:3306": {
                "address": "server10-prod:3306", 
                "instanceErrors": [
                    "ERROR: applier thread of Group Replication Recovery channel stopped with an error: Worker 1 failed executing transaction 'd99d8528-9bab-11f1-af75-fa163effda98:1128175' at source log binlog.000004, end_log_pos 129419526; Could not execute Update_rows event on table parrot_v2.media_merge_tasks; Can't find record in 'media_merge_tasks', Error_code: 1032; handler error HA_ERR_KEY_NOT_FOUND; the event's source log binlog.000004, end_log_pos 129419526 (1032) at 2026-09-06 01:00:32.178402", 
                    "ERROR: group_replication has stopped with an error."
                ], 
                "memberRole": "SECONDARY", 
                "memberState": "ERROR", 
                "mode": "n/a", 
                "readReplicas": {}, 
                "role": "HA", 
                "status": "(MISSING)", 
                "version": "8.0.46"
            }, 
            "server11-prod:3306": {
                "address": "server11-prod:3306", 
                "memberRole": "SECONDARY", 
                "mode": "R/O", 
                "readReplicas": {}, 
                "replicationLag": "applier_queue_applied", 
                "role": "HA", 
                "status": "ONLINE", 
                "version": "8.0.46"
            }, 
            "server12-prod:3306": {
                "address": "server12-prod:3306", 
                "memberRole": "PRIMARY", 
                "mode": "R/W", 
                "readReplicas": {}, 
                "replicationLag": "applier_queue_applied", 
                "role": "HA", 
                "status": "ONLINE", 
                "version": "8.0.46"
            }
        }, 
        "topologyMode": "Single-Primary"
    }, 
    "groupInformationSourceMember": "server12-prod:3306"
}
 MySQL  server12-prod:3306 ssl  JS > cluster.addInstance('clusteradmin@server10-prod:3306', {recoveryMethod: 'clone'});
Cluster.addInstance: The instance 'server10-prod:3306' is already part of another InnoDB Cluster (RuntimeError)
 MySQL  server12-prod:3306 ssl  JS > cluster.removeInstance('server10-prod:3306', {force: true});
NOTE: server10-prod:3306 is reachable but has state ERROR
The instance will be removed from the InnoDB Cluster.

NOTE: Transaction sync was skipped
* Instance 'server10-prod:3306' is attempting to leave the cluster...

The instance 'server10-prod:3306' was successfully removed from the cluster.

 MySQL  server12-prod:3306 ssl  JS > cluster.addInstance('clusteradmin@server10-prod:3306', {recoveryMethod: 'clone'});

Clone based recovery selected through the recoveryMethod option

Validating instance configuration at server10-prod:3306...

This instance reports its own address as server10-prod:3306

Instance configuration is suitable.
NOTE: Group Replication will communicate with other members using 'server10-prod:3306'. Use the localAddress option to override.

* Checking connectivity and SSL configuration...

A new instance will be added to the InnoDB Cluster. Depending on the amount of
data on the cluster this might take from a few seconds to several hours.

Adding instance to the cluster...

Monitoring recovery process of the new cluster member. Press ^C to stop monitoring and let it continue in background.
Clone based state recovery is now in progress.

NOTE: A server restart is expected to happen as part of the clone process. If the
server does not support the RESTART command or does not come back after a
while, you may need to manually start it back.

* Waiting for clone to finish...
NOTE: server10-prod:3306 is being cloned from server12-prod:3306
** Stage DROP DATA: Completed 
** Clone Transfer  
    FILE COPY  ############################################################  100%  Completed
    PAGE COPY  ############################################################  100%  Completed
    REDO COPY  ############################################################  100%  Completed

NOTE: server10-prod:3306 is shutting down...

* Waiting for server restart... ready 
* server10-prod:3306 has restarted, waiting for clone to finish...
** Stage RESTART: Completed
* Clone process has finished: 2.52 GB transferred in 21 sec (119.84 MB/s)

Incremental state recovery is now in progress.

* Waiting for distributed recovery to finish...
NOTE: 'server10-prod:3306' is being recovered from 'server12-prod:3306'
* Distributed recovery has finished

The instance 'server10-prod:3306' was successfully added to the cluster.

 MySQL  server12-prod:3306 ssl  JS > cluster.status();
{
    "clusterName": "myCluster", 
    "defaultReplicaSet": {
        "name": "default", 
        "primary": "server12-prod:3306", 
        "ssl": "REQUIRED", 
        "status": "OK", 
        "statusText": "Cluster is ONLINE and can tolerate up to ONE failure.", 
        "topology": {
            "server10-prod:3306": {
                "address": "server10-prod:3306", 
                "memberRole": "SECONDARY", 
                "mode": "R/O", 
                "readReplicas": {}, 
                "replicationLag": "applier_queue_applied", 
                "role": "HA", 
                "status": "ONLINE", 
                "version": "8.0.46"
            }, 
            "server11-prod:3306": {
                "address": "server11-prod:3306", 
                "memberRole": "SECONDARY", 
                "mode": "R/O", 
                "readReplicas": {}, 
                "replicationLag": "applier_queue_applied", 
                "role": "HA", 
                "status": "ONLINE", 
                "version": "8.0.46"
            }, 
            "server12-prod:3306": {
                "address": "server12-prod:3306", 
                "memberRole": "PRIMARY", 
                "mode": "R/W", 
                "readReplicas": {}, 
                "replicationLag": "applier_queue_applied", 
                "role": "HA", 
                "status": "ONLINE", 
                "version": "8.0.46"
            }
        }, 
        "topologyMode": "Single-Primary"
    }, 
    "groupInformationSourceMember": "server12-prod:3306"
}
```
