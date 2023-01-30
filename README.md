 The application allows users to create ads with a picture. Ads are stored in a RDS database in AWS. The project is configured to use MySQL, but that can be changed to use a different database (for instance, PostgreSQL). Pictures within ads are stored in a bucket in S3.

Also the application has two different Spring profiles: dev and prod. While running prod, the application gets all properties(username, password, databasename...) to connect to a RDS database using AWS Secrets Manager for security purposes.
 
 
# Prerequisites
Create an RDS Instance
WARNING: These instructions allow you to run and test the application from within your development environment (i.e., without deploying it to AWS) using an RDS instance open to the world, which is something you should avoid in production.

![Screenshot (99)](https://user-images.githubusercontent.com/93249038/215415905-9dbd6b36-bc89-4bcd-8ace-307c12fb3cb6.png)



First, create a security group that will be used to allow ingress connections from outside AWS. Whithin the security group just created, create a new access rule with the following configuration:

Type: MySQL/Aurora
Source: Anywhere
CIDR: 0.0.0.0/0
Then, create an RDS instance, with these properties:

1)Engine MySQL

2)Type dev/test

3)DB Instance Class: t2.micro
 
4)Multizone AZ: no
 
5)DB Instance identifier: springaws (we will provide this as app argument cloud.aws.rds.dbInstanceIdentifier)

6)Master username: springaws (we will provide this as app argument cloud.aws.rds.springaws.username)

7)Master password: > (we will provide this as app argument cloud.aws.rds.springaws.password)

8)Ensure Default VPC is enabled

9)Ensure Publicly accessible is yes

10)VPC security group: choose the security group previously created
Database name: springaws
Launch!!



# Create an S3 bucket
Create an S3 bucket, name it spring-cloud-aws-sample-s3 and give read permissions to anonymous users. Just copy and paste this aws policy to enable anonymous read access:

{
  "Version":"2012-10-17",
  "Statement":[
    {
      "Sid":"AddPerm",
      "Effect":"Allow",
      "Principal": "*",
      "Action":["s3:GetObject"],
      "Resource":["arn:aws:s3:::spring-cloud-aws-sample-s3/*"]
    }
  ]
}

# AWS Secrets Manager
Create a new secret named as: /secrets-app/springaws_prod. You can insert your desired values but if you want to use the previously created RDS database just put the next key-values:

Secret Key	Secret Value
cloud.aws.rds.dbInstanceIdentifier	springaws
cloud.aws.rds.springaws.password	<your_password>
cloud.aws.rds.springaws.username	springaws
cloud.aws.rds.springaws.databaseName	springaws
# To run locally
Some configurations are required in your AWS account for this sample to work. Basically, an S3 bucket (by default spring-cloud-aws-sample-s3 is used, but it can be changed using cloud.aws.s3.bucket property), and an RDS MySQL instance open to the world. Additionally, we need an IAM user with access key and programmatic access to AWS API so that we can access AWS resources from our development machine.

# Create an IAM User
Enable programmatic access
Generate an access key for the user
Give the user the following permissions:
AmazonS3FullAccess
AmazonRDSFullAccess
SecretsManagerReadWrite
To run on EC2
Create an IAM role
Create an IAM role with the following properties:

EC2 role (i.e., a role to be attached to EC2 instances)
Policies:
AmazonS3FullAccess
AmazonRDSFullAccess
SecretsManagerReadWrite
WARNING: It's a good practice to limit policies and not give all available permissions. We're selecting FullAccess as a proof of concept. Real deployment environments must have more restrictive policies.

# Create an EC2 instance
It has been tested with an instance with the following properties:

AMI: Ubuntu 18.04
Type: t2.micro
Storage: 20Gb
Security group: choose or create one with ports 22 and 8080 opened
Attach the IAM role created previously
Once the instance has been started, ssh'd into the machine and issue the following commands:

sudo apt-get update
sudo apt-get install openjdk-8-jre-headless
Then from your own machine, build the jar file and upload it to your EC2 instance:

mvn package -DskipTests
scp -i <your key> spring-cloud-aws-sample-0.2.0.jar ubuntu@<your ec2 ip>:/home/ubuntu/


# Run the application
Locally (dev)
If you have AWS CLI installed in your machine, spring-cloud-aws reads credentials automatically from your machine while trying to use AWS services, but If you don't have it installed, you need to specify the credentials in the application-dev.properties file or pass these as parameters launching the jar file:

cloud.aws.credentials.accessKey="your key"
cloud.aws.credentials.secretKey="your secret"
# If you have AWS CLI you can just run:

java -jar spring-cloud-aws-sample-0.2.0-SNAPSHOT.jar \
	--spring.profiles.active=dev \
	--cloud.aws.rds.dbInstanceIdentifier=springaws \
	--cloud.aws.rds.springaws.password=<your password> \
	--cloud.aws.rds.springaws.username=springaws \
	--cloud.aws.rds.springaws.databaseName=springaws
# On AWS (prod)
If your EC2 instance has the appropriate role (see prerequisites above), and the jar file has been uploaded, and you have created your Secret

java -jar spring-cloud-aws-sample-0.1.0-SNAPSHOT.jar \
--spring.profiles.active=prod
As you can see is not necessary to put database credentials to run the application, it gets the necessary values from AWS Secret Manager.

# Using CloudFormation
To run with CloudFormation is not necessary to create any AWS resources, only secrets. Steps are the following

Insert your secret properties as explained in section AWS Secrets Manager
As parameters to run your stack, you'll need to specify:
Database password
Key Name (for ssh)
Go to your application by click to the link given at the output section of the cloudformation after the stack have been created.                                           ![cloudformation-overview](https://user-images.githubusercontent.com/93249038/215416422-d78740a9-5d56-430d-8245-17b9ea48b15a.png)


# Supported AWS integrations

S.NO    AWS Service	Spring Cloud AWS 2.x	Spring Cloud AWS 3.x
  1       S3	                 ✅	                     ✅
  
  2       SNS	                 ✅	                     ✅
  
  3       SES	                 ✅	                     ✅
  
  4      Parameter Store	 ✅	                     ✅
  
  5      Secrets Manager	 ✅	                     ✅
  
  6          SQS	         ✅	                     ✅
  
  7          RDS	         ✅	                  TODO #322
  

  8        EC2	                 ✅	                     ❌
  
  9      ElastiCache	         ✅	                     ❌
  
  10      CloudFormation	 ✅	                     ❌
  
  
  11       CloudWatch	         ✅	                     ✅
  
 
  12       Cognito	         ✅	           Covered by Spring Boot
  
  13        DynamoDB	         ❌	                      ✅


# Let's have a look at JDBC interceptor  
 
  ![jdbc-retry-interceptor](https://user-images.githubusercontent.com/93249038/215416684-6810c52a-941e-41aa-a7c2-353bd7794f3d.png)
  
# sns Overview 
![sns-overview](https://user-images.githubusercontent.com/93249038/215416774-ec2ff6ce-e4d2-4056-b5b2-7dbae1ff8394.png)

# REFERENCES 

1) https://docs.awspring.io/spring-cloud-aws/docs/3.0.0-M3/reference/html/index.html

2) https://docs.awspring.io/spring-cloud-aws/docs/2.4.2/reference/html/index.html  

3) https://docs.awspring.io/spring-cloud-aws/docs/2.3.5/reference/html/index.html

4) https://docs.awspring.io/spring-cloud-aws/docs/3.0.0-M3/apidocs/index.html

