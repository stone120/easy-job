---
title: IHS plugin-cfg best pratices
date: 2017-11-22 20:18:43
tags: [IBM]
categories: 技术
---

reference from [IBM web](http://www-01.ibm.com/support/docview.wss?uid=swg21318463)

best pratices for IBM plugin-cfg.xml , include: 
> MaxConnections 
> LoadBalanceWeight 
> ConnectTimeout 
> ServerIOTimeout 

<!-- more -->

# Question
In the web server plug-in, what do the LoadBalanceWeight, MaxConnections, ConnectTimeout, ServerIOTimeout, RetryInterval, IgnoreAffinityRequests, and GetDWLMTable options mean and what are the recommended settings for these options?

What affect does Session Affinity have?

How are connections handled during plug-in fail-over?

What is the effect of using more than one web server child process?
Answer

# Answer
To understand how load balancing works in the web server plug-in, see Understanding IBM HTTP Server plug-in Load Balancing in a clustered environment.

To understand how fail-over works in the web server plug-in, see Understanding HTTP plug-in failover in a clustered environment.


## LoadBalanceWeight 
is a starting "weight". The value is dynamically changed by the plug-in during runtime. The "weight" of a server (or clone) is lowered each time a request is assigned to that clone. When all weights for all servers drop to 0 or below, the plug-in has to readjust all of the weights so that they are above 0. Using a starting value of only 2 (default), means that the weights will get to 0 very quickly and the plug-in will constantly be readjusting the weights. Therefore, It is recommended to start with a higher LoadBalanceWeight. The IBM WebSphere Application Server administrative console will allow a value up to 20 for this. However, it is certainly possible to manually edit the plugin-cfg.xml file and specify some other value for LoadBalanceWeight that is higher than 20.

Note: At runtime, the LoadBalanceWeight of each appserver in a cluster are normalized by their highest common factor. For example, 100, 90, 80 have a common factor of 10. So, these configured weights would be divided by 10 at runtime, resulting in actual starting weights of only 10, 9, 8. Setting all clones to the same starting LoadBalanceWeight (for example: 20, 20, 20) will result in an actual starting weight of only 1 for each, because of normalization. So, it is recommended to set the weight of at least one of the clones to be off by a value of 1. For example, if there are 3 clones, you might choose the starting LoadBalanceWeights to be: 20, 20, 19. After normalization the weights will be unchanged.
Recommended values = all clones the same, except one clone off by one (for example: 20, 20, 19)


## MaxConnections 
is used to gauge when a server is "starting to become overwhelmed". It is not used to determine when to fail-over (mark the server "down"). When a request is sent from the plug-in to the WAS appserver, it is called a "PendingRequest", until the response comes back. If the application running in WebSphere Application Server is handling requests quickly, each request will only be PENDING for a very short time. So, under ideal conditions, MaxConnections is not needed and therefore the default is (-1) meaning unlimited. However, sometimes an application may start to become overwhelmed and the application may not be able to handle the requests as quickly. Consequently Pending Requests start to build up. MaxConnections can be used to put a limit on the number of PENDING requests per server. When the MaxConnections limit is reached, the plug-in will stop sending requests to that appserver, but it is not marked down. The optimal value for MaxConnections will depend on how quickly the application and appserver respond to each request. If normal responses are returned in less than one second, it may be appropriate to set a low value for MaxConnections, like 20 or so. However, if it normally takes several seconds to get a response from the application, then it would be prudent to use a higher value for MaxConnections, like 100. Please note that if the MaxConnections limit has been reached the plug-in will not send ANY more requests to that server until responses come back for the current PENDING requests, and the pendingRequests count drops back down below the MaxConnections limit.
Recommended value = 20 - 100 depending on application response times

*Best Practices: with MaxConnections="-1" use LogLevel="Stats" to monitor the pendingRequests numbers in the plug-in log, under normal conditions. Then, choose a value for MaxConnections that is significantly higher than the highest number shown in the log. This method will help you to determine a MaxConnections value that is right for your specific environment.


## ConnectTimeout means 
"how long should the plug-in wait when trying to open a socket to the Application Server"? If there are streams already open and available to the Application Server, the plug-in will use one of those. However, sometimes the plug-in needs to open a new stream to the Application Server. That should not take very long, so the value for ConnectTimeout should be very small. A ConnectTimeout value of 0 means never time-out. In that case, the time-out is left up to the OS TCP layer, which is NOT ideal. It is much better to specify a small positive number (like 5 seconds).
Recommended value = 5


## ServerIOTimeout 
means "how long should the plug-in wait for a response from the application". After the socket is opened, the plug-in sends the request to the Application Server. The application processes the request and a response is sent back to the client, through the plug-in. How long should that take? What is reasonable, based on the application? There is no single correct answer here. It depends on the application. If the application is very quick to respond, then you can use a lower value for ServerIOTimeout. However, if the application requires more time to process the request (maybe to retrieve data from a database), then you should use a higher number for ServerIOTimeout. Using a value of 0 means that the plug-in will NOT time-out the request. This is often NOT ideal. A positive value means that the plug-in will NOT mark the appserver down after a ServerIOTimeout pops. So, if you want the plug-in to continue sending requests to the timed-out appserver, use a positive value. On the other hand, a negative value means that the plug-in WILL mark the appserver down after a ServerIOTimeout pops. So, if you want the plug-in to immediately mark the appserver down and fail-over to another appserver in the same cluster, use a negative value.
Recommended value = -900 (that is negative 900)

Note: The ability to use a negative ServerIOTimeout value was introduced in plug-in apar PK72097.

*Best Practices: use traces to determine the amount of time it takes for your application to respond to requests under normal conditions. Be sure to include the longest running requests that take the most time to respond. Then choose a value for ServerIOTimeout that is much larger (2X or 3X or more) than the longest response time. This method will ensure that your ServerIOTimeout is high enough to allow adequate time for the application to respond normally. Make it a negative value so that if the ServerIOTimeout pops, the plug-in will immediately mark the server down, and retry the request to a different appserver.


## ServerIOTimeoutRetry 
can be used to decrease the number of retries after ServerIOTimeout has fired. By default, the Plug-in will try a request equal to the number of members in the cluster. For example, if the cluster has four members, and there is a ServerIOTimeout, the Plug-in will try it a second time. If the retry fails with ServerIOTimeout fired, then it will try a third time, and a fourth time if needed. But after four attempts, the Plug-in will give up and stop retrying. If you want to override this default behavior and reduce the number of retries after ServerIOTimeout, you can set ServerIOTimeoutRetry to a value that is less than the number of members in the cluster.
Recommended value = -1 (this is the default)

Note: This property was introduced by Plug-in apar PM70559.


RetryInterval is the time that the plug-in will wait before trying again to use an appserver that was marked down. The optimal value for RetryInterval depends on the number of appservers in the cluster, and the value used for ServerIOTimeout. You can use the following formula to determine the maximum RetryInterval value for your plug-in config:

(number of appservers in cluster - 1) x (absolute ServerIOTimeout) - 1

For example, if there are two appservers in the cluster, and the value of ServerIOTimeout is -900, then the maximum RetryInterval setting would be:
(2 - 1) x (900) - 1 = 899 seconds or less

Another example, if there are four appservers in the cluster, and the value of ServerIOTimeout is -900, then the maximum RetryInterval setting would be:
(4 - 1) x (900) -1 = 2699 seconds or less

Warning: Setting RetryInterval to a value higher than the recommended maximum, based on the formula above, can lead to an undesirable situation where all of the appservers in the cluster may be marked down simultaneously resulting in all requests temporarily failing.
Recommended value = 60 (this is the default)


Affinity requests are requests that contain a session cookie (ie. JSESSIONID). The session cookie is set by the Session Manager in WAS to ensure that all subsequent requests from the same client return to the same app server in the cluster. The session cookie contains the clone ID (or partition ID) of that specific app server. The web server Plug-in will look for the session cookie and use the clone ID to send the request to that specific WAS app server. An affinity request is not load balanced.

In the plug-in config there is a property called IgnoreAffinityRequests. This property determines if affinity requests will affect the load balance weights, or not. The default value for IgnoreAffinityRequests is True, which means that affinity requests will not have any affect on the load balance weights. This is best for environments where there are no, or few, affinity requests. On the other hand, for an environment with a lot of affinity requests, it may be better to set IgnoreAffinityRequests to False so that the load balance weights will accurately reflect the effect of having a lot of affinity requests.


Fail-over occurs when the plug-in marks a cluster member appserver (or clone) as "down", and then sends the pending requests to other members of the same cluster. This can happen if the plug-in is unable to open a new connection to the appserver within the ConnectTimeout. Or fail-over can happen if the plug-in has already sent the request to the appserver, but does not receive a response from the application within ServerIOTimeout. When the plug-in marks a cluster member appserver "down", it will handle the PENDING requests in one of two ways: Prior to plug-in apar PM12112, the plug-in would send all of the pending requests to the very next appserver in the cluster. However, after plug-in apar PM12112, the plug-in will randomly send the pending requests to any of the available appservers in the cluster. While the appserver is marked "down" the plug-in will no longer send any requests to it. After RetryInterval the plug-in will check to see if that appserver can be used successfully again. If so, the "down" flag will be removed and the appserver will be used again.

Note: By default, the number of attempts to handle a request is limited by the number of appservers in the cluster. For example, if there are only two appservers in the cluster, and the request fails once, the plug-in will only attempt that request one more time (total of two attempts). Or another example, if there are five appservers in the same cluster, and the request fails once, then the plug-in will attempt to retry that same request up to four more times (total of five attempts). That number includes retries sent to the same appserver (session affinity), or attempts sent to different appservers (fail-over).

Update: The plug-in apar PM70559 introduced a new setting called "ServerIOTimeoutRetry" that can be used to control the number of retries due to ServerIOTimeout.


If Memory-to-Memory (M2M) session replication is enabled in WebSphere Application Server, then the GetDWLMTable setting in the plug-in config must be changed to "true". Memory-to-Memory replication uses partition IDs rather than clone IDs. This can lead to broken session affinity if GetDWLMTable is set to false (which is the default). So it is very important to set GetDWLMTable="true" whenever using M2M in WebSphere Application Server.
Recommendation = GetDWLMTable="true" whenever M2M is used in WebSphere Application Server.


Each web server child process loads a separate instance of the web server plug-in. And multiple running instances of the web server plug-in do not share information with each other. For example if the IBM HTTP Server web server is configured to start 3 child processes (StartServers 3), then there will be 3 instances of the web server plug-in running (one for each IBM HTTP Server child process). The dynamically changing LoadBalanceWeight of each cluster member is not shared between the plug-in instances. So, in one instance of the plug-in "member1" might be considered UP with a weight of 5, when in another instance of the plug-in "member1" might be considered DOWN and unusable. This can result in possibly different behaviors depending on which child process / plug-in instance handles each incoming request. For this reason it is recommended that you should configure the web server to use only a few web server child processes with many threads on each. See Tuning IBM HTTP Server to maximize the number of client connections to WebSphere Application Server.

If you choose to use more than one web server child process, keep in mind that the plug-in settings are handled on a per instance basis. For example, MaxConnections means the number of pending requests that will be allowed on that server, for each plug-in instance. If MaxConnections = 20, and there are 3 web server child processes (3 plug-in instances), then each instance will allow 20 pending connections to that Application Server for a total of 60 pending connections.

# config
[IBM web](http://www-01.ibm.com/support/docview.wss?uid=swg21460889)
path: websphere application server -> server11 -> web服务器插件属性(页面最下) -