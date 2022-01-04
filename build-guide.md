# Build guide - digital currency analysis system

## Step 1. Install `docker`

Refer to the official [installation page](https://docs.docker.com/v17.12/docker-for-mac/install/#download-docker-for-mac) to install docker.

```
# remove all docker containers
docker rm -f $(docker ps -a -q)
# remove one docker container
docker rm -f containerID
```

## Step 2. Install Kafka python API

```
pip3 install schedule kafka-python requests 
```


## Step 3. Add `data-producer.py` and `data-consumer.py`

```
# run zookeeper
docker run -d -p 2181:2181 -p 2888:2888 -p 3888:3888 --name zookeeper confluent/zookeeper
# run kafka
docker run -d -p 9092:9092 -e KAFKA_ADVERTISED_HOST_NAME=localhost -e KAFKA_ADVERTISED_PORT=9092 --name kafka --link zookeeper:zookeeper confluent/kafka
# start producer
python data-producer.py BTC-USD analyzer 127.0.0.1:9092
# start consumer
python data-consumer.py analyzer 127.0.0.1:9092
```

```

## Step 4. Use HBase interactively

```
# run hbase container
docker run -d -h myhbase -p 8080:8080 -p 8085:8085 -p 9090:9090 -p 9095:9095 -p 16000:16000 -p 16010:16010 -p 16201:16201 -p 16301:16301 --name hbase harisekhon/hbase
# start bash shell in the running container
docker exec -it hbase /bin/bash
# enter into hbase cli
hbase shell
```

```
# in hbase shell, execute the following commands to get familar with hbase cmds.
describe 't'

create 't1', {NAME=>'f1', VERSIONS=>5, BLOCKCACHE=>'false'}

describe 't1'

put 't1','r1','f1:c1','value1',1526200761
put 't1','r1','f1:c1','value2',1526200762
put 't1','r1','f1:c1','value3',1526200763
put 't1','r1','f1:c1','value4',1526200764
put 't1','r1','f1:c2','value5',1526200765
put 't1','r2','f1:c1','value6',1526200766
scan 't1'
get 't1', 'r1'
get 't1', 'r1', 'f1:c1'
get 't1', 'r1', {COLUMN=>'f1:c1', VERSIONS=>100}
get 't1', 'r1', {COLUMN=>'f1:c1', VERSIONS=>2}
delete 't1','r1','f1:c1', 1526200763
disable 't'
drop 't'
```

## Step 4. Add `data-storage-writer.py` `data-storage-reader.py`

```
# install happybase - a hbase python API
pip3 install happybase
```

Since a running hbase container already has a running zookeeper, just link a kafka container to the existing zookeeper.

```
docker run -d -p 9093:9093 -e KAFKA_ADVERTISED_HOST_NAME=localhost -e KAFKA_ADVERTISED_PORT=9093 --link hbase:zookeeper confluent/kafka
```

Add port number to `/etc/hosts`

```
# added for currency-analysis project
127.0.0.1   myhbase
```

```
# run data producer
python3 data-producer.py BTC-USD analyzer 127.0.0.1:9092
# sink data into hbase
python3 data-storage-writer.py analyzer 127.0.0.1:9092 crytocurrency myhbase
```

## Step 5. Add `data-stream.py` and `data-stream-test.py`

Install [Java JDK](https://apple.stackexchange.com/a/283872)

```
# test data-stream.py
python3 data-stream-test.py
```


```
# run spark-streaming job
spark-submit --packages org.apache.spark:spark-streaming-kafka-0-8_2.11:2.0.0 data-stream.py analyzer average-price 127.0.0.1:9092 5
# check kafka bus
python3 data-consumer.py average-price 127.0.0.1:9092
```


## Step 6. Encapsulate chart data in redis queue.

```
# run redis container
docker run -d -p 6379:6379 --name redis redis:alpine
# start redis publisher
python3 redis-publisher.py average-price 127.0.0.1:9092 price 127.0.0.1 6379
```

## Step 7. Add NodeJS to host real-time data and D3.js visualization

```
# init a node project
npm init --yes
# install dependencies
npm install socket.io express redis minimist bootstrap jquery nvd3 d3@3.5.17
# start node server and host at 3000
node index.js --redis_host=localhost --redis_port=6379 --redis_channel=price --port=3000
```

## Troubleshoot

```
pip3 install cython thriftpy
```