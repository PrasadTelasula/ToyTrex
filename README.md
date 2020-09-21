# ToyTrex
ToyTrex Assignment

## Q3 - SCENARIO
````bash
To explain the scenario first we will go through the EKS cluster architecture.

For applications to run 24/7 with highly available and Fault tolernt the underlaying infrastructure will play a key role. If inftrastructure not planned well application may face down time.

Below diagram will illustrates the best practises that need to be taken care. while desgining the infrasturcture.
1. VPC with 6 Subnets.
    1.1 two Public Subnets (for web tier)
    1.2 two Application Subnets (for middleware tier)
    1.3 two Database Subnets (backend tier)
2. Subnets should be created in at least two availability zones. [i.e, ap-south-1a & ap-south-1b]
````

# EKS AWS Cluster Architecture 
![Alt text](https://github.com/PrasadTelasula/ToyTrex/blob/master/img/Infrastructure.png?raw=true "Architecture")


# ToyTrex Application Architecture 
````bash
As per the given scenario, application should be deployed in kubernetes as a microservices. Below diagram will illustrates the application architecture.
1. web layer should be deployed as a Deployment with desrired replica count. [below diagram representing replicas=3]
2. Middleware layer should be deployed as a Deployment with desried number of replica count [below diagram representing replicas=3]
3. Database should be created using AWS RDS instance. [Below diagram representing Mariadb RDS instance]
````

# Communication
````bash
# To establish the communication between web layer pods to the middleware pods
A service should be created with type ClusterIP. 

# To establish the communication between middleware pods to the backend RDS database
A service should be created with type ExternalIP

# To expose the application to the end customers
Weblayer pods should be exposed as a NodePort service.
````

# High Availability
````bash
- Node level high availabilty will be taken care by the EKS cluster with the help of AutoScaling feature in AWS.
- To achive the Pod level high availability.
    - we have to deploy metric server inside the kubernetes cluster.
    - Based on the metrics collected by the metric server we can enable HPA object [Horizontal Pod Autoscaler]. By leverging HPA we can scale pods based on the demand. [Scale-out & Scale-in]
````

# Enable Health Checks.
````bash
- Pods need some time to be availble to accept the requests from services.
- By default servcies can't check the Health of the pods. Blindly it will send the requests to the pods. In this case while scaling/restarting the pods, end users may face 503 errors.
- To avoid this we should implement the Liveness & Readiness Probes while deploying the objects.
  - Readiness Probe: Sometimes, pods are temporarily unable to serve traffic. For example pods need to load large data or configuration files during startup, or depend on external services like RDS after startup. In such cases we shouldn't kill the Pod, but we don't want to send it requests either. Kubernetes provied Readiness probes to detect and mitiagate these situations.
  - Liveness Probe: Many pods running for long periods of time eventually transition to broken status, and cannot recover except by being restarted. Using Liveness probes we can detect and remedy such situations.
````

# Expose application
````bash
# Ingress
Ingress will provide load balancing, SSL termination and name-based virtual hosting. It exposes HTTP and HTTPS traffic routes from outside world to the cluster to services within the kubernetes cluster. Traffic routing is controlled by rules defind in the Ingress Resource.

# ALB Ingress 
Application loadbalancer should be pointed to the NodePort service.

# Domain Registrtaion.
Using AWS Route53 product domain can be registered to the Loadbalancer. 

# ACM [AWS Certificate Manager]
To Implement SSL for the registered domain, aws is providing SSL certificates. we can generate one SSL certificate with our domain name and the same can be used in the Loadbalancer configuration at the 443 Listener.

Example: https://toytrex.com
````
![Alt text](https://github.com/PrasadTelasula/ToyTrex/blob/master/img/Kubernetes.png?raw=true "Architecture")

# Enable DevOps
````bash
To relase the new application releases faster we can enable DevOps using AWS services.
EKS service is tighlty coupled with other AWS services. 

CICD process can be achivied with below AWS services.
- CodePipeline
- CodeCommit
- CodeBuild 
- CodeDeploy
````

![Alt text](https://github.com/PrasadTelasula/ToyTrex/blob/master/img/CICD-Process.png?raw=true "CICD")

## Above diagram will illustrate the CICD process.
- Step 01: Developer will push the code to CodeCommit.
- Step 02: CodePipeline will be triggered based on CodeCommit event.
- Step 03: CodePipeline will trigger the CodeBuild to build the DockerImage based on the build instructions, and it will push image to ECR.
  * All build instructions should be maintained in buildspec.yml file.
  * Docker image should be pushed to ECR in the format of <image_name>:<build_number> 
- Step 04: Once build process over. CodePipeline will trigger the CodeDeploy to deploy the objects in the cluster.
  * ConfigMap object should be created.
  * It will allow CodeDeploy to configure the kubectl to deploy the objects in the cluster.
- Step 06: CodeDeploy will deploy the objects in the EKS cluster.
  * EKS cluster will pull the DockerImage from The ECR registry.
  * CodeDeploy will deploy the manifests in to the EKS cluster to deploy the application stack.
  
  
  
           





