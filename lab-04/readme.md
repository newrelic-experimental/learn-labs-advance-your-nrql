# Lab 04: Building interactive dashboards
In this lab you will learn how to build more flexible interactive dashboards by leveraging the **variable templating** features.

## Pre-requisites
To complete this lab you need access to the Demotron v2 account in New Relic. This can be accessed via your NRU learning account.

## Filtering dashboards
People create dashboards to curate the information they want to see or share with their team. It's common for the same dashboard to be rebuilt multiple times for different contexts. For instance you might build a dashboard that surfaces data about a specific application's performance, golden signals and business metrics and then you then might replicate that entire dashboard for all of your other similar applications. This can be toilsome and require many dashboards to be created to cater for different filters required. 

Dashboard template variables allow you to avoid some of this replication by adding options to the Dashboard that a user can select which changes what data is displayed. The fact that you can add multiple variables allows you to create quite complicated versatile dashboards.

### ✏️ Task #1: Setup
To work through this lab you'll need a dashboard! Go to New Relic and create a new dashboard in the Demotron v2 account. Set the dashboard visibility to **private** and name it something appropriate.

Add a billboard widget to the dashboard using the following query:
```
SELECT count(*) AS 'Page views' FROM PageView WHERE appName='WebPortal' SINCE 1 day ago
```

We recommend that you duplicate widgets and edit them as you work through the tasks and challenges.


### ✏️ Task #2: Create your first variable
The widget we added to the dashboard shows the number of page views over the last day. Our customers can use the app on desktop or mobile, lets add a template variable to allow us to filter by device type.

- Click the Add Variable button 
- Set the "Name to use in queries" field to `viewDevice`. This is the variable identifier. It can be any string starting with a letter and no spaces. We'll use this later in our query.
- Set the "Display name" field to "Device". This is the 'friendly' label that will be shown along side our variable.
- Set the Type to "List". This type allows us to specify a list of values to be selected, we'll explore the others later.
- Enter the values "Desktop", "Mobile" and "Tablet" in the values field, separated by a comma: `Desktop,Mobile,Tablet`
- Enable the "multi-value" display option
- For default value, choose "Select All"
- Set the output format to "String"
- Save changes and observe the variable "Device" has appeared above the dashboard


![Device variable setup](images/device-var.png)


### ✏️ Task #3: Wire up the variable to query
You've successfully created the "viewDevice" variable but you're not doing anything with it yet. Lets connect the variable to the query driving the Page Views widget.

Duplicate the billboard widget and adjust the query by adding the following to the end: `WHERE deviceType in ({{viewDevice}})`

```
SELECT count(*) AS 'Page views' FROM PageView where appName = 'WebPortal' SINCE 1 day ago WHERE deviceType in ({{viewDevice}})
```

This `WHERE` clause filters the results by the `deviceType` attribute. We reference the variable `viewDevice` by enclosing the variable identifier in double curly braces: `{{viewDevice}}`. 

Notice we use `WHERE attr IN (..)` rather than `WHERE attr = '...'`. This is because the variable can be a list of multiple values. Its best practice to always use this structure with template variables.

### ✏️ Challenge #1
It would be good to be able to filter the data broken down by browser user agent so that we can see how popular each browser is. 

Create a new template variable called `'browser'` with the following values: `Safari,Microsoft Edge,Chrome,Android Browser,IE,IE Mobile,Firefox`.

Duplicate the previous dashboard widget and wire up the query so that that the query also filters by this new variable.

> Hint: You need to filter the `userAgentName` attribute using your new variable.


### ✏️ Task #4: Dynamically generated filters
In the previous challenge you used a list variable to filter by user agent. This is good for when you have a fixed list of known values to choose from but its not un-common that you instead wish to generate that list from your data. Good news! The 'query' type variable allows you to do just this. 

Lets update the variable we just created to use a query instead of a hard coded list of values.

- Click the edit icon on the dashboard (it looks like a pencil, top right)
- Click the down arrow on the Browser variable and choose 'Edit'
- Change the type to 'Query'
- Ensure the Demotron V2 account is selected (You can see here that you can also source the data from another account)
- Enter the query: `select uniques(userAgentName) from PageView where appName='WebPortal'` and press enter.
- In the default drop down choose 'Select All'
- Click 'Save'
- Click 'Done editing' (top right of Dashboard)

Try changing the variable selections and observe how the list is the same as the hard coded list. Also, take a look at the query for the widget by clicking on the ellipsis and choosing 'View query'. Notice how the variable is replaced by the values.

**Important:** When using a query to provide a list of values you **must** use the [`uniques(attr,limit)`](https://docs.newrelic.com/docs/query-your-data/nrql-new-relic-query-language/get-started/nrql-syntax-clauses-functions/#func-uniques) function as we did here. If you think there may be a lot of unique values then ensure to set the limit parameter of the `uniques(atrr,limit)` function. You can set the limit up to 10,000.

e.g. `uniques(hostname,10000)`

### ✏️ Challenge #2
Our application has an international audience and so another way we might want to view the data is by geography. Create a new variable that allows filtering by `city` create a new widget that has a query filtered by this new variable.

### Incomplete data sets
You may have noticed that when all the variables are set to 'Select All' the widgets filtering on city do not display the same values as the other widgets. Why is this?

Our filters work by including all records with an attribute IN the list of values provided by our variable. But what happens if there is no value? In this case the record is filtered out, it's not in the list after all. Depending on your use case you may choose to keep these null records. You can do this by adding an `OR attribute IS NULL` clause. For example for our city filter:

```
From: ...AND city in ({{chosenCity}}) 

To:   ...AND ( city in ({{chosenCity}}) OR city IS NULL )
```

You *could* even go a step further and make the inclusion of null cities an option chosen by the user via another *additional* variable: (There is an example of this on the solution dashboard.)

```
( 
    ({{includeNullCity}}='yes' and city is null) OR (city in ({{chosenCity}}))
)
```

### ✏️ Task #5: Numeric template variables
So far we've filtered the Dashboard by string values. Some of our NRQL functions take numeric parameters. One good example is the `percentile()` function. We can use template variables here to to allow the user to switch between percentile values for the whole dashboard. Lets see how this works:

- Create a new 'List' variable called `chosenPercentile`
- For the list values specify: `50,75,90,99`
- Do not enable 'multi-value'
- Choose a default of your choice.
- Set the output type to `Number`. This is important!

Now create a new Dashboard widget with the following query:

```
SELECT percentile(duration,{{chosenPercentile}}) AS 'Page views' FROM PageView WHERE appName='WebPortal' AND deviceType in ({{viewDevice}}) SINCE 1 day AGO TIMESERIES FACET deviceType
```

The important part of this query is: `percentile(duration,{{chosenPercentile}})`. You can see that just like with the filtering we supply the value for our percentile threshold using `{{variableName}}`

Try changing the percentile value and observe how the chart changes. You can apply this technique to any function that takes a numeric parameter.


### ✏️ Task #6: User chosen identifiers
We've seen how to filter queries based on variables and how to supply numeric values to functions. There is one more way we can affect the query and that is by using *identifier variables* to change the query itself. Consider a dashboard showing performance data for an application. You may want to view the data faceted  by device type, and then by city, and then by user agent. Without template variables you would have to create widgets for each of these cases, but variables allow us to make this user selectable.

The previous chart we built is faceted by deviceType. Lets upgrade it so that we can facet it by an attribute of our choice.

First create a new variable:

- Call it `chosenFacet`
- Set the type as List and add some attribute names, e.g. `deviceType,countryCode,userAgentName`
- Do not enable multi-value
- Select a default
- Select `Identifier` as the output format

Now edit the widget and amend the facet so that it uses the variable. 

```
SELECT percentile(duration,{{chosenPercentile}}) AS 'Page views' FROM PageView WHERE appName='WebPortal' AND deviceType in ({{viewDevice}}) SINCE 1 day AGO TIMESERIES FACET {{chosenFacet}}
```

Now when you change the value for facet in the variable the widget updates and the new facts are displayed.


### ✏️ Task #7: Textfield variables for user supplied input
All the variables we have used so far have been drop down lists. What if we wanted to be able to filter our dashboard by a string provided by the user. Well we can do that too! Lets see how we might filter our dashboard to a specific user.

Create another new variable:

- Call it `chosenUsername`
- Set the type as 'Text Field'
- Set the default value to `%`
- Set the output format to 'String'

Add a new widget to your dashboard with the following query:
```
SELECT max(PotentialCartGrandTotal), sum(duration) FROM PageView WHERE appName='WebPortal' AND username LIKE {{chosenUsername}} FACET username 
```

Notice in this query we have added a `LIKE` filter on the variable: `username LIKE {{chosenUsername}}`
Try typing a name in the email. You need to specify the `%` wild card for partial matches, tye `%aol.com` for example.

This is very powerful. we can create Dashboards that filter to user supplied content without having to edit the dashboard!


### ✏️ Task #8: Dynamically extracted list values
In this final task we'll explore how to generate variable lists with partial values extracted from the data. Consider a case where you want to filter the data on the dashboard to a set of page groups. The variable list will contain the page groups and when you choose a group all those records matching will be displayed.

Create another new variable:

- Call it 'chosenPageGroup`
- Choose 'Query' type and enter the following query: `SELECT uniques(aparse(browserTransactionName, '%/*/%')) as segment from PageView since 1 day ago`
- Enable multi value and set the default to select all.

Add a new widget to the dashboard with this query:
```
SELECT duration, browserTransactionName, session  FROM PageView  WHERE appName='WebPortal' AND aparse(browserTransactionName, '%/*/%') IN ({{chosenPageGroup}}) 
```

There's a number of things going on here, lets break it down.

Firstly we are extracting the 'page group' from the browserTransactionName field using `aparse()`. This is basically capturing the part after the first `/` and before the next `/`:

```
Input  : webportal.telco.nrdemo.com:80/static/services.jsp
Output : static
```

This generates a list of groups: `static`, `browse` and `purchase` which populates our variable. 

The query in the widget performs the same `aparse()` capture and filters it using `IN` in the `WHERE` clause: `aparse(browserTransactionName, '%/*/%') IN ({{chosenPageGroup}})`

You should find that choosing the page group from the drop down change the pages displayed in the widget to just those in the group selected.


### ✏️ Bonus Challenge #3
Put what you've learnt into practice! As an optional exercise build a dashboard containing three widgets representing different metrics for an application of your choice. Add two different template variables that are wired up to all of the widgets.

> Hint: Alternatively you could just tidy up the dashboard you created during this lab and wire up all the variables to the widgets. 

## Wrapping up
In this lab you've learnt how to make your dashboards interactive and more versatile by leveraging template variables. 
