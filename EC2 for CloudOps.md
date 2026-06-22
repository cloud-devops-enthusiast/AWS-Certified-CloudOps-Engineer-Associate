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

