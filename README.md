# Jet Cluster: trade
  
The `trade` cluster bundle installs a Jet cluster that includes the `build_app` script to clone and build the `realtime-trade-monitor` project maintained at the GitHub URL shown below.

[https://github.com/cangencer/realtime-trade-monitor.git](https://github.com/cangencer/realtime-trade-monitor.git)

## Installing Bundle

```console
install_bundle -download bundle-jet-3-cluster-trade
```

## Use Case

This use case demonstrates Jet aggregating Kafka streamed trade data in real time. It includes a trade blotter UI for monitoring the trade aggregations executed in the Jet cluster.

![Jet Trade Diagram](/images/jet-trade.png)

## Required Software

You must first install Kafka to run the `realtime-trade-monitor` app. To follow the steps shown in this document, you need to set the KAFKA_HOME environment variable to the Kafka installation directory. For example,

```console
export KAFKA_HOME=~/Work/products/kafka_2.12-2.3.1
export PATH=$PATH:$KAFKA_HOME/bin
```

## Installing and Building the `trade` Bundle

To build it, run the `build_app` script in the `trade` cluster's `bin_sh` directory as follows:

```console
# Switch to the trade cluster
switch_cluster trade
cd bin_sh

# Build app
./build_app
```

## Running Cluster

Make sure to add at least one member before starting the cluster.

```console
# Add 2 members
add_member; add_member

# Start cluster
start_cluster
```

## Running Apps

The `build_app` script clones the `realtime-trade-monitor` repo in the cluster directory.

```console
switch_cluster trade
cd realtime-trade-monitor
```

The following lists the commands described in the `realtime-trade-monitor` repo in the order they should be executed.

```console
# 1. Start Zookeeper and Kafka
zookeeper-server-start.sh $KAFKA_HOME/config/zookeeper.properties 
kafka-server-start.sh $KAFKA_HOME/config/server.properties

# 2. Create the 'trades' topic to which we'll publish trades
kafka-topics.sh --create --replication-factor 1 --partitions 4 --topic trades --zookeeper localhost:2181

# 3. Load stock symbols into the 'symbols' map in Jet
java -jar trade-queries/target/trade-queries-1.0-SNAPSHOT.jar load-symbols

# 4. Ingest (drain) all trade objects in the 'trades' topic in Kafka via Jet
java -jar trade-queries/target/trade-queries-1.0-SNAPSHOT.jar ingest-trades localhost:9092

# 5. Aggregate trades from the 'trades' topic in Kafka via Jet
java -jar trade-queries/target/trade-queries-1.0-SNAPSHOT.jar aggregate-query localhost:9092

# 6. Publish data into the 'trades' topic in Kafka
java -jar trade-producer/target/trade-producer-1.0-SNAPSHOT.jar localhost:9092 10

# 7. Start web server
java -jar webapp/target/webapp-1.0-SNAPSHOT.jar
```

**Trade Blotter URL:** [http://localhost:9000](http://localhost:9000)

## Tearing Down

```console
# 1. Ctrl-C servers running in the foreground:
#    Javalin web server (7),  trade-producter (6), Kafka server (1), Zookeeper (1)

# 2. Stop Jet cluster
stop_cluster
```

## Tips

```console
# To clear the 'trade' topic, change the retention perioid to 1 sec
kafka-topics.sh --zookeeper localhost:2181 --alter --topic trades --config retention.ms=1000

# Wait 1+ sec and change back the retention period (5 days)
kafka-topics.sh --zookeeper localhost:2181 --alter --topic trades --config retention.ms=432000000
```
