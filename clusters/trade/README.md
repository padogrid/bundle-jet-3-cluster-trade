# Jet Cluster: trade
  
The `trade` cluster bundle installs a Jet cluster that includes the `build_app` script to clone and build the `realtime-trade-monitor` project maintained by cangencer. You must install this bundle in a Jet workspace.

## Required Software

You must first install Kafka to run the `realtime-trade-monitor` app. To follow the steps shown in this document, you need to set the KAFKA_HOME environment variable to the Kafka installation directory. For example,

```console
export KAFKA_HOME=~/Work/products/kafka_2.12-2.3.1
export PATH=$PATH:$KAFKA_HOME
```

## Installing and Building the `trade` Bundle

To build it, run the `build_app` script from inside the `bin_sh` directory as follows:

```console
cd bin_sh
./build_app
```

## Running Cluster

Make sure to add at least one member before starting the cluster.

```console
# Switch to the trade cluster
switch_cluster trade

# Add 2 members
add_member
add_member

# Start cluster
start_cluster
```

## Running apps

The `build_app` clones the repo in the cluster directory. For instructions, please see README.md in the cloned project directory as follows.

```console
switch_cluster trade
cd realtime-trade-monitor
cat README.md
```

The following shows the order in which the commands described in the above `realtime-trade-monitor/README.md` file should be executed.

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

## Tips

```console
# To clear the 'trade' topic, change the retention perioid to 1 sec
kafka-topics.sh --zookeeper localhost:2181 --alter --topic trades --config retention.ms=1000

# Wait 1+ sec and change back the retention period (5 days)
kafka-topics.sh --zookeeper localhost:2181 --alter --topic trades --config retention.ms=432000000
```
