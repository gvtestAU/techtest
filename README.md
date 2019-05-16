# Intro
This repo will allow you to deploy the application hosted at https://github.com/vibrato/TechTestApp on AWS using ec2 and AutoScaling to serve the frontend and RDS on Postgres to serve the frontend.

The EC2 instances will compile and serve the latest commit on the master branch.

The database is bootstrapped once and data retained across version bumps and frontend stop/starts.

# How to deploy
IMPORTANT: you MUST use the ap-southeast-2 AWS region when deploying this Cloudformation Stack

Create a Cloudformation Stack using the file Stack.yml as template source. You can name the Stack as you wish. You can either use the AWS CLI or the AWS Web console to create the Stack.

The template supports 3 parameters:
- EnvironmentName: a unique name to identify the stack resources. Defaults to dev
- AppServerNodes: the number of frontend instances to spin and load balance. Defaults to 2.
- AppInstanceType: the ec2 type of the frontend instances. Defaults to t2.micro.

Wait for the stack to fully create and grab the AWS Application Load Balancer host from the "Outputs" under the "ClusterEndpoint" Key.

It will take a couple of minutes to then have the AutoScaling group to spin the necessary ec2 instances and serve the application. Point your browser to the value of the "ClusterEndpoint" Cloudformation Stack output, port 80.

# Assumptions

The template has few but important assumptions you need to be aware of:
- Althogh the template is almost completely AWS region agnostic, it MUST be deployed to the ap-southeast-2 region. If you want to deploy it on another region, you MUST add the Ubuntu 18.04 AMI to the AMIRegionMap mapping in the template.
- The template assumes that the region you deploy it in, have AZ 'a' and AZ 'b'. Although this is true to almost all cases, there is a chance some regions may not.
- the RDS type instance is fixed at db.r4.large

# Advanced and additional information
- the RDS secret is managed by the AWS Secrets Manager and created within the Cloudformation Stack. The ec2 instances transparently consume this secret and configures the frontend properly.
- the only endpoint open to the Internet is the Application Load Balancer serving the frontend, on port 80. All ec2 instances and the RDS Cluster Instances are within a separate subnet that is protected from any inbound Internet access.
- the ec2 instance bootstraps the necessary component once during its lifetime via the AutoScaling LaunchConfiguration UserData

# What to do next:

The Cloudformation template presented here is a good start to automate the deployment, monitoring and maintenance of this test app. However I suggest the following items to be added to further improve the solution:
- logging to AWS Cloudwatch on the ec2 instances
- consider AWS Fargate to serve the frontend
- a proper CI environment separated from the environment itself
- use ECR to host and serve docker registries
- serve the frontend via HTTPS using an ACM cert on the ALB

