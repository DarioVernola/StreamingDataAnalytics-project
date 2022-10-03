# KsqlDB - Tomato Pick
## Set up
**Start up the infrastructure**

Bring up the entire stack by running:

```
docker-compose up -d
```
**Open the ksqlDB CLI**
```
docker exec -it ksqldb-cli ksql http://ksqldb-server:8088
```

**Configure it for this demo**

Run the following commands that allows to rerun the queries always from the beginning of the data streams. This is **not to use in production!** It is for debugging/educational purpose only.

```
SET 'auto.offset.reset' = 'earliest';
```

## Lab
Please refer to the [EPL version](https://github.com/DarioVernola/StreamingDataAnalytics-project/blob/main/epl_tomatopick/readme.md) .

### Recap
> Suppose you want to monitor a swarm of autonomous drones that pick tomatoes in a field with a stream processing engine. A data stream includes the information about the trees to pick tomatoes from. The most important ones are the id of the tree, its GPS position, the kind of tomatoes it produces, and the time interval in which the drones can pick the tomatoes from it. Another stream tracks each drone. It contains an event every time it picks a tomato. Each event reports the id of the drone, the id of the tree it is picking tomatoes from, and its GPS position.

**Modeling**
> Propose a model for the described streams.
 ```
 CREATE STREAM TreeToPickTomatoesFrom_STREAM (treeID INT, position VARCHAR, type VARCHAR, ts VARCHAR, pick_start VARCHAR, pick_end VARCHAR)
  WITH (
  kafka_topic='RoboticArm_topic', 
  value_format='json', 
  partitions=1,
  timestamp='ts',
  timestamp_format='yyyy-MM-dd''T''HH:mm:ssZ');
  
  CREATE STREAM DronePicking_STREAM (droneID INT, servicedTreeID INT, position VARCHAR, ts VARCHAR)
  WITH (
  kafka_topic='RoboticArm_topic', 
  value_format='json', 
  partitions=1,
  timestamp='ts',
  timestamp_format='yyyy-MM-dd''T''HH:mm:ssZ');
 ```
 
 **Data stream Generation**
```
INSERT INTO TreeToPickTomatoesFrom_STREAM (treeID, position, type, ts, pick_start, pick_end) VALUES (1, 'A1', 'cherry', '2021-10-23T06:00:01+0200', '2021-10-23T06:00:01+0200', '2021-10-23T06:40:01+0200');
INSERT INTO DronePicking_STREAM (droneID, servicedTreeID, position, ts) VALUES (1, 1, 'A1', '2021-10-23T06:00:20+0200');
INSERT INTO TreeToPickTomatoesFrom_STREAM (treeID, position, type, ts, pick_start, pick_end) VALUES (2, 'B2', 'yellow', '2021-10-23T06:00:20+0200', '2021-10-23T06:00:20+0200', '2021-10-23T07:10:01+0200');
INSERT INTO DronePicking_STREAM (droneID, servicedTreeID, position, ts) VALUES (2, 2, 'B2', '2021-10-23T06:00:40+0200');
INSERT INTO DronePicking_STREAM (droneID, servicedTreeID, position, ts) VALUES (1, 1, 'A1', '2021-10-23T06:01:00+0200');
INSERT INTO TreeToPickTomatoesFrom_STREAM (treeID, position, type, ts, pick_start, pick_end) VALUES (1, 'A1', 'cherry', '2021-10-23T06:01:20+0200', '2021-10-23T06:01:20+0200', '2021-10-23T06:8:20+0200');
INSERT INTO DronePicking_STREAM (droneID, servicedTreeID, position, ts) VALUES (1, 1, 'A1', '2021-10-23T06:01:20+0200');
INSERT INTO DronePicking_STREAM (droneID, servicedTreeID, position, ts) VALUES (2, 2, 'B2', '2021-10-23T06:01:20+0200');
INSERT INTO DronePicking_STREAM (droneID, servicedTreeID, position, ts) VALUES (2, 2, 'B2', '2021-10-23T06:06:20+0200');
```
```
CREATE STREAM RequestFullfilled_STREAM AS SELECT TPTF.treeID AS tID, DP.droneID AS dID, TPTF.type AS tom_type
FROM TreeToPickTomatoesFrom_STREAM AS TPTF JOIN DronePicking_STREAM AS DP ON TPTF.treeID = DP.servicedTreeID
EMIT CHANGES;
```

```
SELECT tom_type, count(*)
FROM RequestFullfilled_STREAM WINDOW HOPPING(SIZE 5 MINUTES, ADVANCE BY 20 SECONDS)
GROUP BY tom_type
EMIT CHANGES;

```
