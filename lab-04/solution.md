# Lab 04: Solutions
Here you can find all the solutions to the challenges in Lab 04. You can also find a dashboard demonstrating the result of completing all the tasks here [this dashboard.](https://onenr.io/0EjOzvMqDR6)

### Challenge 1
The variable is added to the `WHERE` clause: `AND userAgentName in ({{browser}})`

```
SELECT count(*) as 'Page views' from PageView where appName='WebPortal' SINCE 1 day ago WHERE deviceType in ({{viewDevice}}) AND userAgentName in ({{browser}})
```

### Challenge 2
You can choose your own variable identifier, we've gone for `chosenCity`.

The query in the variable is:
```
SELECT uniques(city,10000) from PageView where appName='WebPortal' since 1 day ago
```

The filter was aded to the query: `AND city in ({{chosenCity}})`

```
SELECT count(*) as 'Page views' from PageView where appName='WebPortal' SINCE 1 day ago WHERE deviceType in ({{viewDevice}}) AND userAgentName in ({{browser}}) AND city in ({{chosenCity}})
```

### Challenge 3
This is a freeform exercise, have fun!
