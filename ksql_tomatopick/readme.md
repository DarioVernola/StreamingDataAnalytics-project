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

