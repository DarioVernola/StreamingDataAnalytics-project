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

