---
weight: 1
bookFlatSection: true
bookToc: false
title: Introduction
type: docs
---

# Where abouts

## About me

My name is Peter Mostert and I'm a DevOps engineer servicing several AWS-customers. Just being engaged in daily operations doesn't necessarily mean building your experience within DevOps scape. Because I need to recertify for AWS in August I created this self inflicted project in order to document my journey. AWS provided me with small budget to enable me to do this. The budget will also expire in August, this project really took-off May 1st.

## About the project

### Getting focus
You are currently reading the Template front page of the site. It is powered by an t4g, ARM instance on AWS. The instance runs a self-contained Kubernetes cluster (k3s). The objective is to run multiple containers using just one t4g instance. The instance will be deployed in the default VPC. 
This Template content will get enriched (or replaced) with content written in markdown on my GNU/Linux workstation. 

### Using tools
The infra will be deployed by AWS CloudFormation (cfn) in the default VPC. This setup enables to minimize costs (only paying for the instance and outbound traffic as far as I can see) and vendor locking. At the same time we can explore the feature cfn offers. We might introduce a little subproject later to get introduced to terraform which is another (third party) InfraStructure As Code (ISAC) tool. 
The content will be served by one or more nginx containers from within the Kubernetes cluster (which is got deployed within the 1 t4g instance). The site-content will be written in markdown on my local workstation. The content is stored in a git-repo and a CI/CD pipeline will be triggered when the content is pushed. The CI/CD pieline should read the git-repo and then convert it to HTML by suing a too called Hugo. Hugo should deploy the content to a shared storage space (nfs-mount) which can be read by all nginx-containers. 
This environment will be powered by AWS graviton2 architecture (t4g, ARM) which provides more bang for the buck. A single instance K3S setup will be used as the Orchestration tool to serve and manage the containers. A CI/CD pipeline will build and deploy the site-content using a git-repository as its source. In the end the HTML-content should be replaced automatically according to the pushed markdown content written on the local workstation.

### Getting experienced
One of the objectives for this part of the project is to test and get experience with a new workflow. Besides that it provides an excellent opportunity to enhance my DevOps skill set. One or more practical "DevOps certification" courses will be picked as a guideline to test and practice.

