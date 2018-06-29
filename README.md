How to Visualize and Refine Your Network’s Security by Adding Security Group IDs to Your VPC Flow Logs
================================================================

## About this lab
### Scenario

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

>Download Putty and PuTTYgen: IF you don’t already have the **PuTTy client/PuTTYgen** installed on your machine, you can download and then launch it from here:
https://www.chiark.greenend.org.uk/~sgtatham/putty/latest.html

## Lab tutorial
### Create your Amazon ES cluster and VPC Flow Logs

1.1. 	In the **AWS Management Console**, on the service menu, choose **Elasticsearch** Service under Analytics.

1.2. 	Choose Create a new domain or Get started.

1.3. 	Type es-flowlogs for the **Elasticsearch domain name**.

1.4. 	Set **Version** to **5.1** in the drop-down list. Choose **Next**.

1.5. 	Set **Instance count** to **1** and set **Instance type** to **t2.small.elasticsearch**. Choose **Next**.

1.6. 	For Network configuration, select **Public Access**.

1.7. 	For Access Policy, Set the domain access policy to **Allow open access to the domain**. Click **I accept the risk** and choose **OK**. 

1.8. 	Choose **Next**.

1.9. 	On the next page, choose **Confirm**.

### Enable VPC flow logs

2.1. 	In the AWS Management Console, choose **CloudWatch** under **Management Tools**.

2.2. 	Click **Logs** in the navigation pane.

2.3. 	From the Actions drop-down list, choose **Create log group**.

2.4. 	Type **Flowlogs** as the **Log Group Name**.

2.5. 	In the AWS Management Console, choose **VPC** under **Networking & Content Delivery**.

2.6. 	Choose **Your VPCs** in the navigation pane, and select the VPC you would like to analyze.

2.7. 	Choose the **Flow Logs** tab in the bottom pane, and then choose **Create Flow Log**.

2.8. 	In the text beneath the **Role** box, choose **Set Up Permissions** (this will open an IAM management page).

2.9. 	Choose **Allow** on the **IAM management** page. Return to the VPC Flow Logs setup page.

2.10. 	Choose **All** from the **Filter** drop-down list.

2.11. 	Choose **flowlogsRole** from the **Role** drop-down list (you created this role in previous steps in this procedure).

2.12. 	Choose **Flowlogs** from the **Destination Log Group** drop-down list.

2.13. 	Choose **Create Flow Log**.

### Set up AWS Lambda to enrich the VPC Flow Logs dataset with security group IDs

3.1. 	Update a In the **AWS Management Console**, on the **service** menu, click **EC2**.

3.2. 	Click **Launch Instance**.

3.3. 	In the navigation pane, choose **Quick Start**, in the row for **Amazon Linux AMI**, click **Select**.

3.4. 	On **Step2: Choose a Instance Type** page, make sure **t2.micro** is selected and click **Next: Configure Instance Details**.

3.5. 	On **Step3: Configure Instance Details** page, enter the following and leave all other values with their default:

* Network: Default VPC
* Subnet: No preference
* Auto-assign Public IP: click Enable

3.6. 	Click **Next: Add Storage**, leave all values with their default.

3.7. 	Click **Next: Tag Instance**.

3.8. 	On **Step5: Tag Instance** page, enter the following information:

* Key: Name
* Value: Lab Server

3.9. 	Click **Next: Configure Security Group**.

3.10. 	On **Setp6: Configure Security Group** page, click **create a new security group**, enter the following information:

* Security group name: LabSecurityGroup
* Description: Enable SSH, HTTP and HTTPS access

3.11. 	Click **Add Rule**.

3.12. 	For Type, click **SSH (22), HTTP (80)**.

![2.png](/images/2.png)

3.13. 	Click **Review and Launch**.

3.14. 	Review the instance information and click Launch.

3.15. 	Click **Create a new key pair**, enter the **Key pair name (ex. amazonec2_keypair_oregon)**, click **Download Key Pair**.

3.16. 	Click **Launch Instances**.

3.17. 	Scroll down and click **View Instances**.

3.18. 	Wait until **Lab Server** shows 2/2 checks passed in the **Status Checks** column. This will take 3-5 minutes. Use the refresh icon at the top right to check for updates.

### Connect to your Linux instance (From Windows client)

4.1. 	Start PuTTYgen.exe, click **Load**. By default, PuTTYgen display only files with the extension .ppk. to locate your .pem file, select the option to display files of all types.

![3.png](/images/3.png)

4.2. 	Select your .pem file (**ex. amazonec2_keypair_oregon.pem**), and then click **Open**. Click **OK** to dismiss the confirmation dialog box.

4.3. 	Click **Save private key** to save the key in the format that PuTTY can use. PuTTYgen displays a warning about saving the key without a passphrase, click **Yes**.

4.4. 	Specify the same name for the key that you used for the key pair (**ex. amazonec2_keypair_oregon.ppk**). PuTTY automatically adds the .ppk extension.

4.5. 	Start **PuTTY.exe**, enter **Host Name**, Select Lab Server, and copy the **public IP** value.

![4.png](/images/4.png)

4.6. 	On the navigation pane, click **Connect>SSH>Auth**, click **Browse** to choose your key pair (**ex. amazonec2_keypair_oregon.ppk**), click **Open**.

![5.png](/images/5.png)

4.7. 	Enter **ec2-user**, 

![6.png](/images/6.png)

4.8. 	You are successfully connecting to EC2.

![7.png](/images/7.png)


### Connect to your instance (Linux/OSX only)

>Note: This section is for Linux and Max OSX users only. If you are running Windows but have not yet connected to your instance, go back to previous step. If you have already connected to your instance, skip ahead to next step.

5.1. To connect to your EC2 instance, run the following commands in Terminal:
	
	chmod 400  <path and name of pem>
	ssh –i <path and name of pem> ec2-user@<public IP>

* For **path and name of pem**, substitute the path/filename to the .pem file you downloaded.
* For **public IP**, substitute the public IP address for your **Web Server** instance which you copied into a text editor earlier in the lab.


### Install and Setup into Amazon Linux instance

6.1. 	Install NPM into Amazon Linux instance

	[ec2-user ~]$ wget -qO- https://raw.githubusercontent.com/creationix/nvm/v0.33.8/install.sh | bash
	[ec2-user ~]$ . ~/.nvm/nvm.sh
	[ec2-user ~]$ nvm install 6.11.5
	[ec2-user ~]$ command -v nvm
                  //you will see nvm as output, then it should install success

6.2. 	Install GIT and Prepare to deploy lambda

	[ec2-user ~]$ sudo yum install git
	[ec2-user ~]$ git clone https://github.com/awslabs/aws-vpc-flow-log-appender
	[ec2-user ~]$ cd aws-vpc-flow-log-appender/decorator
	[ec2-user ~]$ npm install
	[ec2-user ~]$ cd ../ingestor
	[ec2-user ~]$ npm install
	[ec2-user ~]$ cd ..
	[ec2-user ~]$ aws configure
		//Enter Information as below
	ACCESS_KEY
	Secret_ACCESS_Key
	Region : us-east-1
	Format : json

6.3. 	Deploy Lambda Functions and Create buckets

	[ec2-user ~]$ aws s3 mb s3://YOUR_BUCKET_NAME
	[ec2-user ~]$ aws cloudformation package --template-file app-sam.yaml --s3-bucket YOUR_BUCKET_NAME --output-template-file app-sam-output.yaml
	[ec2-user ~]$ aws cloudformation deploy --template-file app-sam-output.yaml --stack-name vpc-flow-log-appender-dev --capabilities CAPABILITY_IAM


### Set up Firehose

7.1. 	In the AWS Management Console, choose **Kinesis** under Analytics.

7.2. 	Click **Get Started**, choose **Create delivery stream** on Deliver streaming data with Kinesis Firehose delivery streams.

7.3. 	For Delivery stream name, type **VPCFlowLogsToElasticSearch** (the name must match the default environment variable in the ingestion Lambda function). Choose **Next**.

7.4. 	For **Transform source records with Lambda**, choose **Enabled**.

7.5. 	Choose **vpc-flow-log-appender-dev-FlowLogDecoratorFunction-xxxxx** from the **Lambda function** drop-down list (make sure you select the Decorator function). Choose **Next**.

7.6. 	Choose **Amazon Elasticsearch Service** from Destination.

7.7. 	Choose **es-flowlogs** from the Elasticsearch domain drop-down list. (The Amazon ES cluster configuration state must be Active for es-flowlogs to be available in the drop-down list.)

7.8. 	For **Index**, type **cwl**.

7.9. 	Choose **Every day** from the Index rotation drop-down list.

7.10. 	For **Type**, type **log**.

7.11. 	For S3 Backup, choose **Failed Documents Only**.

7.12. 	For Backup S3 bucket, choose S3 bucket name from the drop-down list, or choose Create S3 bucket. Choose **Next**.

7.13. 	Under **IAM role**, choose **Create new, or Choose**.

7.14. 	Choose **Allow**. This takes you back to the Firehose Configuration.

7.15. 	Choose **Next**, and then choose **Create Delivery Stream**.

### Stream data to Firehose

8.1. 	In the AWS Management Console, choose **CloudWatch** under Management Tools.

8.2. 	Choose **Logs** in the navigation pane, and select the check box next to **Flowlogs** under Log Groups.

8.3. 	From the Actions menu, choose **Stream to AWS Lambda**. Choose **vpc-flow-log-appender-dev-FlowLogIngestionFunction-xxxxxxx** (select the Ingestion function).
 Choose **Next**.

8.4. 	Choose **Amazon VPC Flow Logs** from the Log Format drop-down list.

![8.png](/images/8.png)

8.5. 	Choose **Next**.

8.6. 	Choose **Start Streaming**.

Data is now flowing to your Amazon ES cluster, but be patient because it can take up to 30 minutes for the data to begin appearing in your Amazon ES cluster.

### Using the SGDashboard to analyze VPC network traffic

9.1. 	In the AWS Management Console, click **Elasticsearch** Service under Analytics.

9.2. 	Choose **es-flowlogs** under Elasticsearch domain name.

9.3. 	Click the link next to **Kibana**, as shown in the following screenshot.

![9.png](/images/9.png)

The first time you access Kibana, you will be asked to set the *defaultindex*. To set the *defaultindex* in the Amazon ES cluster:

9.4. 	Set the Index name or pattern to **cwl-***.

![10.png](/images/10.png)

9.5. 	For Time-field name, type **@timestamp**.

9.6. 	Choose **Create**.

Load the SGDashboard:

9.7. 	Download this JSON file and save it to your computer. The file includes a dashboard and visualizations. Download Link: https://s3-us-west-2.amazonaws.com/aws-ecv-training/FlowLogDashboard.json

9.8. 	In Kibana, choose **Management** in the navigation pane, choose **Saved Objects**, and then import the file you just downloaded.

9.9. 	Choose **Dashboard** and Open to **load the SGDashboard** you just imported. (You might have to press Enter in the top search box to have the dashboard load the first time.)

9.10. 	The following screenshot shows the SGDashboard after it has loaded.

![11.png](/images/11.png)

## Conclusion

Congratulations! You now have learned how to:

* The VPC posts its flow log data to Amazon CloudWatch Logs.
* The Lambda ingestor function passes the data to Firehose.
* Firehose then passes the data to the Lambda decorator function.
* The Lambda decorator function performs a number of lookups for each record and returns the data to Firehose with additional fields.
* Firehose then posts the enhanced dataset to the Amazon ES endpoint and any errors to Amazon S3.
