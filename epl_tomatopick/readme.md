# Tomato pick example 
## Resources

- [espertech](https://www.espertech.com/)
- [EPL documentation](http://esper.espertech.com/release-8.7.0/reference-esper/html/index.html)
- [EPL playground](http://esper-epl-tryout.appspot.com/epltryout/mainform.html)

## The problem

> Suppose you want to monitor a swarm of autonomous drones that pick tomatoes in a field with a stream processing engine. 
A data stream includes the information about the trees to pick tomatoes from. The most important ones are the id of the tree, its GPS position, the kind of tomatoes it produces, and the time interval in which the drones can pick the tomatoes from it. Another stream tracks each drone. It contains an event every time it picks a tomato. Each event reports the id of the drone, the id of the tree it is picking tomatoes from, and its GPS position.

### Modeling
> Formalize in EPL the schema of the two streams.

Reading the description carefully it is possible to derive the following schemes:


```  
create schema TreeToPickTomatoesFrom(
treeID int,
position string,
type string,
pick_start long,
pick_end long
);

create schema DronePicking(
droneID int,
servicedTreeID int,
position string
);
```

An instance of the **TreeToPickTomatoesFrom** schema translates to a command given to the drones to pick tomatoes from it, whereas an instance of the **DronePicking** schema is issued once a certain drone has picked one tomato.

The position attribute is represented as a string as it will not be used in this example.

For now these attributes will suffice. We will modify them in the future.

### Data stream generation
In order to test the queries that will be later presented we will use the following data stream. It is suggested to modify it using different configurations to test wether your queries will work in different settings.

```  
TreeToPickTomatoesFrom = {treeID = 1 , position = 'A1', type = 'cherry', pick_start = 1, pick_end = 40}
t=t.plus(20 seconds)
DronePicking = {droneID = 1, servicedTreeID = 1, position = 'A1'}
TreeToPickTomatoesFrom = {treeID = 2 , position = 'B2', type = 'yellow', pick_start = 3, pick_end = 70}
t=t.plus(20 seconds)
DronePicking = {droneID = 2, servicedTreeID = 2, position = 'A1'}

t = t.plus(20 seconds)

DronePicking = {droneID = 1, servicedTreeID = 1, position = 'A1'}

t=t.plus(20 seconds) 

DronePicking = {droneID = 1, servicedTreeID = 1, position = 'A1'}
DronePicking = {droneID = 2, servicedTreeID = 2, position = 'B2'}
t=t.plus(20 seconds)



DronePicking = {droneID = 2, servicedTreeID = 2, position = 'B2'}
t=t.plus(20 seconds)
```

### Assignment

> Tell every 20 seconds how many tomatoes the drones picked in the last 5 minutes grouped by their kind.

```  
@Name('QInsert')
INSERT INTO requestFullfilled
SELECT b.droneID as dID, a.type as type
FROM pattern[
every a = TreeToPickTomatoesFrom() 
-> 
every b = DronePicking(
b.servicedTreeID = a.treeID
)];
```

```  
@Name('QFinal')
SELECT type, count(*)
FROM requestFullfilled.win:time(5 minutes)
GROUP BY type
output  all every 20 seconds;
```




