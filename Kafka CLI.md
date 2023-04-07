# Kafka CLI

## Topics

Da bi radili sa topics mormao da koristimo komandu <span style="color:#ff4433"> kafka-topics</span>

Bitne komande:

-  <span style="color:#ff4433">--create</span>
-  <span style="color:#ff4433">--list</span>
-  <span style="color:#ff4433">--delete</span>
-  <span style="color:#ff4433">--topic</span>
-  <span style="color:#ff4433">--bootstrap-server</span>
-  <span style="color:#ff4433">--partitions</span>
-  <span style="color:#ff4433">--replication-factor</span>

### Create topic

kafka-topics --create --topic NAME --bootstrap-server IP --partitions NUM --replication-factor NUM_BROKERS

```
kafka-topics --create --topic topic1 --bootstrap-server localhost:29092 --partitions 4 --replication-factor 1
```



### List topics

kafka-topics --list --bootstrap-server IP

```
kafka-topics --list --bootstrap-server localhost:29092
```



### Topic details

kafka-topics --describe --topic NAME --bootstrap-server localhost:29092

```
kafka-topics --describe --topic topic1 --bootstrap-server localhost:29092
```



## Producer

Koristimo base komandu <span style="color:#ff4433"> kafka-console-producer</span>

Bitne komande:

- <span style="color:#ff4433">--broker-list </span>
- <span style="color:#ff4433">--topic</span>
- <span style="color:#ff4433">--property</span>

kafka-console-producer --broker-list IP --topic NAME

```
kafka-console-produer --broker-list localhost:29092 --topic topic1
```

Kada ovako saljemo podatke u kafku necemo imate key za to moramo koristiti dodatne komande 

kafka-console-produer --broker-list IP --topic NAME --property parse.key=true --property key.separator=SEPARATOR

```
kafka-console-produer --broker-list IP --topic NAME --property parse.key=true --property key.separator=:
```

Sada format poruke koji moramo da pisemo je key:value ako nemamo : baca exce



## Consumer

Koristimo base komandu<span style="color:#ff4433"> kafka-console-consumer</span>

- <span style="color:#ff4433">--bootstrap-server</span>
- <span style="color:#ff4433">--topic</span>
- <span style="color:#ff4433">--property</span>

 kafka-console-consumer --bootstrap-server IP --topic NAME

```
kafka-console-consumer --bootstrap-server localhost:29092 --topic topic1
```

--property print.timestamp=true

--property print.key=true

--property print.value=true