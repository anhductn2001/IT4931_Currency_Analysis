# Big Data System for Digital Currency Analysis

This system is a big data pipeline to analyze real-time prices of digital currencies, like bitcoin, litecoin, and etc. Considering large amounts of transactions happening every second, big data frameworks (HBase, Kafka, Spark, etc.) are used to build an efficient real-time pipeline to track currency prices.

## Architect

The major components are:

![](./readme-figs/architect.png)

* data-producer:
	* fetch real-time `LastTradePrice` from gdax API every second and buffer them into a kafka data-bus.
* data-storage:
	* persist/sink data from kafka into hbase for later analysis usage.
* data-stream:
	* process data stream in kafka including calculating average, open, close, low, and high by using Spark Streaming.
* publish and display:
	* queue up real-time data in redis and display a user dashboard with nodeJS.
 a minimal and flexible Node.js web application framework that provides a robust set of features for web and mobile applications

## Build Guide
See [this seperate markdown document](./build-guide.md)

![](./readme-figs/currency-analysis-demo.png)
