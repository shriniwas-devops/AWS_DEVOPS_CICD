# AWS_DEVOPS_CICD

PROJECT ARCHITECTURE:

![image](https://user-images.githubusercontent.com/122585172/235337737-128b8d5a-90c9-4d63-a4ef-ce6f0edc9891.png)


Create an AWS Code Pipeline with AWS Code Commit, Code Build & Code Deploy with Amazon Fargate (ECS)

In this project, I have implemented a fully automated CodePipeline which creates containerized application and deployed it to Amazon FARGATE.

AWS Fargate is an Amazon ECS solution that allows you to run containers without managing servers or clusters. Amazon ECS is a fully managed container orchestration service that helps you easily deploy, manage, and scale containerized applications. It integrates with the rest of the AWS platform to provide a secure and easy-to-use solution for running container workloads in the cloud.

This project includes AWS CodePipeline with AWS CodeCommit, CodeBuild, and CodeDeploy. It is a simple web page application with few pictures. Every time you commit new changes in the CodeCommit repository, it creates a new docker image, pushes it to Amazon ECR and then deploys a new container with the latest image to the ECS-Fargate cluster and we can access a newly updated web page.

Stage1 - Migrate a Git repository to AWS CodeCommit
Below are the steps to migrate an existing Git repository to a CodeCommit repository.

#Create a CodeCommit repository
Use the CodeCommit console to create the CodeCommit repository

![image](https://user-images.githubusercontent.com/122585172/235337761-7f5be339-48be-42b2-aa60-46968c40fb09.png)


After it is created, Clone your repository to your local computer. Configure the AWS CLI with a profile by using the configure command. And then run the following command:

git clone https://git-codecommit.us-east-1.amazonaws.com/v1/repos/ECS-CodePipeline-App

#Clone the repository and push it to the CodeCommit repository
Clone a GitHub repository to your local computer, creating what is called a local repo. You then push the contents of the local repo to the CodeCommit repository you created earlier.

From your local computer, run the git clone command with the --mirror option to clone a bare copy of the remote repository into a new folder named github-repo. This is a bare repo meant only for migration. It is not the local repo for interacting with the migrated repository in CodeCommit.We will delete this repo once we copied its content to CodeCommit.

Change directoty github-repo and run the git push command, specifying the URL and name of the destination CodeCommit repository and the --all option

#Check files in CodeCommit


![image](https://user-images.githubusercontent.com/122585172/235337811-31a2b02c-d614-4216-8a3a-dac371f03e15.png)


Stage2 - Configure CodeBuild
#Create Repository in Elastic Container Registry
Go to Amazon Elastic Container Registry console and create a repository.

![image](https://user-images.githubusercontent.com/122585172/235337823-5c77b4fd-1e3e-402f-8464-17b9f9a99bff.png)


This is the repository that CodeBuild will store the docker image in, created from the CodeCommit repository.

#Setup a CodeBuild project
Now, we will configure a CodeBuild project city-pipeline-codebuild to take what's in the CodeCommit repo, build a docker image & store it within ECR in the above repository.

![image](https://user-images.githubusercontent.com/122585172/235337836-6345df8b-0fc4-4862-be35-fdc0c923cf60.png)


![image](https://user-images.githubusercontent.com/122585172/235337847-3b8b19b6-36b9-448a-9de9-40403af2dc5c.png)


AWS_DEFAULT_REGION with a value of us-east-1
AWS_ACCOUNT_ID with a value of your <AWS_ACCOUNT_ID>
IMAGE_TAG with a value of latest
IMAGE_REPO_NAME with a value of your <ECR_REPO_NAME>


![image](https://user-images.githubusercontent.com/122585172/235337870-720a3c56-4d53-4f9c-aff0-907b98597a1c.png)


Make sure IMAGE_REPO_NAME's value should be matching with your ECR repository name.

![image](https://user-images.githubusercontent.com/122585172/235337884-2005e7ed-26a7-4a7e-8b26-0be5439451c4.png)


![image](https://user-images.githubusercontent.com/122585172/235337886-6bb50856-b5e0-4232-b090-f37e11aad19b.png)


#Build security and permissions
Our build project will be accessing ECR to store the resultant docker image, and we need to ensure it has the permissions to do that. The build process will use an IAM role created by CodeBuild, so we need to update that role permissions with ALLOWS for ECR.

Goto IAM console.

![image](https://user-images.githubusercontent.com/122585172/235337896-3a981cd1-f99c-478c-a096-7290bdb57f7f.png)


![image](https://user-images.githubusercontent.com/122585172/235337904-a2e6019a-8562-4d0a-b4e9-823e985b0760.png)


![image](https://user-images.githubusercontent.com/122585172/235337912-9e80c98d-0c0b-4eba-8dfa-2956b7807899.png)


Goto CodeBuild console and select city-pipeline-codebuild and run Build.


![image](https://user-images.githubusercontent.com/122585172/235337925-14c38ef1-550d-4e1e-9c5a-2f3f6c77e5d4.png)


Stage3 - Create a CodePipeline
Whenever a new commit is made to the CodeCommit repository, a new docker image is created and pushed to ECR.

Goto AWS CodePipeline Console and create a new Pipeline with namecity-CodePipeline

![image](https://user-images.githubusercontent.com/122585172/235337956-e89f9e31-5968-41b4-9d5b-705de440c688.png)


![image](https://user-images.githubusercontent.com/122585172/235337968-9645fad9-9ecc-4a4b-b5ce-8d311377e40a.png)


![image](https://user-images.githubusercontent.com/122585172/235337974-ef949ab7-5270-47d9-988d-553b38d53140.png)


Skip Deploy Stage for now. Review and Create Pipeline.


![image](https://user-images.githubusercontent.com/122585172/235337981-c352c796-0c12-433d-a501-5f44db3f703d.png)


The pipeline gets initiated and succeeded as below.


![image](https://user-images.githubusercontent.com/122585172/235337998-69769e51-ca78-4c1b-bf58-6cdcf8bc28d6.png)


Stage4 - Create CodeDeploy
Now you will configure the automated deployment of the city-pipeline application to ECS Fargate.

#Configure a load balancer
Go to the EC2 Console -> then Load Balancing -> Load Balancers -> Create Load Balancer.
Create an application load balancer with name city-pipeline-alb



![image](https://user-images.githubusercontent.com/122585172/235338009-5f9e3cf2-b091-490a-a6d5-395fcc8e98ac.png)


Internet-facing IPv4 For network, select your default VPC and pick ALL subnets in the VPC.

Create a new security group city-pipeline-alb-sg and delete the default VPC in the list Add an inbound rule, select HTTP and for the source IP address choose 0.0.0.0/0 Create the security group.


![image](https://user-images.githubusercontent.com/122585172/235338016-897325e2-2f34-450f-9cec-059d408edabd.png)


Return to the original tab, click the refresh icon next to the security group dropdown, and select city-pipeline-alb-sg and remove the default security group.

![image](https://user-images.githubusercontent.com/122585172/235338034-6ecf0f53-2466-4898-9ac5-1517717d34f8.png)


Under listners and routing make sure HTTP:80 is configured for the listener.'

![image](https://user-images.githubusercontent.com/122585172/235338041-e0598ba4-9bf2-445d-a466-4ce214a769eb.png)


Create a target group, this will open a new tab call it city-pipeline-alb-targetgroup.

Select target type as IP Addresses, HTTP:80, HTTP1 and the default VPC are selected.


![image](https://user-images.githubusercontent.com/122585172/235338052-c23a0836-fc19-497d-9ea7-01851b9dc01f.png)


Return to the load balance tab, hit the refresh icon next to target group and pick city-pipeline-alb-targetgroup from the list. Skip steps for register targets.
Then create the load balancer.


![image](https://user-images.githubusercontent.com/122585172/235338061-20a98cf7-28a3-4a59-a449-41a71decaa50.png)


![image](https://user-images.githubusercontent.com/122585172/235338067-70db912e-3158-4523-a68d-74a321fc9c65.png)

#Create a ECS-Fargate cluster
Move to the ECS console, Create a Cluster with name city-pipeline-ecs-cluster


![image](https://user-images.githubusercontent.com/122585172/235338075-4358c5a2-3a62-432e-a7a7-43930f71a564.png)


![image](https://user-images.githubusercontent.com/122585172/235338084-a9177d18-9620-4c1c-8f1e-5beee07b3b84.png)


#Create Task and Container Definitions
In Container Details, Name put city-pipeline, then in Image URI move back to the ECR console and click Copy URI next to the latest image.

Make sure Container Name city-pipeline should be matched with Amazon ECR repository name city-pipeline.
Scroll to the bottom and click Next

![image](https://user-images.githubusercontent.com/122585172/235338098-cbf62197-a53c-4939-9569-5a8f6bcbeffc.png)


![image](https://user-images.githubusercontent.com/122585172/235338102-f27f1a11-7490-40d5-bd5e-5c61d757c78c.png)


Go to next and configure the environment as mentioned below.

Create ECS-task-role for providing access to AWS Services to run ECS tasks.




![image](https://user-images.githubusercontent.com/122585172/235338109-9f3592a5-3bdb-435e-9c9c-f4d2a201a7bb.png)


![image](https://user-images.githubusercontent.com/122585172/235338114-14845250-3246-4a97-a44e-0b61ef0f1521.png)

Go to next and Create.

#Deploy To ECS - Create a Service'

![image](https://user-images.githubusercontent.com/122585172/235338126-e1ee6bc0-eb77-4fb9-a6d1-abc812efc94d.png)


Create Service with name city-pipeline-service and Launch Type as FARGATE and Desired Tasks 2.

![image](https://user-images.githubusercontent.com/122585172/235338136-461a0487-691d-4c2c-b1bc-d788c487d695.png)


![image](https://user-images.githubusercontent.com/122585172/235338142-4a87603d-d50d-4fb6-a629-a5d31b16dbfd.png)

For Networking pick the default VPC for Subnets make sure all subnets are selected.

![image](https://user-images.githubusercontent.com/122585172/235338151-20d366b9-3eb2-406b-ad51-69bfafb2c643.png)


![image](https://user-images.githubusercontent.com/122585172/235338155-50bc68c3-13d9-4b17-af69-e70a8b4bec39.png)


Keep rest other setting as it is and create Service.


![image](https://user-images.githubusercontent.com/122585172/235338162-7c828135-1ce5-4816-939c-a7da7e9490de.png)


The service is now running with the :latest version of the container on ECR.

#Verify the application on ALB.
Move to the load balancer console and copy ALB DNS.

![image](https://user-images.githubusercontent.com/122585172/235338170-09f6ee5e-5ecf-44d3-88e1-bb2786420172.png)


![image](https://user-images.githubusercontent.com/122585172/235338174-adad6638-edfa-47ec-a3d7-db065138086b.png)


Stage5 - Configure Deploy Stage to Code Pipeline
Goto Code pipeline console. Select city-CodePipeline then edit Click + Add Stage

![image](https://user-images.githubusercontent.com/122585172/235338178-e6ce703a-cca3-43db-ba87-a9d4c948cc1b.png)


![image](https://user-images.githubusercontent.com/122585172/235338182-455539c4-b1eb-483b-9853-a8ed5d52a63f.png)



![image](https://user-images.githubusercontent.com/122585172/235338187-8f997777-7f9b-462e-b07d-b1d18c0a97fa.png)



![image](https://user-images.githubusercontent.com/122585172/235338191-9a380a3a-a5bd-4fc7-b560-b63d3a4b5927.png)


In the local repository edit the index.html file.

git add -A .
git commit -m "test pipeline"
git push


Watch the code pipeline console.

![image](https://user-images.githubusercontent.com/122585172/235338203-903e9632-e4cc-45b7-a580-a905f39f4796.png)


Move to the load balancer console and copy ALB DNS. And open it in the browser.

Congratulation! Your application is now running on ECS-Fargate Cluster. We have automated this deployment through AWS Code Pipeline with AWS Code Commit, Code Build & Code Deploy.

![image](https://user-images.githubusercontent.com/122585172/235338214-3ffdd738-8db4-4d7e-978b-72c077673576.png)


Stage6 - Cleanup
In this stage, you're going to clean up and remove all resources which we created during the session. So that it will not be charged to you afterward.

ECS: Remove Service, Task Definition and Cluster

EC2: Remove ALB, Target Groups

Code: Remove CodePipeline, CodeCommit, CodeBuild

S3: Remove Artefact Bucket

ECR: Remove the repository

