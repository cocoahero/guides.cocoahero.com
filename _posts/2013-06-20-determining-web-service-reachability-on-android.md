---
layout: post
title: "Determining Web Service Reachability on Android"
author: "Jonathan Baker"
summary: "Most mobile applications these days talk to a web service. Making sure you can reach your destination prior to performing any requests is good practice. In this guide, we will take a look at a thorough approach to do just that."
tags:
    - Android
    - Java
    - Reachability
    - Web Service
    - Mobile
---
It is pretty common for a mobile application to communicate with some sort of web service. Generally, this process is triggered by some sort of user interaction. If the user taps a button labeled "refresh news feed", he or she expects exactly that to happen. What happens if the device isn't connected to the Internet? Whether it is because of airplane mode or bad cellular signal, it is important to let the user know that what they requested won't happen and more importantly, why.

The solution to this is to do what is known as a reachability test before performing the actual web service request. If the test fails, we can notify the user immediately via an alert dialog or by some other means. We could also disable functionality to prevent the user from trying again. If we skip this step, the request would be attempted and would eventually fail wasting not only computational time, but also the user's.

Determining reachability to a web service involves checking three conditions.

1. Is the device connected to a network?
2. Is the device able to resolve the hostname of your service?
3. Is the device able to open a socket connection to your service? 

Each one of these conditions checks a specific stage of a request. Only when all three are met can you be justified in attempting the request. There's no point in trying to upload a user's photo if the device can't resolve "my.service.com".

### Step One: Network Availability

The first step is to determine if the device is connected to a network. On Android, this can be determined by using the [ConnectivityManager][1] service. You retrieve an instance of this class by asking for it on a [Context][5], normally the current [Activity][6]. Once you have a ConnectivityManager, you can ask it for the currently active network information by calling [getActiveNetworkInfo()][3]. This will return a [NetworkInfo][2] object, or `null` if the device is currently not connected to any type of network. Finally, you can call [NetworkInfo#isConnected()][4] to ensure the device is connected to the active network. Below is what this looks like in code.

<script src="https://gist.github.com/cocoahero/063a3cbb4230d5f4f126.js"></script>

### Step Two: Hostname Resolution

Generally speaking, you connect to your web service using some sort of hostname. For example, "my.webservice.com" or "api.heroku.com". At some point in the request, the system has to translate this hostname into an IP address in order to open the connection. This is done by using a [DNS (Domain Name System)][7] server located somewhere on the network. Usually this server is owned by an ISP and will have no trouble resolving the hostname correctly. However, there is a chance that the device is on a restricted or private network with a custom DNS solution that may not be able (or allowed) to resolve your hostname. This is what the second step in our reachability check will determine.

Apart of the Java Standard Library, and thus Android, is a class called [InetAddress][8]. An instance of this class represents an IP address and is created by calling one of the class factory methods. In our case, we want to use [InetAddress#getByName(String)][9] and pass in our web service's hostname. This method will either return an instance of `InetAddress` or throw an `UnknownHostException`. If this exception is thrown, we know that the DNS server was unable to resolve the hostname and should abort our request.

<script src="https://gist.github.com/cocoahero/017dbc8895fa00d6d3aa.js"></script>

### Step Three: Establishing a Connection

We've now determined the device is on a network and can resolve the hostname. You may be wondering what could possibly stop a request from happening at this point. Simply put, many times users may be using your app at work, which means they are probably on a restricted network. There could be [packet shapers][10], firewalls, and content filters standing between your app and the web service. The easiest way to determine if this will be a problem is to try and open a socket connection.

By using the `InetAddress` instance we were returned in the previous step, we can open a simple TCP socket to our web service on a given port. For most web services this will be port 80, or 443 if you utilize HTTPS. [TCP sockets][11] are another feature of the Java Standard Library that is available on Android. The code shown below demonstrates how to open a socket.

<script src="https://gist.github.com/cocoahero/e1bc6bf5a1a888d62d2b.js"></script>

### The Main Thread and You

The last step is to simply combine all of the previous three into an easy to use API. Now if you have done some networking related development on Android before, you may wondering about doing all of this on the main thread. For those of you who don't know, if you target at least Honeycomb (API 11) and you do any sort of network activity on the main thread, **your application will crash** with a [NetworkOnMainThreadException][12]. While this seems rather strict and rude, it is for a good reason. Blocking the main thread with network activity will make the user interface unresponsive and will result in a poor user experience. Simply put, you should **never** perform network activity on the main thread. Just say no.

Android provides a handy class that will suit us perfectly for solving our threading woes: [AsyncTask][13]. By performing all of our checks within the `#doInBackground()` method, we will stay off the main thread and avoid the strict exception. Now because our API is now asynchronous, we need a way to communicate the results back to the caller. This can be done with a simple callback interface, as shown below.

<script src="https://gist.github.com/cocoahero/01a24c4fcccf40dcdd99.js?file=ReachabilityTest.java"></script>

To use your newly made reachability test, simply create a new anonymous instance and provide a callback. The methods within the callback will be called on the main thread so you can modify the user interface safely.

<script src="https://gist.github.com/cocoahero/01a24c4fcccf40dcdd99.js?file=MyActivity.java"></script>

### Summary

While it looks like it may not be necessary, performing a reachability test as soon as possible will allow you to notify the user and let them know their limitations. It is better to be upfront about it than to try and fail with a potentially obscure error code the user doesn't understand. 

Overall, the test does not take much time computationally. The active network information is always up to date as it is maintained by the Android OS so retrieving the status is trivial. Performing the DNS query will be relatively quick and will cause the result to be cached for subsequent requests saving time in the future. The longest step by far is the socket connection, but will be no more than the timeout value set, and is usually much less.

[1]: http://developer.android.com/reference/android/net/ConnectivityManager.html
[2]: http://developer.android.com/reference/android/net/NetworkInfo.html
[3]: http://developer.android.com/reference/android/net/ConnectivityManager.html#getActiveNetworkInfo()
[4]: http://developer.android.com/reference/android/net/NetworkInfo.html#isConnected()
[5]: http://developer.android.com/reference/android/content/Context.html
[6]: http://developer.android.com/reference/android/app/Activity.html
[7]: http://en.wikipedia.org/wiki/Domain_Name_System
[8]: http://developer.android.com/reference/java/net/InetAddress.html
[9]: http://developer.android.com/reference/java/net/InetAddress.html#getByName(java.lang.String)
[10]: http://en.wikipedia.org/wiki/Traffic_shaping
[11]: http://developer.android.com/reference/java/net/Socket.html
[12]: http://developer.android.com/reference/android/os/NetworkOnMainThreadException.html
[13]: http://developer.android.com/reference/android/os/AsyncTask.html