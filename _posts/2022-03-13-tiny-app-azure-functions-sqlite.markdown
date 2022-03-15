---
layout: post
title:  "Building a micro web application using Azure Functions and SQLite"
permalink: "micro-web-application-azure-functions-sqlite"
date:   2022-03-13 20:37:44 -0400
image_path: ../uploads/2022-03-13-23-39-52.png
image_alt: Proof-of-concept application built to validate Azure Functions with SQLite database
description: Recently, I read about the utility of SQLite for web servers and was intrigued by the possibility of using SQLite and an Azure Function to host an effectively free web application. Since an Azure Functions app comes along with an Azure Files storage account by default, the Azure Function should be able to store and read an SQLite file. 
---

Recently, I read about [the utility of SQLite for web servers](https://news.ycombinator.com/item?id=26580614#:~:text=SQLite%20is%20not%20a%20toy%20database%20%7C%20Hacker%20News&text=%3E%20There%20is%20a%20popular%20opinion,snapshots%2C%20concurrent%20backups%2C%20etc.) and was intrigued by the
possibility of using SQLite and an Azure Function to host an effectively free web application. Since an Azure Functions app comes along with an Azure Files storage account by default, the Azure Function should be able to store and read an SQLite file. 

Researching into this concept, I found that it had been [tried in the past](https://www.codeproject.com/Articles/5312004/Building-a-Micro-Web-API-with-Azure-Functions-and) but **I was still curious about the scalability and performance of such a solution**. I also wanted to validate the possibility of hosting an entire application on an Azure Function rather than just an API, with the goal of offloading the frontend SPA-like processing from my old device. This is the project I built to achieve this objective.

![](../uploads/2022-03-13-23-39-52.png)

## Application summary

Personal Finance Tracker is a proof-of-concept application demonstrating the possibility of hosting an entire application within an Azure Functions application. The NodeJS Azure Function dynamically generates HTML for the interface, processes API requests for any CRUD actions, and stores data in an SQLite database file.

The application allows the manual entry of statements of accounts and shows a graph of the history of the total of the amounts in every account.

## Source code

The Azure Function app was scaffolded using Visual Studio Code's Azure extension. 5 HTTP Trigger functions with anonymous authorization were created. `GET /api/Home` generates and returns the HTML to be rendered by the browser. `POST /api/CreateAccount` and `DELETE /api/DeleteAccount` are REST API endpoints that allow the creation and deletion of an account (column). `POST /api/CreateStatements` and `DELETE /api/DeleteStatements` are also REST API endpoints that allow the creation and deletion of statements (row). A shared database access helper file is also created in the `Shared` folder.

*Note that Azure Functions are typically used as APIs, and have their routes prepended with `/api/`. However, since we are using Azure Functions in a manner counter to intended, the `/api/Home` route will return an HTML page.*

![](../uploads/2022-03-13-21-22-16.png)

The helper file `/Home/renderHTML.js` is responsible for generating the HTML file. Styles and scripts are included within the HTML since they cannot be referred to as files as this is not a typical web server. Additional configurations would have had to be done to serve these static files. This makes the `renderHTML.js` file somewhat busy, and an area for potential improvements.

The application makes use of [sqlite](https://www.npmjs.com/package/sqlite) and [sqlite3](https://www.npmjs.com/package/sqlite3) npm packages, as well as [ChartJS](https://www.chartjs.org/) and [Bootstrap](https://getbootstrap.com/) for the HTML interface. 

The database contains 2 tables: `accounts` represent the accounts identified by their title, and `statements` represent the amount in an account on a specific date. 

![](../uploads/2022-03-13-21-46-53.png)

## Deployment process

For this project, you will need the [Azure Functions Core Tools CLI](https://docs.microsoft.com/en-us/azure/azure-functions/functions-run-local?tabs=v4%2Cwindows%2Ccsharp%2Cportal%2Cbash) and the [Azure CLI](https://docs.microsoft.com/en-us/cli/azure/install-azure-cli).

If you want to deploy the application as a Windows Function App, keep reading. If you prefer to deploy as a Linux Function App, jump to the note in the Additional Notes section below.

0. Clone the source code of the application locally. Run `func start` and ensure the application is working locally.
1. Create an Azure Function resource in the Azure Portal, configured as a Consumption app with Windows as the OS. Take note of the name of the Function App you just created. This step will create the accompanying storage account which will store the SQLite database.
2. Deploy the application using the `func azure functionapp publish <FUNCTION_APP_NAME> --nozip`. This deploys the local source code to the Azure Function resource, and the `--nozip` options [turns off Run-From-Package mode](https://docs.microsoft.com/en-us/azure/azure-functions/functions-core-tools-reference?tabs=v2#func-azure-functionapp-publish). This will allow us to manually install npm packages in the Azure portal. 
3. From the Azure Portal, navigate to the Azure Function. In the left menu pane of the Azure Function, navigate to Console.

![](../uploads/2022-03-13-22-16-51.png)

4. Run `npm install sqlite sqlite3` in the console to install `sqlite` and `sqlite3` npm packages for the Azure Function. This may take a few minutes.
5. Try out the application! Navigate to `https://<FUNCTION_APP_NAME>.azurewebsites.net/api/Home` and test it out!

**NPM packages `sqlite` and `sqlite3` must be installed within the Azure portal to ensure that the native bindings used by `sqlite3` correspond to the OS version of the host machine running the Azure Functions**. Those packages only need to be installed once. Future deployments will only update the source code being used.

## Performance & benchmarking

My main concern with such a usage of SQLite on Azure Functions would be the performance. Since this is not an intended usage of Azure Functions, the file locking and concurrency of the SQLite file could slow down the response times of the Azure Function. Therefore, I benchmarked the response times under varying loads using Apache Benchmark. Note that this benchmarking was done on functions that were 'warmed-up' to avoid the response delay caused by a function cold-start.


`ab -n 100 -c 10 "https://FUNCTION_APP_NAME.azurewebsites.net/api/home"`

```
Connection Times (ms)
              min  mean[+/-sd] median   max
Connect:       75   96  11.2     94     152
Processing:   266  613 209.6    570    1435
Waiting:      242  582 209.2    541    1409
Total:        356  708 209.0    664    1524

Percentage of the requests served within a certain time (ms)
  50%    664
  66%    781
  75%    849
  80%    887
  90%    974
  95%   1068
  98%   1327
  99%   1524
 100%   1524 (longest request)
```

These results demonstrate the response times if 10 users concurrently made 10 requests each. With a mean of 0.7s and a longest time of 1.5s, we can see that these response times are decent.

As expected, increasing the concurrency count increases the response times. With a concurrency of 100 users, the application response times make for a poor user experience. 

`ab -n 100 -c 100 "https://FUNCTION_APP_NAME.azurewebsites.net/api/home"`

```
Connection Times (ms)
              min  mean[+/-sd] median   max
Connect:       89  233 166.7    208    1174
Processing:   310 5538 1167.8   5648    7071
Waiting:      250 5509 1164.6   5623    7037
Total:        399 5771 1184.4   5991    7350

Percentage of the requests served within a certain time (ms)
  50%   5991
  66%   6563
  75%   6657
  80%   6754
  90%   6964
  95%   7123
  98%   7280
  99%   7350
 100%   7350 (longest request)
```

These results show that the **SQLite-backed Azure Function has mean response times of 5.8s under a load of 100 concurrent users, which would result in a poor user experience**. 

For comparison's sake, I decided to benchmark an Azure Function API that uses CosmosDB as storage. Here are the results for 100 concurrent users:

`ab -n 100 -c 100 "https://<FUNCTION_APP_NAME>.com/api/getMeetingInfo?meetingId=meetingID"`
```
Connection Times (ms)
              min  mean[+/-sd] median   max
Connect:      140  187  15.5    188     227
Processing:   116  851 303.1    934    2657
Waiting:      114  850 303.9    933    2657
Total:        271 1038 304.4   1115    2812

Percentage of the requests served within a certain time (ms)
  50%   1115
  66%   1184
  75%   1246
  80%   1263
  90%   1312
  95%   1342
  98%   1357
  99%   2812
 100%   2812 (longest request)
```

As we can see, CosmosDB performs much better. CosmosDB-backed Azure Functions managed 1.0s mean response times with 100 concurrent users, compared to the 5.8s of the SQLite-backed Azure Functions used in this project. **Response times of an SQLite-backed Azure Function are about 5-6 times those of a CosmosDB-backed Azure Function.** 

**Cold starts are also much worse with an SQLite-backed Azure Function than a CosmosDB one.** My experience is that CosmosDB-backed functions have 4-6s response times for cold functions, compared to 10-15s response times for cold SQLite-backed functions. I would hypothesize that additional time is required to mount the storage to the Azure Function compute which isn't required when using an external database such as CosmosDB. Clearly, Azure does not intend for the file system to be used in this way.

## Closing thoughts

* **SQLite with Azure Functions is possible, but it is slow for 100 concurrent users** (~6s response times). It also aggravates the cold-start delay by a factor of 2-3x. Building an application with Azure Functions and SQLite would only make sense for a personal project with a handful of users, and even then, other options such as CosmosDB/Azure Table/Azure SQL should be considered.

* The **deployment of SQLite was complicated** because of the need to get correct native bindings for SQLite file access.  

* Creating an HTML file as a JavaScript template literal is not ideal since data from the backend JS needs to be copied over to and the frontend JS using JSON.parse() and goes against the concept of separation of concerns.

* If I were to rebuild this application, it would be as a static web application (React or HTML/JS), with Azure Functions providing REST API endpoints and CosmosDB providing storage. The development process would be simplified, more reliable, and officially supported by Azure Static Web Apps.

## Conclusion

This project was a proof-of-concept of SQLite on Azure Functions. While the concept is proven, the **poor performance of the solution is a limiting factor** for any serious use cases. This approach could only be considered for personal projects, but even then, **Azure Static Web Apps and Cosmos DB should be strongly considered** for their superior performance and better developer experience. 

## Additional Notes

Linux Function Apps do not provide read access to files after deployment, unlike Windows Function Apps. This means that Windows Function Apps would be a better candidate for an Azure Function/SQLite deployment since the db.sqlite3 could be downloaded and viewed at any time. However, Linux Function Apps allow remote build (with the `func azure functionapp publish <FUNCTION_APP_NAME> -b remote` command), which facilitates the deployment as the build process downloads the correct SQLite3 native bindings.

## Resources

[Building a Micro Web API with Azure Functions and SQLite](https://www.codeproject.com/Articles/5312004/Building-a-Micro-Web-API-with-Azure-Functions-and), [SQLite is not a toy database
](https://antonz.org/sqlite-is-not-a-toy-database/), [Deployment technologies in Azure Functions](https://docs.microsoft.com/en-us/azure/azure-functions/functions-deployment-technologies) 
