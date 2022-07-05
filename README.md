# Market Ticks Streaming and Analytics With GridGain and Confluent Kafka

Confluent Kafka and GridGain/Ignite Market Ticks Streaming and Processing Demo.

The demo shows how to process market ticks using Confluent Kafka as a streaming engine and GridGain/Ignite as an
in-memory database with persistence layer. Confluent and GridGain are hooked up together via GridGain Kafka Connect
integration. SQL is a primary language used by Confluent and GridGain for data processing.

## GridGain Cluster Setup and Launch

### Launching the Demo Cluster
* Download and decompress [GridGain Enterprise Edition 8.8.19 or later](https://www.gridgain.com/resources/download)
* Move `{gridgain}/libs/optional/control-center-agent` folder to `{gridgain}/libs`
* Navigate to `{gridgain}/bin` folder and start a server node(s): `./ignite.sh {demo_project_root}/src/main/resources/gridgain-cfg.xml`
* (Optional) Start several nodes

### Monitoring the Demo Cluster
* Sign up on `portal.gridgain.com` and go to Monitoring screen
* Take the UUID from your server log and use it to add your cluster to Control Center
* Now you have the cluster started and can monitor it!
* Activate the cluster by going to the Cluster tab, select the '...' menu to the right of the cluster name and click 'Activate'

### Create tables and seed data in GridGain
* Start `Bootstrapper` class from your IDE to create demo tables and preload initial data

## Confluent Platform and GridGain Setup

If Java 9+ is used by default then switch to Java 8 in a command line window to be used for Confluent
scripts. Use this command on Mac OS:
```bash
export JAVA_HOME="/Library/Java/JavaVirtualMachines/jdk1.8.0_191.jdk/Contents/Home/"
```

### Install Confluent and GridGain Connect
* Go to `{gridgain}/integration/gridgain-kafka-connect/` and execute `./copy-dependencies.sh` script from there
* [Download](https://www.confluent.io/get-started/?product=software) and extract Confluent Platform 7.7.1 or later
* Go to `{confluent}`
* Edit `etc/kafka/connect-standalone.properties` using your favourite text editor
* Make the last line: `plugin.path={gridgain}/integrations/gridgain-kafka-connect,{demo_project_root}/target/market-ticks-1.0-SNAPSHOT.jar`
* Save the file
* Edit `cfg/gridgain-sink.properties`
* Make sure `igniteCfg` points to your configuration file (the full path to `src/main/resources/gridgain-cfg.xml`)
* Save

### Prepare MarketOrder Source Connector
* Build this project's JAR
  [with Intellij IDEA](https://stackoverflow.com/questions/36303535/intellij-build-build-artifacts-deactivated)
  or from your favourite IDE or using Maven from the project directory;
```bash
mvn clean package
```

### Starting Confluent, Loading Source and Sink
1. Go to {confluent}
2. Start ZooKeeper: `bin/zookeeper-server-start etc/kafka/zookeeper.properties`
3. Start Kafka Server: `bin/kafka-server-start etc/kafka/server.properties`
4. Start the two Connectors: `bin/connect-standalone etc/kafka/connect-standalone.properties {demo_project_root}/cfg/gridgain-sink.properties {demo_project_root}/cfg/market-orders-source.properties`
5. (Optional) Start the Kafka Control Center: `bin/control-center-start etc/confluent-control-center/control-center-minimal.properties`. You will be able to access it at http://localhost:9021/

### Real-Time Analytics with GridGain

Events will be generated and arriving in the cluster in real-time. Run sample queries like those below for
the demo purpose:

Total spending per buyer;

```SQL
SELECT first_name, last_name, SUM(bid_price) as mp
FROM MarketOrder as m JOIN Buyer as b ON m.buyer_id = b.id
GROUP BY first_name, last_name ORDER BY mp DESC;
```

Cash Distribution (fraction of money spent per symbol);

```SQL
SELECT symbol, SUM(bid_price) as mp
FROM MarketOrder GROUP BY symbol ORDER BY mp DESC;
```

Restart GridGain cluster (activate with Control Center or the CMD tool if needed) and show that the day
is still in the cluster.