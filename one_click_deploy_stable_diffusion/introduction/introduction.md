# Introduction

## About this Workshop

Many companies aspire to harness the power of AI models, such as large language models and diffusion models, but often struggle with where to begin. Oracle Cloud Infrastructure (OCI) offers a wide range of deployment options, including the OCI Generative AI (GenAI) service—a fully managed solution that provides state-of-the-art, customizable large language models for various use cases, including chat, text generation, summarization, and text embeddings.  

However, if you prefer to leverage open-source generative AI models while only paying for the underlying infrastructure, OCI’s high-performance GPU compute makes it easy. By combining OCI with common CI/CD tools, you can seamlessly deploy open-source models to the cloud and integrate them into your existing services.  

In this workshop, you will learn how to rapidly deploy an open-source diffusion model using tools like Terraform, GitHub Actions, and OCI infrastructure. This powerful combination enables a one-click deployment process, allowing you to utilize generative AI models on OCI for your specific use cases with ease.

Estimated Workshop Time: 45 minutes 

### Objectives

* Deploy GPU instance on OCI using Terraform
* Deploy open-source diffusion model application on OCI using GitHub Actions
* Test drive the deployed diffusion model

### Prerequisites

This lab assumes you have:
* [Download the code repo zip file](files/Stable_Diffusion_OCI_Deployment.zip)
* A Github account
* [Set up Git locally](https://docs.github.com/en/get-started/git-basics/set-up-git)
* An Oracle account
* Generate an SSH key pair and store it in the default directory (`~/.ssh/` on Linux/macOS or `C:\Users\YourUsername\.ssh\` on Windows)
* An OCI compartment. An Oracle Cloud account comes with two pre-configured compartments - The tenancy (root compartment) and ManagedCompartmentForPaaS (created by Oracle for Oracle Platform services).
* The logged-in user should have the necessary privileges to create and manage Compute instances in this compartment. You can configure these privileges via an OCI IAM Policy. If you are using a Free Tier account, it is likely that you already have all the necessary privileges.
* [Install Terraform](https://developer.hashicorp.com/terraform/install)
* [Install and configure OCI CLI](https://docs.oracle.com/en-us/iaas/Content/API/SDKDocs/cliinstall.htm#Quickstart)
* Review Appendix for explanation of the yaml file within the Github Repository

## Codebase Overview

This section provides a structured breakdown of essential files and directories, helping you understand their contents and purpose. Read before proceeding to the next section.

### **infra/terraform/main.tf**:
Primary Terraform configuration file that defines the infrastructure resources to be deployed. Specifically, it includes OCI provider info, resource definitions, existing data sources, variables (for ssh public key and compartment id) and output (for displaying instance's public IP after provisioning).

### **infra/terraform/terraform.tfvars.example**:
A template file that defines Terraform variables, specifically `compartment_id` and `ssh_public_key`. Users should copy and rename it to terraform.tfvars, then update values accordingly.

### **.github/workflows/deployment.yml**:
This yaml file is what controls the workflow for the Github action. The job that it runs is a series of sequential steps that:
* Setup SSH connection
* Installs NVIDIA drive and disable Nouveau
* Checks to see if NVIDIA driver was installed after reboot
* Installs Python 3.9 and dependencies
* Runs app.py and outputs the log back into Github Action page

### **src/app.py**:
This python application does the following:
* Checks to see if the GPU is available to use
* Loads Stability AI’s Image to Video diffusion model
* Creates a Gradio Interface
    * Upload photo
    * Processing time
    * Video output
* Launches Gradio app

### **src/requirments.txt**:
This file tells ‘pip’ what python packages and versions to install onto the compute instance. 


## Acknowledgements
* **Author** - Jason Yan, Enterprise Cloud Architect; Blake Ramos, Enterprise Cloud Architect
* **Last Updated By/Date** - Jason Yan, Enterprise Cloud Architect, February 2025
