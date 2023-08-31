# Amazon EKS - Building a Cloud Native Application

Lab Link: https://cloudacademy.com/lab/eks-voteapp

## Connecting to the Virtual Machine using EC2 Instance Connect

1. Login to AWS Console
2. In the AWS Management Console search bar, enter EC2, and click the EC2 result under Services.
3. To see available instances, click Instances in the left-hand menu.

## Reviewing Amazon EKS Resources Automatically Created

1. In the AWS Management Console search bar, enter EKS, and click the EKS result under Services:
2. Within the AWS EKS console click on Cluster-1 and review the Cluster info:
3. Within the AWS web console navigate to the AWS EC2 console:
4. Click Instances > Instances on the left-hand side menu: Confirm that the following 3 x EC2 instances are in running status, can take up to 15-25 minutes.

In this lab step, I reviewed and confirmed that all Amazon EKS related resources have achieved their expected operational status.

## Installing Kubernetes Management Tools and Utilities

In preparation to manage your EKS Kubernetes cluster, you will need to install several Kubernetes management-related tools and utilities. In this lab step, you will install.

**kubectl:** the Kubernetes command-line utility which is used for communicating with the Kubernetes Cluster API server.

**awscli:** used to query and retrieve your Amazon EKS cluster connection details, written into the ~/.kube/config file.

1. Download the kubectl utility, give it executable permissions, and copy it into a directory that is part of the PATH environment variable:

   ```bash
   curl -O https://s3.us-west-2.amazonaws.com/amazon-eks/1.24.11/2023-03-17/bin/linux/amd64/kubectl
   chmod +x ./kubectl
   sudo cp ./kubectl /usr/local/bin
   export PATH=/usr/local/bin:$PATH
   ```

   Test the kubectl is installed

   ```bash
   kubectl version --short --client=true
   ```

2. Download the AWS CLI utility, give it executable permissions, and copy it into a directory that is part of the PATH environment variable:

   ```bash
   curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
   unzip awscliv2.zip
   sudo ./aws/install
   ```

   Test the aws utility,

   ```bash
   aws --version
   ```

3. Use the aws utility, to retrieve EKS Cluster name:

   ```bash
   EKS_CLUSTER_NAME=$(aws eks list-clusters --region us-west-2 --query clusters[0] --output text)
   echo $EKS_CLUSTER_NAME
   ```

4. Use the aws utility to query and retrieve your Amazon EKS cluster connection details, saving them into the ~/.kube/config file. Enter the following command in the terminal:
   aws eks update-kubeconfig --name $EKS_CLUSTER_NAME --region us-west-2

5. View the EKS Cluster connection details. This confirms that the EKS authentication credentials required to authenticate have been correctly copied into the ~/.kube/config file. Enter the following command in the terminal:
   cat ~/.kube/config

6. Use the kubectl utility to list the EKS Cluster Worker Nodes:
   kubectl get nodes
   Ensure the cluster's node group is ready and reissue the command periodically.

7. Use the kubectl utility to describe in more detail the EKS Cluster Worker Nodes:
   kubectl describe nodes

## Cloud Native App Architecture Review

### Architecture Review

The cloud native application is architected using containers and is presented to the end-user as a web application accessible over the Internet. The application frontend provides the end-user with the ability to vote on one of 6 programming languages: C#, Python, JavaScript, Go, Java, and/or NodeJS. Voting activity within the browser generates AJAX requests being sent to an API which in turn then saves the results into a MongoDB database backend.

!['Architecture'](/img/app.png)

The architecture of the cloud native application will expose you to configuring, deploying, and managing the following Kubernetes resources:

- Namespace
- Secret
- Deployment
- Service
- StatefulSet
- PersistentVolume
- PersistentVolumeClaim

#### Application Architecture

The EKS cloud native application architecture consists of 3 main components, the Frontend, an API, and a MongoDB database.

The following sequence diagram shows the messaging as used within the application when the application first loads. Key points of this diagram are:

- The browser loads the main application HTML, Javascript, and CSS files
- Each ProgrammingLanguage component makes an AJAX call to the /languages/{name} API endpoint to retrieve the details about the programming language keyed on programming language name
- The API in turn connects to the backend MongoDB database and performs a db.collection.find filtered on the programming language name to return the document that contains all the details about the programming language
- A response containing the programming language then flows back all the way to the browser
- The browser renders the full application view.

  !['Application'](/img/arch-main.png)

  The following sequence diagram shows the messaging as used within the application when an end-user clicks on the voting +1 button for a particular programming language. Key points of this diagram are:

  !['AppFlow'](/img/api-flow.png)

- Clicking the vote +1 button on any one of the 3 programming languages results in an AJAX call being made from within the Vote component to the API endpoint /languages/{name}/vote.
- The API in turn connects to the backend MongoDB database and performs a db.collection.updateOne filtered on the programming language name and increments the vote field on the respective document.
- An update success code is returned in the response and is passed all the way back to the originating AJAX call
- The browser renders and increments the respective Votes div within the Vote component

1. **Frontend Configuration**
   The Frontend is developed in React. For the purposes of this lab, the Frontend has already been compiled and packaged into an existing container image (cloudacademydevops/frontend:v10).
   !['frontend'](/img/arch-web.png)
   The Frontend will be deployed using the following Kubernetes resources:

   **Deployment** - is used to deploy and launch 2 x pods containing the Frontend configured to listen on port 8080
   **Service** - of type LoadBalancer is created to provide a stable virtual IP or VIP to sit in front of the 2 x pods containing the frontend. The Service once deployed into EKS will result in a public facing ELB being created. Externally the Service listens on port 80 and forwards traffic inward to port 8080
   Note: The frontend Deployment resource will be configured with the API's ELB FQDN - this will allow the frontend UI to send AJAX requests back to the API.

   **Source code:** https://github.com/cloudacademy/voteapp-frontend-react-2020

2. **API Configuration**
   The API is developed in Go. For the purposes of this lab, the API has already been compiled and packaged into an existing container image (cloudacademydevops/api:v2).

   !['backend'](/img/arch-api.png)
   The API will be deployed using the following Kubernetes resources:

   - **Deployment** - used to deploy and launch 2 x pods containing the API configured to listen on port 8080
   - **Service** - of type LoadBalancer is created to provide a stable virtual IP or VIP to sit in front of the 2 x pods containing the API. The Service once deployed into EKS will result in a public facing ELB being created. Externally the Service listens on port 80 and forwards traffic inward to port 8080
     The API implements the following CRUD based public endpoints:

   ```bash
   GET /languages
   GET /languages/{name}
   GET /languages/{name}/vote
   GET /ok
   POST /languages/{name}
   DELETE /languages/{name}
   ```

   **Source code:** https://github.com/cloudacademy/voteapp-api-go

3. **Database Configuration**
   A MongoDB NoSQL database is used for the persistence layer. MongoDB provides a feature rich schemaless or NoSQL approach to storing data. The main benefit of using this type of database is that you can store documents or objects in their native data structures without any translation or normalisation required. This type of database is ideal when creating APIs that deal with JSON formatted data - which is the main reason we are using it for managing the data that the API uses. For the purposes of availablity, the MongoDB database will be configured as a MongoDB Replicaset consisting of a single Primary and 2 x Secondary nodes.

   !['database'](/img/arch-db.png)
   The MongoDB database will be deployed using the following Kubernetes resources:

   - **StatefulSet** - used to deploy and launch 3 x pods containing the MongoDB service configured to listen on port 27017
   - **Service** - headless, will be created to sit in front of the StatefulSet, creating a stable network name for each of the individual pods as well as for the StatefulSet as a whole
   - **Persistent Volume (PV)** - x3, 1 for each individual pod in the StatefulSet - MongoDB will be configured to persist all data and configuration into a directory mounted to the persistent volume
   - **Persistent Volume (PVC)** - x3, 1 for each PV, binds a PV to a Pod within the MongoDB StatefulSet

## Deploying a Cloud Native Application

1. Begin by recreating the cloudacademy namespace, which will be used to contain all of the cluster resources that will eventually make up the sample cloud native application.

2. Configure the cloudacademy namespace to be the default.

3. Deploy and configure Stetefulsets, a headless services,,PV and PVC for MongoDB.

4. Deploy and configure Deployments and Services for APIs.

5. Deploy and configure Deployments and Service for frontend web app.

6. Bonus steps:

- Use the Developer Tools within the browser to examine the AJAX calls that are being generated by the voting application when votes are made
- Within the AWS console, examine the ELBs that were created as a result of deploying the Frontend and API Services into the EKS cluster
- Try scaling up and down the Frontend deployment
- Try scaling up and down the API deployment
- Using the mongo shell, examine the replicated data copied into the Mongo Secondary nodes (mongo-1 and mongo-2) within the replica set

**Summary**

In this Lab Step, I learnt how to deploy a cloud native application into EKS. Once deployed and up and running, you used your local workstation's browser to test out the application. You later confirmed that your activity within the application generated data which was captured and recorded successfully within the MongoDB ReplicaSet back end within the cluster.
