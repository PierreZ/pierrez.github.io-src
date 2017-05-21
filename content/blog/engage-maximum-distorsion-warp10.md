+++
date = "2017-05-21"
title = "Engage maximum warp speed in time series analysis with WarpScript"
tags = ["warp10","time series"]
description = "Why Warp10 and WarpScript are the best tools to analyze time series data"
draft = true
+++

We, at Metrics, are working everyday with [Warp10 Platform](http://warp10.io), an open source Time Series database. You may not know it because it's not as famous as [Prometheus](https://prometheus.io/) or [InfluxDB](https://docs.influxdata.com/influxdb/), but Warp10 is the most **powerful and generic solution** to store and analyze sensor data. It's the **core** of Metrics, and many internal teams from OVH are using us to monitor their infrastructure. As a result, we are handling a pretty nice traffic 24/7/365, as you can see below:


{{< tweet 860792423647203331 >}}

[Yes, that's more than **4M datapoints/sec** on our frontends](https://twitter.com/OvhMetrics/status/860792423647203331).

{{< tweet 860791293693317121 >}}
[And more than **5M commits/sec** on HBase, our storage layer](https://twitter.com/OvhMetrics/status/860791293693317121).

Not only Warp10 allows us the reach an unbelievable scalability, it comes with his own language called **WarpScript**, to manipulate and perform heavy time series analysis. Before digging into the need of a new language, let's talk a bit about the need of time series analysis.

# What is a time serie ?

**A time serie, or sensor data, are simply sequences of measurements over time**. The definition is quite generic, because many things can be represented as a time serie:

- The evolution of the stock exchange or a bank account
- The number of calls on a webserver
- The fuel consumption of a car
- The time to insert a value into a database
- the time a customer is taking to register on your website
- The heart rate of a person measured through a smartwatch

From an historical point of view, time series appeared shortly after the creation of the Web, to help engineers monitor the networks. It quickly expands to also monitors servers. Monitoring is **fundamental** to running a stable service. With the right monitoring system, you can have **insights** and **KPIs** about your service:

* **Analysis of long-term trend**
    * How fast is my database growing?
    * At what speed my number of active user accounts grows?

* **The comparison over time**
    * My queries run faster with the new version of my library? Is my site slower than last week?

* **Alerts**
    * Trigger alerts based on advanced queries

* **Displaying data through dashboards** 
    * Dashboards help answer basic questions on the service, and in particular the 4 indispensable metrics: **latency, traffic, errors and service saturation**

* **The possibility of designing retrospective** 
 * Our latency is doubling, what's going on? 

# Time series are complicated to handle

But storage, retrieval and analysis of time series cannot be done through standard relational databases. Generally, highly scalable databases are used to support volumetry. For example, the **300,000 Airbus A380 sensors on board can generate an average of 16 TB of data per flight**. On a smaller scale, **a single sensor that measures every second generates 31.5 million values per year**. Handling time series at scale is difficult, because you're running into advanced distributed systems issues, such as:

- **ingestion scalability**, ie. how to absorb all the datapoints 24/7.
- **query scalability**, ie. how to query in a raisonnable amount of time.
- **delete capability**, ie. how to handle deletes without stopping ingestion and query.

Frustration with existing open source monitoring tools like **Nagios** and **Ganglia** is why the giants created their own tools – **Google has Borgmon** and **Facebook has Claspin**, just to name two. They are closed sources, but the idea of treating time-series data as a data source for generating alerts is now accessible to everyone thanks to the **former Googlers who decided to rewrite Borgmon** outside Google.


# Why another time series database?

Now the time series ecosystem is bigger than ever, here's a short list of what you can find to handle time series data: 

* [DalmatinerDB](https://dalmatiner.io/)
* [InfluxDB](https://influxdata.com/)
* [Prometheus](https://prometheus.io/)
* [Riak TS](http://basho.com/products/riak-ts/)
* [OpenTSDB](http://opentsdb.net/)
* [KairosDB](https://kairosdb.github.io/)
* [Elasticsearch](https://www.elastic.co/products/elasticsearch)
* [Druid](http://druid.io/)
* [Blueflood](http://blueflood.io/)
* [Graphite (Whisper)](https://graphiteapp.org/)

And then there's **Warp10**. The difference is quite simple. Warp10 is **a platform** whereas all the time series listed above are **stores**. This is game changing, for multiples reasons.

* **Security-first design**

    > Security is mandatory for data access and sharing job's results. But in most of the above databases, security access is not handled by default. With Warp10, security is handled with crypto tokens similar to [Macaroons](https://research.google.com/pubs/pub41892.html). 

* **High level analysis capabilities** 

    > Using classical time series database, **high level analysis must be done elsewhere**, with R, Spark, Flink, Python, or whatever languages or frameworks that you want to use. Using Warp10, you can just **submit your script** and voilà!

* **server-side calculation**

    > Algorithms are resource heavy. Whatever they're using CPU, ram, disk and network, you'll hit **limitations** on your personal computer. Can you really aggregate and analyze one year of data from thousands of sensors on your laptop? Maybe, but what if you're submitting the job from a mobile? To be **scalable**, analysis must be done **server-side**.

# Meet WarpScript

![image](/img/engage-maximum-distorsion-warp10/warpscript.png)


* **Server Side Analysis**

    > Yes, you'll be able to run that awesome query that is fetching millions of datapoints.

* **Dataflow language**

    > WarpScript is really easy to code, because the ouput of a function is used as the input of the following one. Coding became logical. First you need to **fetch** your points. Then **applying some downsampling**. Then **aggregate**. These 3 sentences are translated into **3 lines of WarpScript**. 

* **Rich programming QL**

    > WarpScript is coming with more than 800 functions, ready to use. Patterns detections, outliers, rolling average, FFT, are only functions to apply on data.

* **Geo-Fencing capabilities**

    > Both **space** (location) and **time** are considered **first class citizens**. Complex searches like “**find all the sensors active during last Monday in the perimeter delimited by this geo-fencing polygon**” can be done without involving expensive joins between separate time series for the same source.


* **Unified Language** 

    > WarpScript can be used in batch mode, or in real-time.

# Geez, give me an example!

```warpscript
[ $token ‘temperature’ {} NOW 1 h ] FETCH        // Fetching all values

[ SWAP bucketizer.max	0 1 m 0 ] BUCKETIZE     // Get max value for each minute

[ SWAP mapper.round 0 0 0 ] MAPPER  		// Round to nearest decimal

[ SWAP [ 'buildingID' ] reducer.max ] REDUCE    // aggregate according to labels 
```

# What about a more complex example?

you're still here? Good, let's have a more complex example. Let's say that I want to to do some patterns recognition. With WarpScript, it's only a 2 functions calls:

* **PATTERNS** is generating a list of motifs.
* **PATTERNDETECTION** is running the list of motifs on all the time series you have.

Let's take an example. Here's a cosinus with an increasing amplitude:

![image](/img/engage-maximum-distorsion-warp10/pattern.png)

I want to **detect the green part** of the time series, because I know that my service is crashing when I have that kind of load. Let's call **PATTERNS** and **PATTERNDETECTION** and see the result! 

![image](/img/engage-maximum-distorsion-warp10/patterns.png)



As you can, PATTERNDETECTION is working even with the increasing amplitude! You can discover this example by yourself by using [Quantum](https://home.cityzendata.net/quantum/preview/#/plot/TkVXR1RTICdjb3MnIFJFTkFNRQoxIDEwODAKPCUgRFVQICdpJyBTVE9SRSBEVVAgMiAqIFBJICogMzYwIC8gQ09TICRpICogTmFOIE5hTiBOYU4gNCBST0xMIEFERFZBTFVFICU+IEZPUgoKWyBTV0FQIGJ1Y2tldGl6ZXIubGFzdCAxMDgwIDEgMCBdIEJVQ0tFVElaRSAnY29zJyBTVE9SRQoKTkVXR1RTICdwYXR0ZXJuLnRvLmRldGVjdCcgUkVOQU1FCjIwMCAzNzAKPCUgIERVUCAnaScgU1RPUkUgRFVQIDIgKiBQSSAqIDM2MCAvIENPUyAkaSAqIE5hTiBOYU4gTmFOIDQgUk9MTCBBRERWQUxVRSAlPiBGT1IKClsgU1dBUCBidWNrZXRpemVyLmxhc3QgMjE2MCAxIDAgXSBCVUNLRVRJWkUgJ3BhdHRlcm4udG8uZGV0ZWN0JyBTVE9SRQoKLy8gQ3JlYXRlIFBhdHRlcm4KMzIgJ3dpbmRvd1NpemUnIFNUT1JFCjggJ3BhdHRlcm5MZW5ndGgnIFNUT1JFCjE2ICdxdWFudGl6YXRpb25TY2FsZScgU1RPUkUKCiRwYXR0ZXJuLnRvLmRldGVjdCAwIEdFVCAkd2luZG93U2l6ZSAkcGF0dGVybkxlbmd0aCAkcXVhbnRpemF0aW9uU2NhbGUgUEFUVEVSTlMgVkFMVUVTICdwYXR0ZXJucycgU1RPUkUKCiRjb3MgJHBhdHRlcm5zICR3aW5kb3dTaXplICRwYXR0ZXJuTGVuZ3RoICRxdWFudGl6YXRpb25TY2FsZSAgUEFUVEVSTkRFVEVDVElPTiAnY29zLmRldGVjdGlvbicgUkVOQU1FICdjb3MuZGV0ZWN0aW9uJyBTVE9SRQoKJGNvcy5kZXRlY3Rpb24KLy8gTGV0J3MgY3JlYXRlIGEgZ3RzIGZvciBlYWNoIHRyaXAKMTAgICAgICAgLy8gIFF1aWV0IHBlcmlvZAo1ICAgICAgICAgLy8gTWluIG51bWJlciBvZiB2YWx1ZXMKJ3N1YlBhdHRlcm4nICAvLyBMYWJlbApUSU1FU1BMSVQKCiRjb3M=/eyJ1cmwiOiJodHRwczovL3dhcnAuY2l0eXplbmRhdGEubmV0L2FwaS92MCIsImhlYWRlck5hbWUiOiJYLUNpdHl6ZW5EYXRhIn0=), the official web-based IDE for WarpScript. **You need to switch X-axis scale to Timestamp in order to see the courbe**.