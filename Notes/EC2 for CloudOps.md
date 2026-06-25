# **Change Instance Type**

This works only for EBS backed Instance.
Select the Instance —> Then select, Stop the Instance —> Go to the instance type, Change Instance Type.
Once done with the steps, Click on Start Instances

#### # [SAA] EC2 Placement Groups
It is for the time when you want to take control over the EC2 instance placement strategy. In this way you can tell your requirements to AWS and they can have your hardware defined in as per your requirement.
When you create a placement group, you can specify one of the following strategies for the group:
* Cluster: Cluster all the instances into low latency group in a single availability zone.
* Spread: Spread instances across the underlying infrastructure or hardware (max 7 Instances per group per Availability Zone) (It is more of recommended for Critical Applications).
* Partition: Spreads instances across many different partitions (which rely on different set of racks) within an availability zones. Scales to 100’s of EC2 instances per group (Hadoop, Cassandra, Kafka) workloads.

#### Placement Groups (Cluster)
In this strategy Cluster or place your instances into a low latency group inside of a single Availability Zone.
Pros: Great network (10Gbps bandwidth between instances with enhanced networking enabled) Recommended.
Cons: If the AZ fails, all instances fails at the same time.
Use Case:
Big data job that needs to complete first.
Application that needs extremely low latency and high network throughput.

#### Placement Groups (Spread)
In this strategy you can spread or scatter your instances across the underlying hardware with maximum of 7 instances per placement group while taking care of any critical applications you deploy into it.
Pros:
Instances can be spreader across Availability Zones
Risk is reduced resulting in less simultaneous failures
In this configuration the EC2 instances are on different physical hardware
Cons:
Number of instances is reduced to 7 per placement group
Use Case:
This setup is for the applications that needs high availability
This is also for critical applications for which the instances should be isolated from failure from each other.


## EC2 SSH troubleshooting
* You have to make sure that the private key (pem file) on your linux machine has 400 permissions, else you will get “Unprotected private key file” error.
* Make sure that the username which you created for your OS of your Instance is given correctly when logging via SSH, else you will be getting errors like “Host key not found”, “Permission denied” or “Connection Closed by [Instance] port 22” error.
* Possible reasons for “Connection Timed Out” to EC2 instance via SSH:
  * SG or Security Group is not configured correctly
  * NACL is not configured correctly
  * Check for your route tables for the subnet (routes traffic destined outside VPC(Virtual Private Cloud) to IGW(Internet Gateway)

## SSH vs EC2 Instance Connect

* #### Connect Using SSH

* When you connect using SSH, your SSH pairs should be same as of source to get conncted to your instance but if your IPv4 pairs are different it blocks your connection.

For example you have Inbound Rules:
Type: SSH
Protocol: TCP
Port: 22
Source: 1.2.3.4/32

Now assume you have a security group inside which you have an instance with above given inbound rules.

Let's try to connect two users with different IPv4s:

* USER 1 (IPv4: 1.2.3.4): This user will be able to connect to the instance.

* USER 2 (IPv4: 5.6.7.8): This user will be blocked while making a connection attempt.

* #### Connect using EC2 Instance Connect

* Now here we will connect using EC2 Instance Connect, It works like you're on any IP address pair, now here the EC2 Console and EC2 instance connect CLI comes into picture where the user connects to instance using these tools. When user conects using this EC2 Instance Connect API it initiates a push request for one-time SSH public key which is valid for 60 second which as a layer of authentication.

For example you have below inbound IP rules:
Type: SSH
Protocol: TCP
Port: 22
Souce: 18.206.107.24/29

When user tries to connect with an IPv4 as 1.2.3.4 it initiates that push request mentioned above. 

For checking the ip ranges for different services you can use this json: https://ip-ranges.amazonaws.com/ip-ranges.json

* #### EC2 Instance Connect (EIC) Endpoint

* This service of AWS lets you to connect your private EC2 instances securely. In this service no internet gateway is required, No NAT Gateway is required, No Internet access is required.

Here we create a AWS EC2 Instance Connect Endpoint which has a security group attached to it where we must have enabled the outbound SSH traffic to the target EC2 instances.

For EC2 instance Security Group we must have allowed inbound SSH traffic from the EIC Endpoint Security Group. This way we can connect to our private EC2 instances securely without any internet access.

* #### CloudWatch Metrics for EC2

##### AWS Provides metrics (AWS pushes them):

  * Basic Monitoring (default): These metrics are collected at 5-minute intervals and are available for free.
  * Detailed Monitoring (Paid): These metrics are collected at 1-minute intervals and incur additional charges.
  * These metrics will include CPU Utilization, Network In/Out, Disk Read/Write and  Status Checks. 

##### Custom Metric (Yours to push):

  * Basic Resolution: These metrics are collected at 1-minute intervals and incur additional charges.
  * High Resolution: These metrics are collected at 1-second intervals and incur additional charges.
  * These metrics will include any custom metrics that you want to push to CloudWatch, such as RAM usage, application performance metrics, or any other data relevant to your application or infrastructure.
  * In this case, you need to make sure the IAM permissions over the EC2 instance are set correctly to allow pushing of custom metrics to CloudWatch.

##### EC2 Included metrics

  * CPU: CPU Utilization + Credit Usage/Balance
  * Network: Network In/Out
  * Status Checks:
    * Instance Status = Checks the EC2 VM
    * System Status = Checks the underlying AWS infrastructure
    * Attached EBS Status = Checks the attached EBS volumes
  * Disk: Disk Read/Write Operations + Disk Read/Write Bytes (Only for Instance Store Volumes)
  * **RAM** is not included in the AWS EC2 metrics.

##### Unified CloudWatch Agent

* These are used for virtual servers (EC2 Instances, On-premises servers and much more).
* This collects additional system-level metrics like RAM, processes, used disk space, etc.
* You can also send your logs to CloudWatch logs.
    * No logs are being sent to from inside of your EC2 instance to CloudWatch logs without installing the CloudWatch Agent.
* If you want to configure the CloudWatch Agent on your EC2 instance, you can use the centralized configuration using SSM parameter store to store your cofigurations.
* Make sure the IAM permissions over the EC2 instance are set correctly to allow pushing of custom metrics and logs to CloudWatch.
* Default namespace for the metrics collected by the Unified CloudWatch Agent is `CWAgent` and you can change it to your own namespace. The namespace can be changed in the configuration file of the CloudWatch Agent but here it will be added as a prefix to the metric name. 

This whole configuration is more of like,

EC2 Instance with Unified CloudWatch Agent installed —> Clouwatch Agent 
(where this Unified CloudWatch Agent will send the metrics and logs to CloudWatch.)

##### Unified CloudWatch Agent - procstat Plugin

* Using this plugin youb will be collecting metrics and monitor system utilization of the individual processes.
* This supports both linux and Windows based servers. For example, amount of time the process uses CPU, amount of memory the process uses, etc.
* Here you can make choice of which processes you need to monitor by
  * pid_file: You can specify the processes identification number (PID) file they create.
  * exe: You can specify the process name thatv matches your specified string name (RegEx is supported).
  * pattern: command line pattern which is used to start the process that matches your specified string name (RegEx is supported).
* Metrics collected by this plugin will be prefixed with `procstat` in the CloudWatch metrics namespace. (For example, `procstat_cpu_usage` is the metric name for CPU usage of the process being monitored or 'procstat_cpu_usage' is the metric name for CPU usage of the process being monitored.)