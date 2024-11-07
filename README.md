# mongodb-replica-set-docker-compose

## Steps of mongo replica set instantiation
- clone this repository at your local host
- apply docker compose command on file ".yaml" file
- interact with mongosh of any instance of mongo. Apply a command to initiate the replica set among them.
- verify the replica-set status.



### Apply docker compose command on file ".yaml" file
```
docker compose -f mongo-replica-set.yaml up -d
```
"-f" flag for indicating file. "-d" for run docker containers in detached mode.

### Interact with mongosh of any instance of mongo cluster and apply a command to initiate the replica set among them
Let's inspect the docker network to find out private ip addresses of mongodb instances.
```
docker network inspect mongodb-replica-set-docker-compose_mongo-replica-cluster
```
Output:
```
[
    {
        "Name": "mongodb-replica-set-docker-compose_mongo-replica-cluster",
        "Id": "9283da3d2bf7e8a55ff797874f4a16d4b2e0d4467efffe67f1226b43ac16ad99",
        "Created": "2024-11-07T10:00:31.538627083+06:00",
        "Scope": "local",
        "Driver": "bridge",
        "EnableIPv6": false,
        "IPAM": {
            "Driver": "default",
            "Options": null,
            "Config": [
                {
                    "Subnet": "172.22.0.0/16",
                    "Gateway": "172.22.0.1"
                }
            ]
        },
        "Internal": false,
        "Attachable": false,
        "Ingress": false,
        "ConfigFrom": {
            "Network": ""
        },
        "ConfigOnly": false,
        "Containers": {
            "4bb2b660310210cb24c219ece06931e6a94e3be1cce302d57661e2380964430a": {
                "Name": "mongo-replica-03",
                "EndpointID": "7890f146ae0ecb40d2e9364d14c94780046e392270c27c49fd80200c281cba7c",
                "MacAddress": "02:42:ac:16:00:02",
                "IPv4Address": "172.22.0.2/16",
                "IPv6Address": ""
            },
            "55a05fb5b15703e4ddd9611099b2ef904d7a3340071eb92ecb7ac1e7d81ff8b2": {
                "Name": "mongo-replica-01",
                "EndpointID": "3d59d420039eaab3e14531ff9e05c41ca6ea76c2c86b5d0f81538f0d2f9fa570",
                "MacAddress": "02:42:ac:16:00:04",
                "IPv4Address": "172.22.0.4/16",
                "IPv6Address": ""
            },
            "8620e16269d899c1142a493140a45da5aac2ac23bf858e1890e9352be9b8c029": {
                "Name": "mongo-replica-02",
                "EndpointID": "cfa36697631e28b6e27a3f40ed7ddebd51659948c903bf930fd6f4cfc49bdba2",
                "MacAddress": "02:42:ac:16:00:03",
                "IPv4Address": "172.22.0.3/16",
                "IPv6Address": ""
            },
            "99dd7d32e1cb8bb9e442032c8b54ff7128cc9027f96a22ad98bc21f35b776ed3": {
                "Name": "mongo-express-replica-set",
                "EndpointID": "c4e32510fbfdead8279f79ee5690b4a3386ac62c0ab20155e3d29975dd7d93ef",
                "MacAddress": "02:42:ac:16:00:05",
                "IPv4Address": "172.22.0.5/16",
                "IPv6Address": ""
            }
        },
        "Options": {},
        "Labels": {
            "com.docker.compose.network": "mongo-replica-cluster",
            "com.docker.compose.project": "mongodb-replica-set-docker-compose",
            "com.docker.compose.version": "2.29.7"
        }
    }
]
```
We can find out the ip addresses such as '172.22.0.4', '172.22.0.3' and '172.22.0.2' for instances 1, 2, 3 respectively.


```
docker exec -it mongoreplica01 mongosh
```
After getting into mongosh interactive shell, you check out the replica-set status.

```
> rs.status()
```
Output will be like this:
```
MongoServerError[NotYetInitialized]: no replset config has been received
```
It means that replica-set is yet to configure.  To configure the replica-set, write this below command to instantiate:

```
rs.initiate({_id: "mongoReplicaSet", members:[{_id:0, host:"172.22.0.4"},{_id:1, host:"172.22.0.3"},{_id:2, host:"172.22.0.2"}]})
```

Output:
```
test> rs.initiate({_id: "mongoReplicaSet", members:[{_id:0, host:"172.22.0.4"},{_id:1, host:"172.22.0.3"},{_id:2, host:"172.22.0.2"}]})
{
  ok: 1,
  '$clusterTime': {
    clusterTime: Timestamp({ t: 1730971950, i: 1 }),
    signature: {
      hash: Binary.createFromBase64('AAAAAAAAAAAAAAAAAAAAAAAAAAA=', 0),
      keyId: Long('0')
    }
  },
  operationTime: Timestamp({ t: 1730971950, i: 1 })
}
Congratulations!! Mongodb replica set is setup successfully.

```
### Verify the replica-set status.

Now checking the status of cluster, you will get proper output:
```
mongoReplicaSet [direct: primary] test> rs.status()
{
  set: 'mongoReplicaSet',
  date: ISODate('2024-11-07T09:32:51.650Z'),
  myState: 1,
  term: Long('1'),
  syncSourceHost: '',
  syncSourceId: -1,
  heartbeatIntervalMillis: Long('2000'),
  majorityVoteCount: 2,
  writeMajorityCount: 2,
  votingMembersCount: 3,
  writableVotingMembersCount: 3,
  optimes: {
    lastCommittedOpTime: { ts: Timestamp({ t: 1730971961, i: 16 }), t: Long('1') },
    lastCommittedWallTime: ISODate('2024-11-07T09:32:41.787Z'),
    readConcernMajorityOpTime: { ts: Timestamp({ t: 1730971961, i: 16 }), t: Long('1') },
    appliedOpTime: { ts: Timestamp({ t: 1730971961, i: 16 }), t: Long('1') },
    durableOpTime: { ts: Timestamp({ t: 1730971961, i: 16 }), t: Long('1') },
    writtenOpTime: { ts: Timestamp({ t: 1730971961, i: 16 }), t: Long('1') },
    lastAppliedWallTime: ISODate('2024-11-07T09:32:41.787Z'),
    lastDurableWallTime: ISODate('2024-11-07T09:32:41.787Z'),
    lastWrittenWallTime: ISODate('2024-11-07T09:32:41.787Z')
  },
  lastStableRecoveryTimestamp: Timestamp({ t: 1730971950, i: 1 }),
  electionCandidateMetrics: {
    lastElectionReason: 'electionTimeout',
    lastElectionDate: ISODate('2024-11-07T09:32:41.694Z'),
    electionTerm: Long('1'),
    lastCommittedOpTimeAtElection: { ts: Timestamp({ t: 1730971950, i: 1 }), t: Long('-1') },
    lastSeenWrittenOpTimeAtElection: { ts: Timestamp({ t: 1730971950, i: 1 }), t: Long('-1') },
    lastSeenOpTimeAtElection: { ts: Timestamp({ t: 1730971950, i: 1 }), t: Long('-1') },
    numVotesNeeded: 2,
    priorityAtElection: 1,
    electionTimeoutMillis: Long('10000'),
    numCatchUpOps: Long('0'),
    newTermStartDate: ISODate('2024-11-07T09:32:41.737Z'),
    wMajorityWriteAvailabilityDate: ISODate('2024-11-07T09:32:42.230Z')
  },
  members: [
    {
      _id: 0,
      name: '172.22.0.4:27017',
      health: 1,
      state: 1,
      stateStr: 'PRIMARY',
      uptime: 1581,
      optime: { ts: Timestamp({ t: 1730971961, i: 16 }), t: Long('1') },
      optimeDate: ISODate('2024-11-07T09:32:41.000Z'),
      optimeWritten: { ts: Timestamp({ t: 1730971961, i: 16 }), t: Long('1') },
      optimeWrittenDate: ISODate('2024-11-07T09:32:41.000Z'),
      lastAppliedWallTime: ISODate('2024-11-07T09:32:41.787Z'),
      lastDurableWallTime: ISODate('2024-11-07T09:32:41.787Z'),
      lastWrittenWallTime: ISODate('2024-11-07T09:32:41.787Z'),
      syncSourceHost: '',
      syncSourceId: -1,
      infoMessage: 'Could not find member to sync from',
      electionTime: Timestamp({ t: 1730971961, i: 1 }),
      electionDate: ISODate('2024-11-07T09:32:41.000Z'),
      configVersion: 1,
      configTerm: 1,
      self: true,
      lastHeartbeatMessage: ''
    },
    {
      _id: 1,
      name: '172.22.0.3:27017',
      health: 1,
      state: 2,
      stateStr: 'SECONDARY',
      uptime: 21,
      optime: { ts: Timestamp({ t: 1730971961, i: 16 }), t: Long('1') },
      optimeDurable: { ts: Timestamp({ t: 1730971961, i: 16 }), t: Long('1') },
      optimeWritten: { ts: Timestamp({ t: 1730971961, i: 16 }), t: Long('1') },
      optimeDate: ISODate('2024-11-07T09:32:41.000Z'),
      optimeDurableDate: ISODate('2024-11-07T09:32:41.000Z'),
      optimeWrittenDate: ISODate('2024-11-07T09:32:41.000Z'),
      lastAppliedWallTime: ISODate('2024-11-07T09:32:41.787Z'),
      lastDurableWallTime: ISODate('2024-11-07T09:32:41.787Z'),
      lastWrittenWallTime: ISODate('2024-11-07T09:32:41.787Z'),
      lastHeartbeat: ISODate('2024-11-07T09:32:49.720Z'),
      lastHeartbeatRecv: ISODate('2024-11-07T09:32:50.725Z'),
      pingMs: Long('0'),
      lastHeartbeatMessage: '',
      syncSourceHost: '172.22.0.4:27017',
      syncSourceId: 0,
      infoMessage: '',
      configVersion: 1,
      configTerm: 1
    },
    {
      _id: 2,
      name: '172.22.0.2:27017',
      health: 1,
      state: 2,
      stateStr: 'SECONDARY',
      uptime: 21,
      optime: { ts: Timestamp({ t: 1730971961, i: 16 }), t: Long('1') },
      optimeDurable: { ts: Timestamp({ t: 1730971961, i: 16 }), t: Long('1') },
      optimeWritten: { ts: Timestamp({ t: 1730971961, i: 16 }), t: Long('1') },
      optimeDate: ISODate('2024-11-07T09:32:41.000Z'),
      optimeDurableDate: ISODate('2024-11-07T09:32:41.000Z'),
      optimeWrittenDate: ISODate('2024-11-07T09:32:41.000Z'),
      lastAppliedWallTime: ISODate('2024-11-07T09:32:41.787Z'),
      lastDurableWallTime: ISODate('2024-11-07T09:32:41.787Z'),
      lastWrittenWallTime: ISODate('2024-11-07T09:32:41.787Z'),
      lastHeartbeat: ISODate('2024-11-07T09:32:49.721Z'),
      lastHeartbeatRecv: ISODate('2024-11-07T09:32:50.722Z'),
      pingMs: Long('0'),
      lastHeartbeatMessage: '',
      syncSourceHost: '172.22.0.4:27017',
      syncSourceId: 0,
      infoMessage: '',
      configVersion: 1,
      configTerm: 1
    }
  ],
  ok: 1,
  '$clusterTime': {
    clusterTime: Timestamp({ t: 1730971961, i: 16 }),
    signature: {
      hash: Binary.createFromBase64('AAAAAAAAAAAAAAAAAAAAAAAAAAA=', 0),
      keyId: Long('0')
    }
  },
  operationTime: Timestamp({ t: 1730971961, i: 16 })
}
```


## Mongo-express for Mongodb UI
We sometimes feel comfortable with UI where we will get database information in interactive mode. Mongo express helps us to connect with mongodb. Let's setup mongo-express container.
```
docker run -d --network mongodb-replica-set-docker-compose_mongo-replica-cluster -e ME_CONFIG_MONGODB_SERVER=mongoreplica01,mongoreplica02,mongoreplica03 --name mongo-express-replica-set -p 8081:8081 mongo-express
```
Now you might access 'http://localhost:8081/' to visualize the mongo express UI.



## Reset the compose
Docker compose provides commands to remove all container with volumes and networks.
```
docker compose -f mongo-replica-set.yaml down -v
```
The flag "-v" indicates docker to remove resources such as volumes and networks.

<i>Note: host mapped volumes (e.g. ./volume/replica01) wouldn't be deleted though '-v' flag is provided in command. You have to delete manaually with root previleges.</i> 

### How to delete volume manually in linux system
```
$ sudo su
$ rm -rf volume/
```