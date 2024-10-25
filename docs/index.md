# Welcome to Kafka Lab!

The goal of this material is to get you started with Apache Kafka. You will be watching a series of short videos from Confluent, a company founded by the same people who created Kafka. Along with the videos, you will get the chance of experimenting with the concepts using PWS (our very own distributed server for Kafka). Running the commands should be very quick. Invest some time in understanding what is going on and why outputs are in a certain way. We will dive deeper into each of these topics in class. 



## Login to PWS 

All the steps below need to be executed from within PWS. So to get started, open your terminal and ssh into the machine. 

!!! Tip "Information for accessing PWS can be found in the [syllabus](https://docs.google.com/document/d/1NNElqfBCUZsK_BgwQZvnPne-38ntbniQEq3OPMKLPgs/edit#heading=h.tavkzf56hnn)"


## Set up

Let's start by exporting some env vars. You need to do this every time you ssh into the machine (or add them to your `.bashrc` in c0, up to you). 

``` 
export KAFKA_PATH='/opt/kafka/bin' 
export BOOTSTRAP_SERVER='cs0.minerva.local:9092' 
```

## Inspecting the cluster 

<iframe width="560" height="315" src="https://www.youtube.com/embed/qu96DFXtbG4?si=cxH_fHKbDlMSiUCx" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>

To start, run: 
```
${KAFKA_PATH}/kafka-metadata-quorum.sh --bootstrap-server $BOOTSTRAP_SERVER describe --status 
```

This is the first step to make sure that the system is working correctly. Your output should look something like: 
```
ClusterId:              dZmpfrGnSaq_XBDWsSsKPg
LeaderId:               0
LeaderEpoch:            25951
HighWatermark:          140536
MaxFollowerLag:         0
MaxFollowerLagTimeMs:   0
CurrentVoters:          [0,1,2]
CurrentObservers:       [3,4,5]
```

CurrentObservers are the actual Kafka brokers that we are interested in. These form our Kafka Cluster. Notice that we have 3 brokers with IDs 3,4,5 (check your own output from terminal). CurrentVoters are the Kafka controllers, responsible for managing consensus. We will not focus on these. 


## Listing the topics 

<iframe width="560" height="315" src="https://www.youtube.com/embed/kj9JH3ZdsBQ?si=_aLSAtxvfeOw7IvY" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>

The command below allows you to see all the topics in our cluster. Remember that a Kafka topic is an abstraction that groups related events together. Consumers can subscribe to topics they are interested to. 

!!! Tip  
    This command will list a few different topics. There should not be any topic with your name yet. We will create it soon. 

```
${KAFKA_PATH}/kafka-topics.sh --list --bootstrap-server $BOOTSTRAP_SERVER
```

## Creating your own topic 

<iframe width="560" height="315" src="https://www.youtube.com/embed/y9BStKvVzSs?si=7UVi35DdLuVuYu3r" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>

First read the command below. Running this command will create a new topic called `<your_username>_topic`. Notice that the topic will have both the number of partitions and the replication factor set as 1. 

```
${KAFKA_PATH}/kafka-topics.sh --create --bootstrap-server $BOOTSTRAP_SERVER --replication-factor 1 --partitions 1 --topic ${USER}_topic
```

Feel free to ignore the warning message. You can ensure that creation was succesful by running all the topics again and finding your own:
```
${KAFKA_PATH}/kafka-topics.sh --list --bootstrap-server $BOOTSTRAP_SERVER
``` 

## Inspecting your topic 

<iframe width="560" height="315" src="https://www.youtube.com/embed/jHnyBSUVcOU?si=V60dBJZD8-vIQeAE" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>

The following command will let you inspect your topic: 

```
${KAFKA_PATH}/kafka-topics.sh --describe --bootstrap-server $BOOTSTRAP_SERVER --topic ${USER}_topic
```

Answer the following questions:   
1. Where is each partition stored? 
1. Is this model fault tolerant? 
1. Is this model scalable? 

## Altering the number of partitions 

Now we will change the number of partitions in the topic. Run: 

```
${KAFKA_PATH}/kafka-topics.sh --alter --bootstrap-server $BOOTSTRAP_SERVER --topic ${USER}_topic --partitions 3
```
If everything is succesful nothing will be printed. Inspect your topic again: 

```
${KAFKA_PATH}/kafka-topics.sh --describe --bootstrap-server $BOOTSTRAP_SERVER --topic ${USER}_topic
```

Things changed! You should now see 3 partitions. Given that we have exactly 3 brokers, Kafka will assign each partition to a different broker to balance load. 

Think again about the questions from before, particularly: 
1. Is this model fault tolerant? 
1. Is this model scalable? 

## Altering the replication factor 

<iframe width="560" height="315" src="https://www.youtube.com/embed/Vo6Mv5YPOJU?si=ozzMs5CVFDoOXUvv" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>

There isn't a way to simply alter the replication factor as we did with the partitions. What we will do instead is to delete your topic, recreate it with the new configuration and finally inspect it: 

```
${KAFKA_PATH}/kafka-topics.sh --bootstrap-server $BOOTSTRAP_SERVER --delete --topic ${USER}_topic
${KAFKA_PATH}/kafka-topics.sh --create --bootstrap-server $BOOTSTRAP_SERVER --replication-factor 2 --partitions 3 --topic ${USER}_topic
${KAFKA_PATH}/kafka-topics.sh --describe --bootstrap-server $BOOTSTRAP_SERVER --topic ${USER}_topic
```

Now you should see each partition being replicated in two brokers: the leader and an additional replica. Once again, Kafka will take care of distributing the partitions evenly across brokers. 

Think about the questions from before, particularly: 
1. Is this model fault tolerant? 
1. Is this model scalable? 


## Writing and reading topics

<iframe width="560" height="315" src="https://www.youtube.com/embed/I7zm3on_cQQ?si=qy-HpT4ZGX2Q2N3M" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>

<iframe width="560" height="315" src="https://www.youtube.com/embed/Z9g4jMQwog0?si=QU2Xg6lqBFdmoi1m" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>

First of all, duplicate your terminal window. SSH into PWS in the additional window. One tab will be the producer and the other will be the consumer. 

Create the consumer in one tab by running: 
```
${KAFKA_PATH}/kafka-console-consumer.sh --bootstrap-server $BOOTSTRAP_SERVER --topic ${USER}_topic --from-beginning  --property print.partition=true
```

You should keep this tab open and running. 

Then, turn your other tab into a producer by running the command below. You can type messages and press enter to publish them. 

```
${KAFKA_PATH}/kafka-console-producer.sh --broker-list $BOOTSTRAP_SERVER --topic ${USER}_topic --producer-property partitioner.class=org.apache.kafka.clients.producer.RoundRobinPartitioner 
```

As you publish new messages, you will see them showing up in the consumer! This indicates that your topics is working well within the Kafka cluster. 


## Writing and reading with keys 

You can now press `ctrl+d` on your producer. This will close it. Reboot your producer using the following command to enable key:value messages: 

```
${KAFKA_PATH}/kafka-console-producer.sh --broker-list $BOOTSTRAP_SERVER --topic ${USER}_topic --property "parse.key=true" --property "key.separator=:" 
```

Now you can send messages in the format `key:value`. For example, you can send `name:thomas`, where name is the key and thomas is the value. Observe how messages with the same key are sent to the same partition. Play around with a few messages. 

## THE END! 

And that's it for today's PCW! You can press `ctrl+d` on the producer and `ctrl+c` on the consumer to close both. Then exit PWS and you will be done! 