
# Answer to Question 1

## Step1) Download below three yaml file on minikube and 

## Step2) run kubectl create -f . 

db-tier.yaml

secrets.yaml

nginx-ingress.yaml

app-tier.yaml

### Step3) To access the frontend applocation run curl http://localhost/wordpress




# Answer to Question 2

I will setup three tier architecture in AWS and below is the list of AWS services I will use:

1) CloudFront
2) Route53
3) Ec2 Instance
4) Autoscaling Group (ASG)
5) Application Load Balancer (ALB)
6) ElastiCache
7) Aurora or RDS Database
8) CloudWatch
9) S3 Bucket
10) WAF

![alt text](https://github.com/rsthakur83/carnext/blob/master/Three%20Tier%20Architecture.jpg "Three Tier Architecture")


Create VPC which has public, private subnet. Public subnet consists of web tier while in private subnet application server and database exists.
Initially user request will come to Application load balancer (ALB) which is attached to the first tier of Web server Autoscaling Group (ASG can scale automatically in case of high resource utilization of servers in ASG or through any other custom metric) and this tier responsible to publish web content to remote user. And then comes second tier (Application Tier) which consist of internal application load balancer and another ASG, traffic from first tier comes to internal ALB which then redirect traffic to the  instances in application tier ,which has business logic and this tier talk to third database tier directly. Final & last tier is database and we can use RDS or aurora here which is AWS managed database service. To improve the performance of database tier, create multiple read replica as per the requirement.
For the fast delivery of static content use the AWS CloudFront service along with S3 bucket to publish the static content, add the DNS entry in the route53 for CDN.
To further improve the application tier performance, use the ElastiCache service which will cache the database content, frequent access data, access pattern.
ASG scale up & down the instance whenever alarm get triggered. CloudWatch monitor the metric like CPU, disk, memory etc. and whenever metric breach the threshold limit it trigger the alarm which take the respective action on ASG.
To fight against the malicious web request, attach the WAF service with Cloudfront and add the custom rule in WAF based on your requirement.


# Answer to Question 3

To migrate the web application running on premise to AWS follow below steps:

1)	Establish the connectivity between on-premise to AWS using VPN or AWS direct connect.
2)	Once the VPN connection established upload the content to S3 directly OR use AWS data sync service to transfer the data from on-premises to AWS.
3)	Use the three tier architecture, first tier ALB --> Web Tier (ASG) , Second Tier Internal ALB --> Application server (ASG). Deploy the application uploaded (S3 bucket) onto the ec2 instance in second tier.
4)	Transferring application data is easy but main challenge comes when we want to do database migration. For the DB migration either we can take the database dump and upload onto s3 bucket then ec2 to restore onto to the RDS instance using mysqldump command. But in case of smooth migration AWS provide DMS service which we can use to transfer large amount database on to AWS. This service help in migrate large size of database without any downtime and with less effort, also can change the schema of source DB to destination DB.
Let’s consider that we have master & slave database on premises which we want to migrate to AWS. First migrate the slave database using DMS service to AWS once the slave DB instance in complete sync with on premise RDS instance (named as M2) and make M2 as slave of on premises master database (named as M1). Once the M2 binlog is same as that of M1 database then create the slave S2 instance from M1. Now we have master and slave database in sync with On premises database’s, it’s time to do the switchover the traffic without downtime. Begin the switchover by moving read only traffic first by pointing one of the app server to the slave S2. Since slave is read-only, it offers easy switchback to the source DB without data loss in case of an issue. After careful testing, you can switch all remaining read-only APP servers to the slave. Use some sort of queue software like Kafka, RabbitMQ to store the incoming request during the switchover period to M2 database so that these requests can be processed to the new database. Following switching read traffic to S2, you can begin read-write traffic switch to M2 in the same manner. The benefit of this approach is since the data is replicated in both ways you can easily switch back to the source DB without losing any data in case of an issue.


