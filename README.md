# Project 2: Tracking User Activity using Spark

In this project, we built ETL pipelines to analyse course assessment data so data scientists can run some queries.
In order to do this, we did the following:

- Publish and consume messages with Kafka
- Use Spark to transform the messages. 
- Use Spark to store data in hdfs.

Here we uploaded `docker-compose.yml` used for spinning the pipeline. The commands can be seen from the file `swetaKafka-8-history.txt`. We used kafka to produce messages and send them to Spark. The Spark commands can be seen in the notebook.


## Data (Assumtions, limitations and queries)

To get the data, I ran:
```
curl -L -o assessment-attempts-20180128-121051-nested.json https://goo.gl/ME6hjp`
```
It is a nested JSON file with many fields. Not all are pertinent to our business questions, so we selected few fields out of this json file and formed a schema in our trasformation through Spark. The question fields are mostly nested andthey are different for different records depending on the course name (exam_name). The options field within the question field provides the true answer out of the other false multiple choice answers for every user id. These are probably multiple choice answers with one true value but that is an assumption. With such low cardinality of some fields (except the id's), it is difficult to get the correct counts.

Another problem was with the certification data. This data has only false and null values. So if we want to find out how many students took the certification option, we can't measure it unless we get some "true" values. 
Similarly, the "max_attempts" data showed only "1" as its value throughout the dataset. That doesn't help us to find out how many students took the assessments multiple times.

Keeping these assumptions and limitations, I formualted the following questions and designed my schema (transformation) accordingly.Here are some questions and answers that I presented in my notebook:

1. What is the most common course taken? **Learning Git**
2. How many people took *Learning Git*? **394**
3. How many courses were assessed? **103**
4. How many assesstments are in the dataset? **3242**
5. On average, what percentage students got correct answers? **62.6%**
6. How many students got all questions right? **841**

## Files submitted in Github

- swetabee-history.txt 
- docker-compose.yml files
- Jupyter Notebook with Spark pipelines
  
  
## Highlights on the ETL Pipeline

 The history file `swetaKafka-8-history.txt` consists of the commands on how to start a container with docker. The `docker-compose.yml` file includes the connections to zookeeper, kafka, mids, coursera and spark. I created a kafka topic with the name **assess** because it is relevant to the course assessment pipeline that I am about to build. I used Curl to get the JSON data and produce messages after connecting through kafka. Finally, I connect to Spark (Notebook `SwetaBee_project2_assess.ipynb`) in order to transform the extracted data and run some queries.
 Following are some highlights from the history file that I used to produce messages in Kafka:
 
 git clone https://github.com/mids-w205-de-sola/project-2-SwetaBee.git <br>
 cd project-2-SwetaBee/<br>
 git branch assignment<br>
 git checkout assignment<br>
 docker-compose up -d<br>
 docker-compose logs -f kafka<br>
 curl -L -o assessment-attempts-20180128-121051-nested.json https://goo.gl/ME6hjp<br>
 docker-compose exec kafka kafka-topics --create --topic assess --partitions 1 --replication-factor 1 --if-not-exists --zookeeper zookeeper:32181<br>
 docker-compose exec kafka kafka-topics --describe --topic assess --zookeeper zookeeper:32181<br>
 docker-compose exec mids bash -c "cat /w205/spark-with-kafka-and-hdfs/assessment-attempts-20180128-121051-nested.json | jq '.[]' -c | kafkacat -P -b kafka:29092 -t assess"<br>
 docker-compose exec cloudera hadoop fs -ls /tmp/<br>
 docker-compose exec spark env PYSPARK_DRIVER_PYTHON=jupyter PYSPARK_DRIVER_PYTHON_OPTS='notebook --no-browser --port 8888 --ip 0.0.0.0 --allow-root --notebook-dir=/w205/' pyspark<br>
 
