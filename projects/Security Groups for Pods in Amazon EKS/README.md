# Using Security Groups for Pods in Amazon EKS

Containerized applications frequently require access to other services running within the cluster as well as external AWS services, such as Amazon Relational Database Service (RDS)

On AWS, controlling network-level access between services is often accomplished via **security groups**.

Early versions of EKS only allowed you to assign security groups at the node level. Because all nodes inside a node group share the security group, by attaching the security group to access the RDS instance to the node group in the image below, all the pods running on these nodes would have access to the database even if only the green pod should have access:

![SG-architecture](/projects/Security%20Groups%20for%20Pods%20in%20Amazon%20EKS/img/sg-1.png)

Security groups for pods integrate Amazon EC2 security groups with Kubernetes pods. You can use Amazon EC2 security groups to define rules that allow inbound and outbound network traffic to and from pods that you deploy to nodes running on many Amazon EC2 instance types. In this lab, you will learn how to integrate security groups with an EKS cluster to access an RDS database and demonstrate the effect of Pod security groups as shown in the following environment diagram:

![SG](/projects/Security%20Groups%20for%20Pods%20in%20Amazon%20EKS/img/sg-2.png)

## Learning Objectives

Upon completion of this lab, you will be able to:

- Configure Amazon EKS node groups to use trunk network interfaces to enable pod security groups
- Use SecurityGroupPolicy custom resources to select pods that are associated with security groups
- Identify network interfaces protected by security groups
- Demonstrate the effects of security groups per pod

## Prerequisites

Familiarity with the following will be beneficial but is not required:

- Basic Linux command line administration
- Basic Kubernetes and Container-based concepts
- Familiarity with AWS resources including VPCs and security groups
