# B5 Big Data Tools and Technologies - Lab 1

## Overview

This lab focuses on setting up the environment required to process Twitter tweets in real time. This will consist of an Elastic Map Reduce cluster on Amazon Web Services, a way to connect to the cluster for administration called SSH which requires a key pair to keep it secure. You will access the notebooks and code on your cluster via a modern web browser like Chrome.

## Steps

These are the tasks you’ll need to complete the lab:
1.	Set up an Amazon Web Services (AWS) account
2.	Create a Key Pair to access the Elastic Map Reduce (EMR) Cluster you are going to launch
3.	Launch an EMR Cluster
4.	Connect to EMR Cluster Master Node using SSH
5.	Terminate Cluster to stop charges

## Set up your Amazon Web Services (AWS) account

### Sign Up for an AWS account

This course contains several hands-on sections which make use of AWS to provision an environment with the necessary big data tools for your learning. Go to https://aws.amazon.com/account/ to sign up.

> Please note that a credit card is needed. If you do not have access to a credit card, there is an alternative which is not recommended as it has some restrictions. However, please contact your course faculty or teaching assistant for more assistance if you wish to explore this alternative.

### Sign up for AWS Educate to receive service credits

Singapore Management University is part of the Amazon Web Services Educate (AWS Educate) grant program which is specially designed for educators and students. As a MITB student, you are given service credits for utilization of AWS in the hands-on sections of this course.

Sign up for your AWS Educate grant to receive your service credits at https://www.awseducate.com/Application.

> Important! You must use your SMU email address for the application. There will be an option to select an AWS Educate Starter Account. You are advised NOT to select that option unless you have chosen the alternative as described in the previous step.

## Create Your Key Pair

A key pair encrypts and decrypts communication for security purposes. The private key resides on your server while the public key can be added to computers that are allowed to access the server.

In this section, you will create a key pair using the Amazon EC2 console. This key pair will be used whenever you access your cluster. 

Create your key pair using the Amazon EC2 console
1.	Open the Amazon EC2 console at https://console.aws.amazon.com/ec2/.

![Imgur](http://i.imgur.com/AjzsLuRh.jpg)

2.	In the navigation pane, under NETWORK & SECURITY, choose Key Pairs.

![Imgur](https://i.imgur.com/gmruMS5.png)

3.	Choose Create Key Pair.

![Imgur](http://i.imgur.com/msNFnEih.png)

4.	Enter `aws_emr_key` for the new key pair in the Key pair name field of the Create Key Pair dialog box, and then choose Create.

![Imgur](http://i.imgur.com/g713BEqh.png)

5.	The private key file is automatically downloaded by your browser. The base file name is the name you specified as the name of your key pair, and the file name extension is `.pem`
6.	**Save the private key file on your Mac or PC desktop**
> Important! This is the only chance for you to save the private key file. You'll need to provide the name of your key pair when you launch an instance and the corresponding private key each time you connect to the instance.

## Launch an Amazon Elastic Map Reduce (EMR) Cluster

[Amazon EMR](https://aws.amazon.com/emr/) provides a managed Hadoop framework. You can also run other popular distributed frameworks such as Apache Spark, HBase, Presto, and Flink in Amazon EMR, and interact with data in other AWS data stores such as Amazon S3 and Amazon DynamoDB.

Amazon EMR securely and reliably handles a broad set of big data use cases, including log analysis, web indexing, data transformations (ETL), machine learning, financial analysis, scientific simulation, and bioinformatics.

1.	Sign in to the AWS Management Console and open the Amazon EMR console at https://console.aws.amazon.com/elasticmapreduce/.
2.	Choose Create cluster

![Imgur](http://i.imgur.com/za7ow8Ch.jpg)

3.	Choose Go to advanced options

![Imgur](http://i.imgur.com/XaWK7Pqh.jpg)

4.	Choose *at least* Hadoop, Hive, Hue and Spark for this lab and any other packages you might be interested in using

![Imgur](http://i.imgur.com/mtacH3hh.jpg)

5.	Click Next
7.	Select m3.xlarge EC2 instance type for master node and 0 Instance count for core nodes. We do not need any core or task nodes for your hands-on and we can minimize the charges of running your EMR custer.

![Imgur](http://i.imgur.com/oSOdEGuh.jpg)

8.	Click Next
9.	Provide a cluster name
10.	Disable Termination Protection

![Imgur](http://i.imgur.com/ScxMcxBh.jpg)

11.	Click Next
12.	Choose the EC2 key pair `aws_emr_key` previously created

![Imgur](http://i.imgur.com/U76mfk1h.jpg)

13.	Click “Create Cluster”

AWS will begin spinning up the EC2 instances and configuring the Hadoop applications selected on the instances. **This could take over 10 minutes**.

When you see the following screen, the cluster is ready
> **Important! Note down the Master public DNS as you will need this often later on. It looks something like `ec2-54-164-153-7.compute-1.amazonaws.com`**

## Authorize Inbound Traffic to your master node

Before connecting to your master node you must allow inbound traffic for port 22 (SSH) to access the command line as well as port 8888 (Hue Web Server) to access the web-based interface from your browser to the instance where your master node resides.

1.	Access the EC2 Dashboard by clicking on `Services` -> `Compute` -> `EC2`

![Imgur](http://i.imgur.com/AjzsLuRh.jpg)

1.  In the sidebar of the Amazon EC2 console under `NETWORK & SECURITY` choose `Security Groups`

![Imgur](http://i.imgur.com/xrwML85h.jpg)

2.	On your right, select the group with the group name `ElasticMapReduce-master`
3.	Click on Actions and select `Edit Inbound rules`.

![Imgur](http://i.imgur.com/SFJqwHnh.jpg)

4.	Click on `Add Rule`. In the new row that appears, select `SSH` under `Type`
5.	Under source, select `Anywhere` and the empty text box on the right hand side will be filled with ‘0.0.0.0/0’ automatically.
4.	Click on `Add Rule`. In the new row that appears, enter `8888` under `Port Range`.
5.	Under source, select `Anywhere` and the empty text box on the right hand side will be filled with ‘0.0.0.0/0’ automatically.

![Imgur](http://i.imgur.com/ld2cEA7h.jpg)

6.	Click Save. You should then see the new rule the next time you view Edit Inbound rules.

> Note: A list of other default incoming ports you may need to allow can be found here - http://docs.aws.amazon.com/emr/latest/ManagementGuide/emr-web-interfaces.html

## Connecting to the web interface from a browser

> Important! You'll need to have opened Port `8888` on the master node to access the web interface. If you have not, go back to the [Authorize Inbound Traffic to your master node](#authorize-inbound-traffic-to-your-master-node) section and carry out the steps.

1. Open a modern browser like Chrome, Safari, Edge or Firefox
2. Browse to `http://your-master-node-public-dns:8888`, for example, `http://ecX-XX-XXX-XXX-X.compute-1.amazonaws.com:8888`


## Connecting to the Master Node for administration using SSH

In order to connect to the master node, we need the following
1.	An application like PuTTY (PC) or Terminal (Mac)
2.	The Key Pair private key (`.pem` file)
3.	The public DNS name of the master node

We use a network protocol called SSH to connect to and issue commands to the master node securely.

Using SSH to connect to the master node gives you the ability to monitor and interact with the cluster. You can issue Linux commands on the master node, run applications such as Hive and Pig interactively, browse directories, read log files, and so on.

In the labs, you will be connecting to the EC2 instance that is acting as the master node of your Hadoop cluster. On PC we use a software called PuTTY. For Mac, we use the Terminal which comes together with the Operating System.

## Connecting from a PC

To connect from a PC, you’ll need to
1.	Convert the `.pem` file to a `.ppk` file that PuTTY can use
2.	Connect to the EMR Cluster Master Node using PuTTY

### Convert your private key

PuTTY does not natively support the private key format (`.pem`) generated by Amazon EC2. PuTTY has a tool named included with the installation called `PuTTYgen` which can convert keys to the required PuTTY format (`.ppk`). You must convert your private key into this format (`.ppk`) before attempting to connect to your instance using PuTTY.

1.	Start `PuTTYgen` (for example, from the Start menu, click All Programs > PuTTY > `PuTTYgen`).
2.	Under Type of key to generate, select SSH-2 RSA.
3.	Click Load. By default, PuTTYgen displays only files with the extension `.ppk`. To locate your `.pem` file, select the option to display files of all types.
4.	Select your `.pem` file for the key pair that you specified when you launch your instance, and then click Open. Click OK to dismiss the confirmation dialog box.
5.	Click Save private key to save the key in the format that PuTTY can use. PuTTYgen displays a warning about saving the key without a passphrase. Click Yes.
Note
A passphrase on a private key is an extra layer of protection, so even if your private key is discovered, it can't be used without the passphrase. The downside to using a passphrase is that it makes automation harder because human intervention is needed to log on to an instance, or copy files to an instance.
1.	Specify the same name for the key that you used for the key pair (for example, `my-key-pair`). PuTTY automatically adds the `.ppk` file extension.
Your private key is now in the correct format for use with PuTTY.

> Important! You'll need to have allowed `SSH` or the `inbound` port `22` on the master node to access the command line interface.  If you have not, go back to the [Authorize Inbound Traffic to your master node](#authorize-inbound-traffic-to-your-master-node) section and carry out the steps.

Connect using PuTTY
Windows users can use an SSH client such as PuTTY to connect to the master node. If you have not already installed PuTTY, please do so from the PuTTY download page.
Double-click putty.exe to start PuTTY. You can also launch PuTTY from the Windows programs list.
1.	If necessary, in the Category list, choose Session.
2.	In the Host Name (or IP address) field, enter the public DNS which you obtained from the previous step. For example:  `ec2-54-164-153-7.compute-1.amazonaws.com`.
3.	In the Category list, expand Connection > SSH, and then choose Auth.
4.	For Private key file for authentication, choose Browse and select the `.ppk` file that you generated.
5.	Choose Open.
6.	Choose Yes to dismiss the PuTTY security alert.

> Important When logging into the master node, type `hadoop` if you are prompted for a user name. **Note that there is no password**.

Upon successful connection, you will have a Command-Line Interface (CLI) access to the master node which will look like the following.

When you are done working on the master node, you can close the SSH connection by closing PuTTY.

> Note: To prevent the SSH connection from timing out, you can choose Connection in the Category list and select the option Enable TCP_keepalives. If you have an active SSH session in PuTTY, you can change your settings by right-clicking the PuTTY title bar and choosing Change Settings.

### Connecting from a Mac

To connect from your PC, open `Terminal` by pressing `cmd + space`. Type in `Terminal` and press enter

enter the following command but replaced with your Master node's public DNS address:
```{r, engine='sh', count_lines}
ssh -i ~/Desktop/aws_emr_key.pem hadoop@`ecX-XX-XXX-XXX-X.compute-1.amazonaws.com`
```

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
