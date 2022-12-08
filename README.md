# Build Docker Container for Java App and Deploying it on Amazon EKS
This repo contains a Sample Spring Boot Java App with the dockerfile which uses Amazon Corretto 17 as base image and manifestes for creating an Amazon EKS cluster and deploying the sample app to the cluster as a container and exposing it with a service and classic load balancer.

### Prerequisites
Docker, AWS Account and IAM user with necessary permissions for creating EKS Cluster, aws cli, configure IAM user with necessary programmatic permissions, eksctl cli, kubectl
Please install and configure above before going further

* You can incur charges in your AWS Account by following this steps below
* The code will deploy in us-west-2 region, change it where ever necessary if deploying in another region

After downloading the repo in the terminal CD to repo directory and follow the steps for
1. Building a Docker Image for a Java App and Pushing it to Amazon ECR.
2. Creating an Amazon EKS cluster with eksctl
3. Deploying the sample app to the EKS cluster.
___
### Steps for Building a Docker Image and Pushing it to Amazon ECR
* Change directory to sample
<pre><code>cd sample</pre></code>

* Run docker daemon
<pre><code>sudo dockerd </pre></code>

* Build an image
<pre><code>docker build --tag sample . </pre></code>

* View local images
<pre><code>docker images</pre></code>

* docker build build stage
<pre><code>docker build -t sample-build --target build . </pre></code>

* docker build production stage
<pre><code>docker build -t sample-production --target production . </pre></code>

* Get ECR Login and pass it to docker
<pre><code>aws ecr get-login-password --region us-west-2 | docker login --username AWS --password-stdin Replace-With-AWS-Account-ID.dkr.ecr.us-west-2.amazonaws.com</pre></code>

* Create ECR repo
<pre><code>aws ecr create-repository --repository-name sample-repo --image-scanning-configuration scanOnPush=true --region us-west-2</pre></code>

* Tag the image
<pre><code>docker tag sample-production:latest Replace-With-AWS-Account-ID.dkr.ecr.us-west-2.amazonaws.com/sample-repo</pre></code>

* Push the Image to ECR Repo
<pre><code>docker push Replace-With-AWS-Account-ID.dkr.ecr.us-west-2.amazonaws.com/sample-repo</pre></code>

___
### Create EKS Cluser
Create an Amazon EKS cluster in us-west-2 region with 2 t3.micro instances
Creation of EKS cluster can take up to 20 minutes
<pre><code>eksctl create cluster -f devcluster-addons-us-west-2.yaml</pre></code>

___
### Deploy Image to EKS Cluster 
Update Image URL in deployment.yaml file Replace-With-AWS-Account-ID

* Deploy Java Sample-App
<pre><code>kubectl apply -f deployment.yaml</pre></code>

* Deploy Java Sample-App Service
<pre><code>kubectl apply -f service.yaml</pre></code>

* [Not Necessary] Deploy Java Sample-App ingress for Application Load balancer, you need to install AWS Load balancer controller as per guide here https://docs.aws.amazon.com/eks/latest/userguide/aws-load-balancer-controller.html
<pre><code>kubectl apply -f ingress.yaml</pre></code>


* Get Deployments
<pre><code>kubectl get deployment sample-app</pre></code>

<pre><code>kubectl get deployments</pre></code>

<pre><code>kubectl get service sample-app -o wide</pre></code>

<pre><code>kubectl get pods -n default</pre></code>

___
### Delete Resources 
* Delete Deployments
<pre><code>kubectl delete deployment sample-app</pre></code>

* Delete services
<pre><code>kubectl delete service sample-app</pre></code>

* Delete ingress if you have created it
<pre><code>kubectl delete ingress sample-app</pre></code>

* Delete Amazon EKS Cluster
<pre><code>eksctl delete cluster -f devcluster-addons-us-west-2.yaml</pre></code>