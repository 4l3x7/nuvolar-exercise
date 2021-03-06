[![Build and Push to ECR, Deploy to EKS](https://github.com/4l3x7/nuvolar-exercise/actions/workflows/build.yml/badge.svg)](https://github.com/4l3x7/nuvolar-exercise/actions/workflows/build.yml)
[![pre-commit](https://img.shields.io/badge/pre--commit-enabled-brightgreen?logo=pre-commit&logoColor=white)](https://github.com/pre-commit/pre-commit)

## AL - NuvolarWorks Technical exercise  - Associate DevOps Engineer - Part 2



### Exercise requirements

  

We have a very simple API that we want to deploy into the Cloud. The system consists of three stateless Spring Boot microservices. Docker images for the microservices can be found here:

  

-   https://hub.docker.com/r/nuvolar/api-gateway
    
-   https://hub.docker.com/r/nuvolar/order-service
    
-   https://hub.docker.com/r/nuvolar/customer-service
    

  

The three microservices talk to each other via HTTP. The api-gateway microservice should be accessible publicly (performing an HTTP GET request to /order endpoint should return 200 OK status code).

  

Requirements

  

-   You can create the solution in any language or framework of your choice
    
-   The infrastructure provider can be either Google Cloud or AWS using the technology you prefer
    
-   Have the API exposed over SSL
    

  

It is optional to deploy your solution, in any case, you should provide complete and clear instructions for us to recreate your setup

  

### Tools selected

  

The tools used in this exercise are

  

-   AWS EKS to host a kubernetes cluster
-   eksctl to create the EKS cluster
-   kubectl to manage the kubernetes cluster
-   AWS ECR repository to use as a docker registry
-   GitHub repository with GitHub Actions enabled to make the automated deployment
-   Kustomize to generate the deployment manifest files
    

  

### Process

-   A EKS cluster has been created in AWS with the command ‘eksctl create cluster’ previously connected to the aws-cli credentials
-   Three ECS container registry has been created, one for every service, to host the build images
	-   api-gw
	-   customer-service
	-   order-service
    

-   A [GitHub repository](https://github.com/4l3x7/nuvolar-exercise) has been created with this folder schema
    

	-   .github folder
		-   workflows folder
			-   build.yml > workflow definition for Github actions
	-   api-gw folder
		-   DockerFile > for api-gw microservice, in this case, with only a FROM statement from the dockerhub image
		-   Here must the placed the source code of the micro-service, if exists
	-   customer-service folder
		-   DockerFile > for customer-service microservice, in this case, with only a FROM statement
		-   Here must the placed the source code of the micro-service, if exists
    
	-   order-service folder
		-   DockerFile > for order-service microservice, in this case, with only a FROM statement
		-   Here must the placed the source code of the micro-service, if exists

	-   manifests folder
		-  	deployment.yml > deployments manifest for the micro-services
		-   services.yml > service manifest for the common services, the LoadBalancer, in this case
		-  kustomize.yml > kustomize file
	-   README.md
    

-   AWS secrets have been added to the repository
    
-   EKS_REPOSITORY_NAME secret has been added to the repository
    
-   SSL autosigned certificate has been created and uploaded to AWS IAM to further add it to the load balancer

### GitHub public repository

  

You can see the public repository [here](https://github.com/4l3x7/nuvolar-exercise)

  

### GitHub Actions Workflow

  

The [GitHub Actions workflow](https://github.com/4l3x7/nuvolar-exercise/blob/main/.github/workflows/build.yml) perform this actions

-   Watch for changes in the micro-services or manifest folders, if present triggers the Workflow

-   Build and push to ECR the micro-services images tagged with the sha of the workflow-id
    
-   Create a manifest with kustomize to deploy the microservices in the EKS cluster
    
-   Tests with the /order method with a simple curl
    

  

## How to test the scenario (AWS)

  

There is an EKS cluster with the application deployed, you can see the result of the /order method in the URL

-   [https://af6ee3f7d4ee34e86815b34ca84a3a57-1677496042.us-west-2.elb.amazonaws.com/order](https://af6ee3f7d4ee34e86815b34ca84a3a57-1677496042.us-west-2.elb.amazonaws.com/order)
-   [http://af6ee3f7d4ee34e86815b34ca84a3a57-1677496042.us-west-2.elb.amazonaws.com/order](http://af6ee3f7d4ee34e86815b34ca84a3a57-1677496042.us-west-2.elb.amazonaws.com/order)

It will be interesting for you to check also the outputs of the [GitHub Actions workflows](https://github.com/4l3x7/nuvolar-exercise/actions) to see the complete flow.

## How to test the scenario (GKE)

  

There is a GKE cluster with the application deployed, you can see the result of the /order method in the URL

-   [https://35.241.8.41/order](https://35.241.8.41/order)
-   [http://35.241.8.41/order](http://35.241.8.41/order)

## How to deploy the scenario

If you want, you can deploy the scenario yourself


-   Fork the [repository](https://github.com/4l3x7/nuvolar-exercise)
-   Create an EKS with the command ‘eksctl create cluster’ previously connected to the aws-cli credentials
-   Create in your AWS account and ECS container registry for every microservice, to host the build images. You can user this commands

	-   aws ecr create-repository --repository-name api-gw
	-   aws ecr create-repository --repository-name customer-service
	-   aws ecr create-repository --repository-name order-service
    

-   Add to your forked repository this secrets
    

	-   AWS_ACCESS_KEY_ID
	-   AWS_ACCOUNT_ID
	-   AWS_SECRET_ACCESS_KEY
	-   EKS_CLUSTER_NAME
	-   AWS_DEFAULT_REGION
    

-   Add a dummy file to any microservice folder or change something in the manifest, the number of replicas, for example, to trigger the workflow
-   Commit and Push the changes to GitHub
-   You can see in the Test LB GW step of the deploy-2-eks job the URL of the LoadBalancer to test and the result of the curl. Change the masked *** with your AWS region for this cluster

![alt text](https://github.com/4l3x7/nuvolar-exercise/raw/main/images/kube.png "LoadBalancer HostName")

## Improvements

This was a testing scenario for evaluation purposes but there are several improvements that can be done, if needed:

- Make individual Workflows in order to only build the image of a microservice if the the source of this microservice has been changed, nowadays all the images are built every time
-   Make the workflow branch-dependent and deploy only in pull requests
-   Make a docker scan of the image
-   Lint and make unit testing with the built image isolated
-   Use fluxctl to optimize the manifest generation
