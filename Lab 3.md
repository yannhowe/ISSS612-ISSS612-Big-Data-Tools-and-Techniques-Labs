# B5 Big Data Tools and Technologies - Lab 3

## Overview

This lab builds on the activities in Lab 1 and 2. In addition to setting up the EMR cluster, you will need to set up access to twitter through their Application Programming Interface (API) and Apache Kafka to monitor a certain hashtag.

## Steps

These are the tasks you’ll need to complete the lab:
1.	Launch an EMR Cluster with some additional configurations and settings
2.	Set up Apache Kafka server
3.	Set up Twitter API access
4.  Read Twitter Feed into Kafka via Spark Streaming
5.  Shutdown and cluster termination

## Launch an Amazon Elastic Map Reduce (EMR) Cluster

[Amazon EMR](https://aws.amazon.com/emr/) provides a managed Hadoop framework. You can also run other popular distributed frameworks such as Apache Spark, HBase, Presto, and Flink in Amazon EMR, and interact with data in other AWS data stores such as Amazon S3 and Amazon DynamoDB.

Amazon EMR securely and reliably handles a broad set of big data use cases, including log analysis, web indexing, data transformations (ETL), machine learning, financial analysis, scientific simulation, and bioinformatics.

1.	Sign in to the AWS Management Console and open the Amazon EMR console at https://console.aws.amazon.com/elasticmapreduce/.
2.	Choose Create cluster

![Imgur](http://i.imgur.com/za7ow8Ch.jpg)
3.	Choose Go to advanced options

![Imgur](http://i.imgur.com/XaWK7Pqh.jpg)
4.	Choose Hadoop, Hive and Spark for this lab

![Imgur](http://i.imgur.com/mtacH3hh.jpg)
5.  You will also need to copy the following block of code (*all in one line*) into the text area `Edit software settings (optional)`.
```{json, count_lines}
[{"configurations":[{"classification":"export","properties":{"PYSPARK_PYTHON":"python34"}}],"classification":"spark-env","properties":{}}]
```
![Imgur](http://i.imgur.com/bzJmM7ch.png)

5.	Click Next
6.	For hardware settings, 1 master node will suffice for this lab. Select m3.xlarge EC2 instance type for master node and 0 Instance count for all other nodes.

![Imgur](http://i.imgur.com/oSOdEGuh.jpg)
8.	Click Next
9.	Provide a cluster name
10.	Disable Termination Protection

![Imgur](http://i.imgur.com/ScxMcxBh.jpg)
9.  Under “Additional Options", expand `Bootstrap Actions`, select the option `custom action` under `Add bootstrap action` and click `Configure and add`

Script Location:
```
s3://aws-bigdata-blog/artifacts/aws-blog-emr-jupyter/install-jupyter-emr5.sh
```
Optional Arguments:
```
--port 8880 --copy-samples
```

![Imgur](https://i.imgur.com/xcI6v0r.png)

11.	Click Next
12.	Choose the EC2 key pair `aws_emr_key` previously created in Lab 1

![Imgur](http://i.imgur.com/U76mfk1h.jpg)
13.	Click “Create Cluster”

AWS will begin spinning up the EC2 instances and configuring the Hadoop applications selected on the instances. **This could take a long time due to the bootstrapping actions**.

![Imgur](http://i.imgur.com/M47GXQdh.jpg)

> **Important! Note down the Master public DNS as you will need this often later on. It looks something like `ec2-54-164-153-7.compute-1.amazonaws.com`**

## Authorize Inbound Traffic to your master node

Previously in Lab 1 we allow the following inbound traffic ports:
* Port 22 (SSH) to access the command line
* Port 8888 (Hue Web Server) to access the Hue web-based interface

Open Port 8880 on the master node to access the Jupyter notebook over the internet

1.	Access the EC2 Dashboard by clicking on `Services` -> `Compute` -> `EC2`

![Imgur](http://i.imgur.com/AjzsLuRh.jpg)
1.  In the sidebar of the Amazon EC2 console under `NETWORK & SECURITY` choose `Security Groups`

![Imgur](http://i.imgur.com/xrwML85h.jpg)
2.	On your right, select the group with the group name `ElasticMapReduce-master`


3.	Click on Actions and select `Edit Inbound rules`.

![Imgur](http://i.imgur.com/SFJqwHnh.jpg)
4.	Click on `Add Rule`. In the new row that appears, enter `8880` under `Port Range`.

![Imgur](http://i.imgur.com/W0nkYkZh.jpg)
5.	Under source, select `Anywhere` and the empty text box on the right hand side will be filled with ‘0.0.0.0/0’ automatically.
6.	Click Save. You should then see the new rule the next time you view Edit Inbound rules.

> Note: A list of other default incoming ports you may need to allow can be found here - http://docs.aws.amazon.com/emr/latest/ManagementGuide/emr-web-interfaces.html

## Connecting to the web interface from a browser

> Important! You'll need to have opened Port `8880` on the master node to access the web interface. If you have not, go back to the [Authorize Inbound Traffic to your master node](#authorize-inbound-traffic-to-your-master-node) section and carry out the steps.

1. Open a modern browser like Chrome, Safari, Edge or Firefox
2. Browse to `http://your-master-node-public-dns:8880`, for example, `http://ecX-XX-XXX-XXX-X.compute-1.amazonaws.com:8880`


## Set up Kafka Server

We need to connect to the Master Node for administration using SSH. Follow the instructions for PC or Mac detailed in Lab 1. Once you have a SSH session, run the following commands

```{r, engine='sh', count_lines}
# Install Tweepy, Kafka-Python, findspark
sudo python3 -m pip install tweepy kafka-python findspark

#	Change owner of the /usr/lib folder to “hadoop”
sudo chown -R hadoop:hadoop /usr/lib

# Navigate to the /usr/lib folder
cd /usr/lib

# Download the Apache Kafka binary archive
sudo wget http://www-us.apache.org/dist/kafka/0.10.2.1/kafka_2.11-0.10.2.1.tgz

# Extract the TAR Archive
sudo tar -xzf kafka_2.11-0.10.2.1.tgz

# Navigate to the extracted Kafka directory
cd kafka_2.11-0.10.2.1

# Start the zookeeper server in the background
sudo /usr/lib/kafka_2.11-0.10.2.1/bin/zookeeper-server-start.sh -daemon config/zookeeper.properties

# Start the Apache Kafka server in the background
sudo /usr/lib/kafka_2.11-0.10.2.1/bin/kafka-server-start.sh -daemon config/server.properties

# Create a Kafka Topic with the topic name “twitterstream”
sudo /usr/lib/kafka_2.11-0.10.2.1/bin/kafka-topics.sh --create --zookeeper localhost:2181 --replication-factor 1 --partitions 1 --topic twitterstream
```

You have now created a topic in Kafka that is ready to receive events. Leave the SSH session open.

## Set up Twitter API Access

1.	Create a twitter account at http://www.twitter.com
2.	Go to https://apps.twitter.com and sign in with your twitter account
3.	Click on the `Create New App` button on upper right, fill in anything and leave the callbackURL blank
4.	Click on the `Keys and Access Tokens` tab.
5. Click on the `Create Access Token`
6. Copy the following four important pieces of information:

`Consumer Key (API Key)`,
`Consumer Secret (API Secret)`,
`Access Token`,
`Access Token Secret`

## Read Twitter Feed into Kafka via Spark Streaming

1.	On the Jupyter notebook hosted on your EMR cluster master node `http://your-master-node-public-dns:8880`, upload the `Lab 3_Jupyter Notebook 1_Twitter to Kafka` and `Lab 3_Jupyter Notebook 2_Using Spark Streaming` notebooks found on elearn.
2.	Open the “Lab 3_Jupyter Notebook 1_Twitter to Kafka” notebook. On the top of the notebook, enter the Twitter credentials you have saved previously.
3. Run the following commands in the SSH session running from the "Set up Kafka Server" section.

```
# To view messages sent to the topic, execute the following command. Keep the PuTTY window open for now:
sudo /usr/lib/kafka_2.11-0.10.2.1/bin/kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic twitterstream --from-beginning

# To pipe the messages sent to the topic into a both the terminal and a file, execute the following command:
sudo /usr/lib/kafka_2.11-0.10.2.1/bin/kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic twitterstream --from-beginning | tee ~/twitterstream.txt
```

Nothing will appear for now until you run the code in the `Lab 3_Jupyter Notebook 1_Twitter to Kafka` notebook.

4.	Run the codes in the first three steps. On the third step, after running the code, you will be able to see the test message appearing on the PuTTY Terminal
4.	Before running the codes in the fourth block, you may choose to add or remove hashtags to retrieve tweets from. For multiple hashtags, use a comma to separate them. Once ready, run the code.

```
# To open and read the file, open a separate SSH session and run the following command:
tail -f ~/twitterstream.txt
```

5.	Open the “Lab 3_Jupyter Notebook 2_Using Spark Streaming” notebook. Read the description and run through the steps as described in the notebook.

## Graceful shutdown
To end the lab, perform the following steps in sequence:
1.	Shutdown both Jupyter notebooks:
2.	In the PuTTY Terminal (PC) or Terminal (Mac), enter “Ctrl” and “C” keys together to terminate the consumer application.
3.	Execute the following commands to shut down Apache Kafka server and Zookeeper server:
```
/usr/lib/kafka_2.11-0.10.2.1/bin/kafka-server-stop.sh
```
```
/usr/lib/kafka_2.11-0.10.2.1/bin/zookeeper-server-stop.sh
```
4.	Close PuTTY Terminal and terminate the AWS EMR Cluster.


## Terminate your cluster
***You will be charged according to the number of hours which your cluster is left running. It is very important to terminate your cluster after you have finished using it. NOT TERMINATING THE CLUSTER CAN RESULT IN HUGE BILLS***.

> Tip: If you forget to terminate your cluster and accumulate a large bill, you can try to write in to Amazon Web Services customer service and request to have it waived.

### Terminate a Cluster Using the Console
You can terminate one or more clusters using the Amazon EMR console.

#### To terminate a cluster with termination protection off
1.	Sign in to the AWS Management Console and open the Amazon EMR console at https://console.aws.amazon.com/elasticmapreduce/.
2.	Select the cluster to terminate. Note that you can select multiple clusters and terminate them at the same time.
3.	Click Terminate.
4.	When prompted, click Terminate.

#### To terminate a cluster with termination protection on
If you have enabled the termination protection, there are a few more steps you need to carry out before being able to terminate your cluster.

1.	Sign in to the AWS Management Console and open the Amazon EMR console at https://console.aws.amazon.com/elasticmapreduce/.
2.  On the Cluster List page, select the cluster to terminate. Note that you can select multiple clusters and terminate them at the same time.
3.  Click Terminate.
4.	When prompted, click Change to turn termination protection off. Note, if you selected multiple clusters, click Turn off all to disable termination protection for all the clusters at once.
5.	In the Terminate clusters dialog, for Termination Protection, click Off and then click the check mark to confirm.
6.	Click Terminate.
