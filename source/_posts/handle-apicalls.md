---
title: How to Handle API Calls Like a Pro?
date: 2020-05-30
tags: web development, API
index_img: /images/thumbnail/api-call.png
---
API is probably one of the terminologies in software development that people heard the most. API stands for Application Programming Interface which can be considered as the portal between front-end and back-end of an application. Any user action within the app, such as click, drag-and-drop, hover can trigger a HTTP request containing the API command. The back-end will then fetch data from database according to the API query parameter, and send back to browser for display. This interaction between client and server is also known as RESTful service, which is a typical web service model used in most of modern web developments nowadays. 

In front-end development, the developer will face a lot of API invokes, especially in API-driven application where loading a page, or a single user action could trigger multiple simultaneous API calls. What are the elegant ways of invoking multiple API calls becomes the question for every front-end developer. 

### A simple user case
Imagine we have a page that has a long list of customers, and each customer is identified by a customer id. What we want to do is scanning through the customer list, compare the status of each customer with the system settings(i.e., the default customer status settings). If the status of the customer matches with the system settings, append that customer to another list on the page. 
```text
Customer List

customer_id: 18273
customer_id: 28391
customer_id: 04829
customer_id: 83920
... 
```
To get the status of each customer, we use `GETCUSTOMERSTATUS` API command with the query parameter of its customer id. Consider sending HTTP request containing the following data payload:
```json 
{
    COMMAND: "GETCUSTOMERSTATUS",
    PROPERTY: {
        CUSTOMERID: customer_id
    }
}
```
What are the best ways of approaching this?

### Intuition 
Probably the most intuitive and straightforward approach is to use a `for loop`. We iterate through each customer item, call `GETCUSTOMERSTATUS` API command, get the response containing the status of current customer, compare with the system customer settings. If matches, append this customer to another list. Pretty simple logics, right? 

However, this approach may cause the browser to become unresponsive. The underlying issue is that `for loop` is synchronous, but the API request-response cycle is **asynchronous**. To elaborate on that, consider the following code:
```javascript 
this.processCustomerList = function() {
    var self = this;
    for(let i = 0; i < customerList.length; i++) {
        var req = {
            method: 'POST',
            url: 'http://test.com',
            data: {
                COMMAND: 'GETCUSTOMERSTATUS',
                PROPERTY: {
                    CUSTOMERID: customerList[i].customer_id
                }
            }
        }
        $http(req).then(function(response) {
            if(self.matchWithSystemSetting(response.data.status)) {
                self.systemUsers.push(response.data.customer_id);
            }
        });
    }
};
```
In the code above, the `for loop` iterates through the customer list, calls `GETCUSTOMERSTATUS` API command, get the status of customer, and append to `systemUsers` if matches with the system customer settings. Since the API call request-response cycle is asynchronous, the for loop will continue to proceed the next iterations regardless of whether not the current response callback has been resolved. 

Imagine the length of the customer list is 100, then by the time all the for loop iterations are finished, there may be 100 simultaneous HTTP requests(depends on the idle time set for `Keep-Alive` in HTTP header) queued in one persistent TCP connection, waiting to be resolved at the server side.

The problem with having too many requests queued in one single TCP connection is that nowadays a well-developed backend service normally has a limit of maximum number of simultaneous API request to accept, in order to prevent malicious attacks such as **DDoS attack**. This is the same idea taken by many popular APIs including `YouTube Data API v3`. YouTube Data API sets [daily quota usage](https://developers.google.com/youtube/v3/getting-started#quota) for each access token to prevent customer from oversending API requests to the video server in which the server performances may be significantly affected due to the overwhelming traffic. 

Therefore, once the number of API requests in TCP conenction exceeds the traffic threshold of backend service, the server will refuse to accept the requests and browser will throw out the timeout error. Now your application becomes unresponsive.

### A working solution
Now we know the underlying issue with using for loop. How about we process API command one at a time, meaning we don't send out the next API request until the current one is resolved? Yes, this will work and only needs a simple **callback function** to do the job.

Consider the following refactored code:
```javascript 
this.count = 0;
this.processCustomerList = function() {
    var self = this;
    if(self.count === this.customerList.length) return;
    var req = {
        method: 'POST',
        url: 'http://test.com',
        data: {
            COMMAND: 'GETCUSTOMERSTATUS',
            PROPERTY: {
                CUSTOMERID: customerList[self.count].customer_id
            }
        }
    }
    $http(req).then(function(response) {
        if(self.matchWithSystemSetting(response.data.status)) {
            self.systemUsers.push(response.data.customer_id);
        }
        self.count++:
        self.processCustomerList();
    });
}
```
We have gotten rid of the for loop, and replaced it with recursion. We also have a new variable `count` which keeps track of the base case of the recursive loop. The method `processCustomerList()` is recursively invoked within the `$http` promise response. This will make sure the next API request won't be sent out until the response for the current request has returned.

By doing so, the traffic burden on the backend server is majorly relieved. However, the tradeoff is also obvious. Since API requests are sent sequentially, the runtime of the whole scanning process may become significant especially when the customer list goes large.

This solution works, but there gotta be some improvements.

### A working, and more elegant solution
The best scanerio would be being able to sent simultaneous API requests, but not exceeding the limit of the backend server. That requires us to process customer list in batch, and in each batch, multiple API requests are sent out. When the current batch is fully resolved, that is, browser has received and processed each response of the API request within the batch, the next batch of the customer list gets proceeded. How should we approach this?

In this scenario, we need to keep track of the status of each asynchronous API cycle. A handy way would be using **Promise**. As one of the main approaches to tackle asynchronous actions in JavaScript, Promise is a more elegant way comparing to callback which may leads developer to "callback hell". Especially in this case, using callback would be hard to manage the code flow.

By backing up with Promsie, now for each batch of the customer list, we can encapsulate each API request action as a Promise Object, and push them into an array. After the iteration of the current batch is done, we use `Promise.all()` to wait on each Promise Object in the array to be resolved, and proceed to the next batch of the list in promise callback.

```javascript 
this.batch = 10;
this.count = 0;
this.makeRequestParams = function(index) {
    return {
        method: 'POST',
        url: 'http://test.com',
        data: {
            COMMAND: 'GETCUSTOMERSTATUS',
            PROPERTY: {
                CUSTOMERID: self.customerList[index].customer_id
            }
        }
    }
}
this.processCustomerList = function() {
    var self = this;
    var bucket = this.batch;
    var promises = [];
    if(self.count === self.customerList.length) return;
    if(self.customerList.length - self.count < bucket) {
        bucket = self.customerList.length - self.count;
    }
    for(let i = 0; i < bucket; i++) {
        promises.push(new Promise(function(resolve) {
            $http(self.makeRequestParams(self.count + i)).then(function(response) {
                if(self.matchWithSystemSetting(response.data.status)) {
                    self.systemUsers.push(response.data.customer_id);
                }
                resolve();
            });
        }));
    }
    Promise.all(promises).then(function() {
        self.count += bucket;
        self.processCustomerList();
    });
}
```
In this case, we assume the maximum limit of API requests is 10. Thus for each recursive iteration, we push 10 HTTP requests into an promise array to resolve afterwards. What `Promise.all()` does is that it waits on each `Promise Object` in array being resolved. And only after that, the count gets incremented and we invoke the recursion to proceed the next bucket of list items. 

Since each time we send maximum number of simultaneous API requests that backend server can handle, we doesn't waste any bandwidth of connection. Compare to sending API request sequentially, the runtime was significantly reduced. 

### Conclusion 
Working with API commands has become one of the daily routines for web developers. Invoking API requests in correct way makes client side highly responsive in terms of retrieving and rendering user data, thus enhance user experience. 

There are also other more handy techniques and methods of handling multiple API calls in different cases, such as `subscribe` and `mergeMap` provided by Angular, but all share the same nature of callback and promise. Using callback and promise wisely will not only solve most of the issues derived from asynchronous actions, but also make the code more readable and managable.

I hope you found this article useful, and stay tuned for more of my own experiences and thoughts on web development!



