#Devops Automation using Jenkins into AWS ECS with Fargate between Non-Prod and Production Accounts

### Whats covered in this guide,

The following tasks will be performed:
> ####Enable the Cross Account policy in AWS for (Dev, QA and Prod)
####Launch the EC2 instance with Jenkins AMI.
####Setup the ECS Cluster
####Create the New-task Defenition in ECS
####Create a Load balancer and Service in ECS
####Configure the Jenkins Jobs for Deployment
####Automate the Build process in Jenkins

###Prerequisites
- IAM Policies 
- IAM Cross Account Roles
- Jenkins with required Plugins
- AWS ECS Cluster (Fargate)
- Bitbucket
- Docker Hub 
- S3 Bucket
- ECS CLI

###What is Amazon ECS?
Amazon Elastic Container Service (ECS) is a highly scalable, high performance container management service that supports Docker containers and allows you to easily run applications on a managed cluster of Amazon EC2 instance.
###What is AWS Fargate?
AWS Fargate is a technology that you can use with Amazon ECS to run containers without having to manage servers or clusters of Amazon EC2 instances. 
###What is Bitbucket?
Bitbucket is a central place to manage git repositories, collaborate on your source code and guide you through the development flow.
###What is Docker Hub?
Docker Hub is a cloud-based repository for Docker users. It allows to create, test, store and distribute container images.
###What is Jenkins?
Jenkins is an open source automation tool for Continuous Integration purpose, and it is used to build and test your software projects continuously making it easier for developers to integrate changes to the project.

##Lets start the Walkthrough,

First, We need to enable the cross-account roles to connect three accounts, so that Jenkins has deployed the docker images in ECS which has launched in QA, Prod Accounts. 

###Step1: Create a Cross Account Role (Dev to QA) Account

1. Sign into the** Dev Account** with administrator Privileges.

2. GO to **IAM -> Roles -> Create Role -> Another AWS Account** 

3. Enter the **QA Account ID <0123456789>**

![](https://github.com/Vahinvishnu/eks-workshop-sample-api-service-go/blob/master/Step1.png?raw=true)

Select the **Administrator Privileges** and Click **Create Role**

![](https://github.com/Vahinvishnu/eks-workshop-sample-api-service-go/blob/master/2.png?raw=true)

###Step 2: Create a New user in QA Account with ECS privileges



1. Login into the QA Account 

2. GO to** IAM -> Users -> Add User**
3. Attach the  **AmazonECS_FullAccess Policy** and **Create User**. 

4. Copy the **user arn** and paste it into the Dev Account Role (Trust Relationships).

![](https://github.com/Vahinvishnu/eks-workshop-sample-api-service-go/blob/master/3.JPG?raw=true)

###Step 3:  Create a IAM Role to acces the ECS via Jenkins

1. Login to the Dev Account

2. Go to IAM -> Roles  -> EC2 -> Administrator Access -> Create Role.

3.  Edit the trust relationshios in created Role and update the arn.

![](https://github.com/Vahinvishnu/eks-workshop-sample-api-service-go/blob/master/4.JPG?raw=true)

###Step 4: Launch a New EC2 instance using an Ami  "Jenkins on Ubuntu"

![](https://github.com/Vahinvishnu/eks-workshop-sample-api-service-go/blob/master/29.png?raw=true)

###Step 5: Setting Up Jenkins

To set up our installation, we need login to Jenkins on its default port, 8080, using the server domain name or IP address: http://ip_address_or_domain_name:8080

We should see “**Unlock Jenkins**” screen, which displays the location of the initial password

![](https://github.com/Vahinvishnu/eks-workshop-sample-api-service-go/blob/master/28.png?raw=true)

###Step 6: Open the terminal window of Jenkins server, use the cat command to display the password

![](https://github.com/Vahinvishnu/eks-workshop-sample-api-service-go/blob/master/27.JPG?raw=true)

Copy the 32-character alphanumeric password from the terminal and paste it into the “**Administrator password**” field, then click “**Continue**”. 

![](https://github.com/Vahinvishnu/eks-workshop-sample-api-service-go/blob/master/26.png?raw=true)


Finally we got an Jenkins Dashboard. Now we create the jobs and configure it. 

###step 7: Install and configure Jenkins plugins

To build environment is to install and configure the Jenkins plugins required to build a Docker image and publish it to a Docker registry

Go to **Jenkins** >  **Manage Jenkins** > **Manage Plugins**

Search the below plugins and Install it. 

- **CloudBees Docker Build and Publish plugin**
- **CloudBees AWS Credentials Plugin**
- **Credentials**
- **S3 publisher**
- **Pipeline: Multibranch**
- **dockerhub plugin**
- **Github plugin****

###Step 8: Open the Jenkins server in CLI and Configure the user and ECS CLI.

Create the user as Jenkins in the Jenkins Instance.

Jenkins can use the ECS CLI. Switch to the jenkins user and configure the AWS CLI, providing your credentials

![](https://github.com/Vahinvishnu/eks-workshop-sample-api-service-go/blob/master/25.png?raw=true)

###Step 9: Login to the Docker Hub

Jenkins user needs to login to Docker Hub before doing the first build.
 
![](https://github.com/Vahinvishnu/eks-workshop-sample-api-service-go/blob/master/24.png?raw=true)
 
#Setup the ECS Cluster in AWS 

Create an ECS Cluster (**Powered by AWS Fargate**) with Container insights enabled.

Goto AWS Console > **ECS** > **Create Cluster **> Select the Cluster template as"**Networking Only**"> **Cluster name** > **VPC **> **Enable container insights** > Click **Create**

![](https://github.com/Vahinvishnu/eks-workshop-sample-api-service-go/blob/master/23.png?raw=true)


#Setup the Task Definition and Services Using ECS CLI

###Step 1: Create a Task definition using below template

Open the Jenkins Instance in putty and run the below scripts.

Create a task definition template for your application

**Note: you will replace the image name with your own repository**

![](https://github.com/Vahinvishnu/eks-workshop-sample-api-service-go/blob/master/22.JPG?raw=true)

Save your task definition template as ***1cloudhub.json***

"The image specified in the above template will be built in the Jenkins job, at this time we will create a dummy task definition and the %BUILD_NUMBER% parameter in your task definition template with a non-existent value (0) and register it with ECS Service."*(refer task-def.json)*

![](https://github.com/Vahinvishnu/eks-workshop-sample-api-service-go/blob/master/21.JPG?raw=true)


*Note the family value **(1cloudhub)** in the above script, it will be needed when we configuring the execute shell step in the Jenkins job.*

#Create an Load Balancer in AWS console

Create an Elastic load balancer to be used in your service definition and note the ELB name as (e.g. 1ch-ecs-LB)

Goto AWS console > EC2 >** Load balancer** > **Create Load Balancer** > Select **Application Load Balancer** > fill the **name, listners, security group, vpc and routing** > Click **Create**

#Create an ECS IAM Role.

Goto AWS Console 

Select **IAM** >** Role **> **Create New Role** > Attach the **Amazon EC2 Container Service Role** as Policies in the created Role.

*Attach the created Role to Jenkins instance.This will allows ECS to create and manage AWS resources, such as an ELB, on your behalf.*

#Create an ECS Services using CLI

Open the Jenkins Instance in putty  and run the below Commands

Create the Service named as **“1ch-demo-service”** specifying the task definition (e.g. 1cloudhub) and the ELB name (e.g. 1ch-ecs-LB) as below. *(Refer ecs-service.json)*

![](https://github.com/Vahinvishnu/eks-workshop-sample-api-service-go/blob/master/7.JPG?raw=true)

Here we have not yet build a Docker image for our task, so in the above configuration the **desired-count flag is set to 0.**

#Configure the Jenkins build

On the Jenkins dashboard, click on **New Item**, select the **Freestyle project job**, add a **Name** for the job, and click **OK. **

##Configure the Jenkins job:

- Open the **Bitbucket**, and **clone** the repository URL as, e.g - https://bitbucket.com/1cloudhub/website.git
In addition to the application source code, the repository contains the Dockerfile used to build the docker image.
- Under **Source Code Management **provide the Repository URL for Git, e.g. https://bitbucket.com/1cloudhub/website.git and select the branch as **Master.**
- In the **Build Triggers section** select * Build when a change is pushed to GitHub.*
- In the **Build section** add a **Docker build and publish** step to the job and configure it to publish to your Docker registry repository 
*(e.g. DockerHub) and add a tag to identify the image (e.g. v_$BUILD_NUMBER).*

![](https://github.com/Vahinvishnu/eks-workshop-sample-api-service-go/blob/master/8.png?raw=true)


The Repository Name specifies the name of the Docker repository where the image will be published.

It has composed of a username **(2256454)** and an image name **(1cloudhub).**

**Note ** the repository name needs to be the same as what is used in the task definition template in ***1cloudhub.json.***

**Add a Execute Shell step and add the ECS CLI commands to start a new task on your ECS cluster.**

![](https://github.com/Vahinvishnu/eks-workshop-sample-api-service-go/blob/master/9.png?raw=true)

The script for the Execute shell step will look like this, 

Save this Job and click **Build** in Jenkins project. *(refer new-task.json)*

![](https://github.com/Vahinvishnu/eks-workshop-sample-api-service-go/blob/master/10.JPG?raw=true)





#Automate the Build Process

To trigger the build process on Jenkins upon pushing to the Bitbucket repository we need to configure a service hook on Bitbucket.
 Go to the Bitbucket repository **settings** page, select **Webhooks and Services **and add a service hook for Jenkins (GitHub plugin). 
Add the Jenkins hook url: **http://<!EC2-DNS-Name>:<port>/bitbucket-webhook/**

![](https://github.com/Vahinvishnu/eks-workshop-sample-api-service-go/blob/master/11.png?raw=true)

This will trigger the Jenkins job. After the job is completed, point your browser to the public DNS name for your ECS Fargate Service and verify that the application is correctly running.

###Advantage of this Approach
- No nodes/servers to manage.
- Launch 10+ builds/containers in seconds.
- Run the containers directly, without any ec2 instances.













