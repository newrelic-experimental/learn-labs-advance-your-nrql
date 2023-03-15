# Lab 03: Solutions
Here you can find all the solutions to the challenges in Lab 03. You can also find these on [this dashboard.](https://onenr.io/0nQxlZ9ndQV)

## Challenge 1
To solve this challenge simply replace the `average()` function with `percentile()`
```
SELECT count(*) from Transaction where appName='WebPortal' AND duration > (SELECT percentile(duration,99) FROM Transaction where appName='WebPortal')
```

## Challenge 2
In order to solve this we need to derive the 99th percentile duration which can be achived with `percentile(duration,99)`. We also need to facet by `name` to get a list, and a `LIMIT max` always helps to ensure we're not clipping our results:

```
SELECT count(*) FROM Transaction WHERE appName='WebPortal' AND duration > (SELECT percentile(duration,99) FROM Transaction WHERE appName='WebPortal') FACET name LIMIT max
```

## Challenge 3
To change the period over which the sub query calculates the 99th percentile add a `SINCE 1 WEEK AGO` clause to the inner query:

```
SELECT count(*) FROM Transaction WHERE appName='WebPortal' AND duration > (SELECT percentile(duration,99) FROM Transaction WHERE appName='WebPortal' SINCE 1 WEEK AGO) FACET name LIMIT max
```

## Challenge 4
The solution is to switch the `count(*)` for `average(duration)` or a `percentile(duration,x)` for your choice. Here we display a few:
 
```
SELECT average(duration), percentile(duration,50,90,99) FROM Transaction WHERE appName='WebPortal' AND duration > (SELECT percentile(duration,99) FROM Transaction WHERE appName='WebPortal' SINCE 1 WEEK AGO) FACET name LIMIT max
```

## Challenge 5
We need to add the `TIMESERIES` clause and also extend our outer query time window to one day:

With `average()`:
```
SELECT average(duration)-(SELECT average(duration) AS 'Delta' FROM Transaction WHERE appName='WebPortal' SINCE 1 week ago) FROM Transaction WHERE appName='WebPortal' SINCE 1 DAY AGO TIMESERIES
```

With `percentile()`:
```
SELECT percentile(duration,99)-(SELECT percentile(duration,99) AS 'Delta' FROM Transaction WHERE appName='WebPortal' SINCE 1 week ago) FROM Transaction WHERE appName='WebPortal' SINCE 1 DAY AGO TIMESERIES
```

## Challenge #6
There are a few  things to change in this query.

 Firstly we need to extend both the inner sub query and outer query to scan for the last day of data: `SINCE 24 hours AGO`

 Next we need to limit the results to just those visiting the contact page: `WHERE name LIKE '%contact.jsp'`

 Finally the question asked for number of customers, this can be gathered with `uniqueCount(session) as 'Customers'`

```
SELECT uniqueCount(session) as 'Customers' FROM PageView WHERE session  IN (SELECT uniques(session,10000) FROM PageView WHERE appName='WebPortal' AND name LIKE '%/confirmation.jsp' LIMIT MAX SINCE 24 hours AGO) SINCE 24 hours AGO WHERE name LIKE '%contact.jsp'
```