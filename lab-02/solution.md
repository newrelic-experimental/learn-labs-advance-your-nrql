# Lab 02: Solutions
Here you can find all the solutions to the challenges in Lab 02. You can also find these on [this dashboard.](https://onenr.io/0kjnv0lOXjo)

## Challenge #1
In this solution we have chosen a threshold of 75% above which we consider the host to be low on memory.

Note that the `LIMIT 5` has been changed to `LIMIT MAX` as we need to consider all hosts (up to the top 2000 at least).

```
SELECT latest(memoryUsedPercent) as '% Used' FROM (FROM SystemSample SELECT latest(memoryUsedPercent) as memoryUsedPercent LIMIT max FACET hostname) WHERE memoryUsedPercent >=75 FACET hostname SINCE 2 hours AGO
```

## Challenge #2
To display a count of hosts with low memory we used the `count()` aggregation function and remove the `FACET` in the outer query:

```
SELECT count(*) as 'Hosts low on memory' FROM (FROM SystemSample SELECT latest(memoryUsedPercent) as memoryUsedPercent LIMIT max FACET hostname) WHERE memoryUsedPercent >=75  SINCE 2 hours AGO
```


## Challenge #3
Timeseries needs to be added to both the inner and outer query. Here we extend the time window to 1 week and set the timeseries bucket size to 1 hour.

```
SELECT count(*) as 'Hosts low on memory' FROM (FROM SystemSample SELECT latest(memoryUsedPercent) as memoryUsedPercent LIMIT max FACET hostname TIMESERIES 1 hour) WHERE memoryUsedPercent >=75 TIMESERIES 1 HOUR SINCE 1 WEEK AGO
```

> It is possible to have the timeseries in only the inner query, this would allow you to, for instance, count how many time buckets a given host was breaching a threshold.