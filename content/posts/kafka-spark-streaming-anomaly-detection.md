---
title: "Event Aggregation and Anomaly Detection with Spark Streaming, AWS Managed Kafka and Athena"
date: 2024-04-27T09:22:22+08:00
tags: ["tutorial", "aws", "kafka", "data-eng"]
---

This explores a hypothetical use-case where electricity usage is monitored in real-time with an intent to reduce consumption to help meet both cost reduction and sustainability goals.  Streaming usage data is aggregated into hourly average consumption for Analysts use.  Additionally real-time alerts are generated whenever a electricity usage anomaly is detected (i.e. a deviation from previous exected usage values).

GitHub repository for project files can be found [here](https://github.com/ckevinhill/aws_kafka_streaming_anomaly).

##### Use-Case Data

Data for use-case exploration is sourced from Boston [Public Library in Copley Square](https://www.bpl.org/locations/central/) and made available via [Analyze Boston](https://data.boston.gov/dataset/central-library-electricity-usage).

Data is structured in a tabular format with 3 columns:

* usage_datetime_start_eastern_time - provides start of monitoring window
* usage_datetime_end_eastern_time - provides end of monitoring window
* usage_kw - measured kilowatt usage during monitoring window.

__Note:__ Data is provided daily, with a temporal granularity of every 5 minutes.  In order to support a streaming data use-case we will "play back" data via a Streaming Producer to simulate a truely streaming data source.

##### Use-Case Technical Overview

This use-case is supported via the following infrastrutcure:

![Architecture Diagram](/images/kafka-usage-streaming-infra.png)

Main architecture systems and technologies discussed include:

* [Amazon Managed Streaming for Kafka](https://aws.amazon.com/msk/) to provide provisioned Kafka infrastructure and brokers for streaming data processing.
* [Terraform](https://www.terraform.io/) for infrastructure deployment automation.
* [Spark Streaming](https://spark.apache.org/streaming/) for streaming analytics and aggregation.
* [Delta Lake format](https://docs.delta.io/latest/index.html) for data table persistence.
* [AWS Athena](https://aws.amazon.com/athena/) to provide a query engine backed by Delta and S3.
* [PowerBI Desktop](https://powerbi.microsoft.com/en-us/desktop/) to provide a Dashboard interface for reporting.
* [Sci-kit Learn](https://scikit-learn.org/stable/) to provide ML packages for anamoly detection.

Electricity usage monitoring events are generated via the Streaming Producer and published to the Kafka `usage` topic.  From there, a Spark Streaming Consumer aggregates hourly average reported usage and writes to S3-backed Delta Lake file via upserts.  PowerBI connects to Athena to execute queries against the Delta tables and provide interactive analytics to a Business Analyst.  Additionally a polling Consumer processes real-time events and compares to values from previous same day/hour/minute periods.  If anamolies are identified (via Isolation Forest ML) an additional event is published to Kafka `anomaly` topic for potential down-stream processing.

## Infrastructure Setup

#### S3 Bucket creation

To establish location for assets (e.g. code) and data persistence we will create a S3 bucket with the following folders:

* assets/ - contains code/files that will be downloaded and executed by EC2 producer and consumer instances
* deltalake/ - will contain the deltalake formatted datatables from streaming aggregation consumer
* checkpoint/ - will contain checkpoints from Spark Streaming writeStream

![S3 bucket](/images/kafka-usage-streaming-s3.png)

Within the `assets` folder you should include the following files for EC2 instance use:

* data_stream.csv - [data from Boston Analyze](https://data.boston.gov/dataset/central-library-electricity-usage) representing usage monitoring stream.
* [producer.py & prod-requirements.txt](https://github.com/ckevinhill/aws_kafka_streaming_anomaly/tree/main/kafka) - provides Python code for streaming Producer and pip install requirements.
* [consumer.py & cons-requirements.txt](https://github.com/ckevinhill/aws_kafka_streaming_anomaly/tree/main/kafka) - provides Python code for Spark Streaming aggregation Consumer and pip install requirements.
* [ml.py & ml-requirements.txt](https://github.com/ckevinhill/aws_kafka_streaming_anomaly/tree/main/ml) - provides Python code for Sci-kit learn anomaly detection Consumer polling implementation.

![S3 Assets](/images/kafka-usage-streaming-s3-assets.png)

__Note:__ Asset file names can be changed but would require respective updates to EC2 Terraform templates.  For instance you can see line [in producer.tftpl](https://github.com/ckevinhill/aws_kafka_streaming_anomaly/blob/main/terraform/producer.tftpl) 'hard-coded' reference to `producer.py`:

```bash
aws s3 cp s3://cscie192-final/assets/producer.py ./producer.py
```

Future updates could consider automating creation of S3 bucket and upload of asset files via Terraform.

#### Deploying Kafka (MKS) Infrastructure with Terraform

In a [previous article](/posts/kafka-faust-elasticsearch-hdfs/) we have looked at running Kafka in a local Docker container.  For this exploration we would like to use various AWS services and so will deploy into a Managed AWS environment ([MSK](https://aws.amazon.com/msk/)).

Given complexity of infrastruture deployment we should automate via Terraform.  [This article](https://medium.com/datamindedbe/simplifying-kafka-cluster-deployment-step-by-step-guide-with-amazon-msk-and-terraform-a3643eaf903a) provides a solid foundation for MSK terraform deployment and outlines default configuration options selected in cluster deployment.  Minor changes specific for use-case can be found in GitHub repository [here](https://github.com/ckevinhill/aws_kafka_streaming_anomaly/tree/main/terraform).  Changes included:

* Adding AWS t2.small EC2 "Producer" Ubuntu instance in public subnet ([template](https://github.com/ckevinhill/aws_kafka_streaming_anomaly/blob/main/terraform/producer.tftpl))
* Adding AWS t2.small EC2 "Consumer" Ubuntu instance in public subnet ([template](https://github.com/ckevinhill/aws_kafka_streaming_anomaly/blob/main/terraform/consumer.tftpl))
* Update AWS MKS Cluster instance_type to "kafka.t3.small" vs. "kafka.m5.large" to reduce dev costs
* Update AWS MKS Cluster EBS to 100G instead of 1Tb to reduce dev costs
* Update [outputs.tf](https://github.com/ckevinhill/aws_kafka_streaming_anomaly/blob/main/terraform/outputs.tf) to include consumer and produce SSH connect strings
* Updated [variables.tf](https://github.com/ckevinhill/aws_kafka_streaming_anomaly/blob/main/terraform/variables.tf) to use `us-east-1` and project-specific global-prefix.
* Creation of role `ec2_s3_full_access_iam_role` to enable S3 bucket access that is applied to EC2 instances via `iam_instance_profile` parameter.

Instructions on installation of WSL and Terraform can be found [here](https://levelup.gitconnected.com/install-terraform-on-windows-subsystem-linux-54de63683515).  Given AWS cloud deployment you will also need to install AWSCli (`sudo apt install awscli`) and configure with an account that has IAM permissions for resource creation.

##### IAM Policies

Below provides list of policies that will likely be convient for tutorial completion:

![IAM](/images/kafka-usage-streaming-iam.png)

Additional IAM role for EC2 access to S3 buckets:

![S3 EC2 IAM](/images/kafka-usage-streaming-iam-s3-ec2.png)

__Warning:__ Permissions are not fine-tuned for security and should not be considered recommendation for a production deployment.

Once installation and configuration is completed, infrastructure can be deployed via below with total deployment taking 20 to 50 minutes:

```bash
> terraform init
> terraform apply
```

__FYI:__ Infrastruture can be torn down via `terraform destroy`.

Once deployment is complete you should be able to connect to bastion, consumer and producer EC2 instances via SSH:

```bash
> sudo ssh -i cert.pem [ubuntu|ec2-user]@[aws_public_ip]
```

__FYI:__ `sudo` command is used to circumvent permissions issues on cert.pem.  User (`ubuntu` or `ec2-user`) varies depending on the Instance type (linux AMI or Ubnuntu) used.  Producer and Consumer Instances are Ubuntu based so use the `ubuntu` user.  Public IPs can be viewed in [EC2 Instance dashboard](https://us-east-1.console.aws.amazon.com/ec2/home?region=us-east-1#Home:) in AWS.

## Streaming Producer

Our Streaming Producer ([code](https://github.com/ckevinhill/aws_kafka_streaming_anomaly/blob/main/kafka/producer.py)) is responsible for "playing back" of the downloaded electric usage data to simulate a real-time stream.  Essentially, the Producer reads the data and publishes the next set of data as if it was being published at that point in time.  Producer can be configured to choose starting point for data publication (i.e. start playback on particular date) as well as accelerator factors (e.g. 10x) to increase speed of playback (turn minute gaps into seconds) to help with simulation testing.  Producer development examples sourced from [here](https://andres-plazas.medium.com/a-python-kafka-producer-edafa7de879c) and [kafka-client documentation](https://docs.confluent.io/kafka-clients/python/current/overview.html).

Producer can be launched via:

```bash
> source aws/bin/activate
> python producer.py --bootstrap_server $broker --file_path data_stream.csv
```

__FYI:__ `$broker` is set as environmental variable during [EC2 instance creation](https://github.com/ckevinhill/aws_kafka_streaming_anomaly/blob/main/terraform/producer.tftpl).

Additional command line options include:

* step_frequency - to set the polling frequency (defaults to 5, inline with electricity usage periodicity)
* accelerator - adds a multiplication factor to "speed up" playback increasing the range of records for each pull by provided factor.

__FYI:__ Other options like playback start date or max number of event iterations can be set directly within the code.

Once launched, console will output events being published (indefinitely until interrupted):

![Events](/images/kafka-usage-streaming-producer-output.png)

## Streaming Aggregation Consumer

Our Streaming Aggregation Consumer ([code](https://github.com/ckevinhill/aws_kafka_streaming_anomaly/blob/main/kafka/consumer.py)) is responsible for streaming kafka events into S3 as delta-lake formmatted tables.  This aggregator will average out usage values within a 1 hour window.  Consumer developement examples sourced from [here](https://medium.com/wehkamp-techblog/streaming-kafka-topic-to-delta-table-s3-with-spark-structured-streaming-2bb3027c7565).

Streaming aggregation can be launched via:

```bash
> source aws/bin/activate
> spark-submit --packages 
                    org.apache.spark:spark-hadoop-cloud_2.12:3.5.1,
                    org.apache.spark:spark-sql-kafka-0-10_2.12:3.5.1,
                    io.delta:delta-spark_2.12:3.1.0 
                --conf "spark.sql.extensions=io.delta.sql.DeltaSparkSessionExtension" 
                --conf "spark.sql.catalog.spark_catalog=org.apache.spark.sql.delta.catalog.DeltaCatalog" 
                consumer.py --boostrap_server $broker --s3_bucket [bucket_name]
```

__Watch-out:__ Command should be executed as *one line* without spaces between packages.  The above is seperated per line for readability.  Specific deltalake, s3 support and kafka packages are required for Consumer execution.

##### Load Kafka Stream

```python
df_kafka_stream = ( spark
                   .readStream
                   .format("kafka")
                   .option("kafka.bootstrap.servers", args.bootstrap_server)
                   .option("subscribe", "usage")
                   .option("startingOffsets", "earliest")
                   .load() 
                )
```

##### Convert Kafka binary JSON to Structured Dataframe

Provide a schema for processing and "explode" nested JSON values into columns.

```python
# Define JSON Schema:
schema = StructType([ 
    StructField("ts", TimestampType(), True),
    StructField("value" , FloatType(), True),
    ])

# Convert Kafka JSON to structured columns:
df_json_stream = ( df_kafka_stream
                  .select(
                      from_json(
                          col("Value").cast("String"),
                          schema).alias("json_value")
                      )
                  ).select(col("json_value.*"))
```

##### Create Aggregation Window

Specify aggregation window of 1 hour.  Additionally set a watermark time-out of 1 hour to prevent inclusion of delayed messages.

```python
# Aggregate streaming values:
df_window_stream = ( df_json_stream
                    .withWatermark("ts", "1 hour")
                    .groupBy(
                        window("ts", "1 hour")
                    ).agg(avg("value").alias("avg_usage"))
                    ).select("window.start", "window.end", "avg_usage")
```

##### Write to Delta Lake format

Establish an upsert strategy based on usage start_date column.  If new messages arrive with same start date overwrite old, or add as new row.  Write out structured data to S3 in deltalake format.

```python
# Create Dataframe w/ window_stream schema
(spark
 .createDataFrame([], df_window_stream.schema)
 .write
 .option("mergeSchema", "true")
 .format("delta")
 .mode("append")
 .save(delta_location))

# Set upsert criteria
def upsertToDelta(df, batch_id): 
  (DeltaTable
   .forPath(spark, delta_location)
   .alias("t")
   .merge(df.alias("s"), "s.start = t.start")
   .whenMatchedUpdateAll()
   .whenNotMatchedInsertAll()
   .execute()) 

# Write to S3 as Delta
w = (df_window_stream
 .writeStream
 .format("delta")
 .option("checkpointLocation", checkpoint_location) 
 .foreachBatch(upsertToDelta) 
 .outputMode("update") 
 .start(delta_location))
```

##### Athena Query Engine

After launch, you should start to see parquet files created in the `deltalake` S3 location indicating that data is starting to stream into tables:

![deltalake](/images/kafka-usage-streaming-aggregation-output.png)

You can additionally add the S3 location as a table datasource in Athena to enable querying of newly created Delta tables:

![athena-add](/images/kafka-usage-streaming-athena-create.png)

![athena-schema](/images/kafka-usage-streaming-athena-schema.png)

Query the deltalake tables with SQL:

![athena-query](/images/kafka-usage-streaming-athena-query.png)

Multiple executions of the query will show data changes as the Streaming Aggregator continues to process messages:

![athena-query-1](/images/kafka-usage-streaming-athena-query-1.png)
![athena-query-1](/images/kafka-usage-streaming-athena-query-2.png)

##### PowerBI Desktop Interface

PowerBI Desktop can be connected to Athena via ODBC drivers with instructions listed [here](https://docs.aws.amazon.com/athena/latest/ug/odbc-v2-driver.html).

![odbc](/images/kafka-usage-streaming-athena-odbc.png)

This will allow us to import (or direct query) Athena tables into PowerBI.  

![pbi-import](/images/kafka-usage-streaming-athena-pbi-import.png)

![pbi-import-table](/images/kafka-usage-streaming-athena-pbi-import-table.png)

For example graphing of the average hourly electricity usage for consecutive days illustrated below:

![pbi](/images/kafka-pbi.png)

![pbi-2](/images/kafka-pbi-2.png)

__Note:__ Other methods of visualizing data based on Athena are also possible - including [AWS QuickSights](https://aws.amazon.com/quicksight/) as well as [AWS managed Grafana](https://aws.amazon.com/grafana/).  These methods were explored and rejected for either cost reasons (i.e. QuickInsights monthly fee) or issues with managing permissions (i.e. Grafana execution was blocked by permission issue conflicts between IAM and IAM Management Center setup).

## Anomaly Detection

Our Anomaly Detection Consumer ([code](https://github.com/ckevinhill/aws_kafka_streaming_anomaly/blob/main/ml/ml.py)) is responsible for identifying outlier usage values and creating an associated message in the Kafka `anomaly` topic for downstream alerting.

An exploration of data trends can be found in [ML EDA Notebook](https://github.com/ckevinhill/aws_kafka_streaming_anomaly/blob/main/ml/ml-eda.ipynb) highlighting generally consistent trends with trend variation at day of week and hour:min time of day:

![ml-eda](/images/kafka-usage-streaming-ml-eda.png)

As an example, selecting a set of 13 comparable usage values on Mondays at 8:10 AM EST we can see fairly consistent usage and one outlier on Jan 23, 2024:

![ml-eda-dataset](/images/kafka-usage-streaming-ml-eda-day-hour-min.png)

Given the above exploration, it is believed that we can feasibly identify outliers - but that we will likely need to compare points from same day, hour and minute to gain quality peer comparison points.  Anamoly detection will need to be unsupervised (i.e. not using a pre-labeled data-set), flexible and have low fit and inference execution time.

Based on articles [here](https://medium.com/@richa.mishr01/anomaly-detection-in-seasonal-time-series-where-anomalies-coincide-with-seasonal-peaks-9859a6a6b8ba), [here](https://towardsdatascience.com/real-time-anomaly-detection-with-apache-kafka-and-python-3a40281c01c9), [here](https://towardsdatascience.com/practical-guide-for-anomaly-detection-in-time-series-with-python-d4847d6c099f
), [here](https://medium.com/@goldengoat/how-to-perform-anomaly-detection-in-time-series-data-with-python-methods-code-example-e83b9c951a37
) and [here](https://neptune.ai/blog/anomaly-detection-in-time-series) it was decided to use an unsupervised anamoly detection approach of [Isolation Forests](https://scikit-learn.org/stable/modules/generated/sklearn.ensemble.IsolationForest.html) from the sci-kit learn package.  Given the by day-hour-min variation, seperate Isolation Forests will be built at this level.

__Note:__ Experimentation was also done with seasonal/ARIMA forecast approaches (e.g. Prophet) but found not as accurate.  Additionally a simpler Isolation Forest fitted at the day level was also experimented with but was unable to capture variance by minute (as categorical or regressive variable).

##### Implementation

A Consumer polling loop [is implemented](https://github.com/ckevinhill/aws_kafka_streaming_anomaly/blob/main/ml/ml.py) to process newly created events in the `usage` topic.  We will store a history of points that will be used for anomaly detection.  In order to protect against memory overflows, a [deque](https://docs.python.org/3/library/collections.html#deque-objects) is used with a fixed size limit to maintain last **N** events:

```python
class FixedEventList:
    
    def __init__(self, max_size) -> None:
        self._queue = deque( maxlen= max_size )
    
    def add(self, ev:Event ):
        self._queue.append(ev)
```

If sufficient comparable points are available (currently set for min of 4) then an Isolation Forest is built on the comparable points and new point is scored using model.  If considered an outlier a `-1` value is returned as prediction, otherwise `1`.

```python
# Build Isolation model from comparable points:
iso_forest.fit( comps[["usage"]] )

# Score new point:
new_item = pd.DataFrame([new_event.toDict()])
prediction = iso_forest.predict( new_item[["usage"]] )

# If prediction is -1 (anamoly) publish to topic:
if prediction[0] == -1:
    print("Publishing anomaly")
    pl = {"ts": msg_json["ts"],"usage" : msg_json["value"]}
    p.produce("anomaly", value=json.dumps(pl))
```

##### Launching Detection Consumer

From the Consumer EC2 instance the Anomaly Detector can be launched via:

```bash
> source aws/bin/activate
> python3 ml.py --bootstrap_server $broker --max_history 70
```

__Note:__ For testing purposes you may want to launch Producer with a high accelerator value to speed up history build:

```bash
>> python3 producer.py --bootstrap_server $broker --file_path data_stream.csv --accelerator 1000
```

As Detector executes it will output available comparable (comp) points:

![comps](/images/kafka-usage-streaming-ml-processing.png)

Once reaching more than 4 comparables predictions will start to be executed:

![Comps-2](/images/kafka-usage-streaming-ml-processing-2.png)

#### Viewing Anomalies

The `anomaly` topic can be monitored from the Consumer EC2 instance using Kafka utilities:

```bash
> /kafka_2.13-3.4.0/bin/kafka-console-consumer.sh --bootstrap-server $broker --topic anomaly
```
![anomaly](/images/kafka-usage-streaming-ml-processing-anom.png)

Below shows Producer (left), ML Anomaly Detection (middle), and Consumer output of `anomaly` topic (right) to illustrate data flow through of full sequence via Kafka topic queues:

![full](/images/kafka-usage-streaming-ml-processing-full.png)

Other Consumers could be implemented to process `anomaly` events, for example posting to messaging channels or interfacing with high-power consumption IOT devices to prompt immediate manual or automated interventions in electricity consumption.