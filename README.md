How to Visualize and Refine Your Network’s Security by Adding Security Group IDs to Your VPC Flow Logs
================================================================

## About this lab
#### Scenario

Many organizations begin their cloud journey to AWS by moving a few applications to demonstrate the power and flexibility of AWS. This initial application architecture includes building security groups that control the network ports, protocols, and IP addresses that govern access and traffic to their AWS Virtual Private Cloud (VPC). When the architecture process is complete and an application is fully functional, some organizations forget to revisit their security groups to optimize rules and help ensure the appropriate level of governance and compliance. Not optimizing security groups can create less-than-optimal security, with ports open that may not be needed or source IP ranges set that are broader than required.

![1.png](/images/1.png)

As illustrated in the preceding diagram, this is how the data flows in this model:

* The VPC posts its flow log data to Amazon CloudWatch Logs.
* The Lambda ingestor function passes the data to Firehose.
* Firehose then passes the data to the Lambda decorator function.
* The Lambda decorator function performs a number of lookups for each record and returns the data to Firehose with additional fields.
* Firehose then posts the enhanced dataset to the Amazon ES endpoint and any errors to Amazon S3.



## Prerequisites

>Make sure your are in US East (N. Virginia), which short name is us-east-1.

>Download Putty: IF you don’t already have the **PuTTy client/PuTTYgen** installed on your machine, you can download and then launch it from here:
https://www.chiark.greenend.org.uk/~sgtatham/putty/latest.html
