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
* Downloaded the code repo [zip file](files/Stable_Diffusion_OCI_Deployment.zip)
* A Github account
* [Set up Git locally](https://docs.github.com/en/get-started/git-basics/set-up-git)
* An Oracle account
* Generated an SSH key pair and store it in the default directory (`~/.ssh/` on Linux/macOS or `C:\Users\YourUsername\.ssh\` on Windows)
* An OCI compartment. An Oracle Cloud account comes with two pre-configured compartments - The tenancy (root compartment) and ManagedCompartmentForPaaS (created by Oracle for Oracle Platform services).
* The logged-in user should have the necessary privileges to create and manage Compute instances in this compartment. You can configure these privileges via an OCI IAM Policy. If you are using a Free Tier account, it is likely that you already have all the necessary privileges.
* [Install Terraform](https://developer.hashicorp.com/terraform/install)
* [Install and configure OCI CLI](https://docs.oracle.com/en-us/iaas/Content/API/SDKDocs/cliinstall.htm#Quickstart)
* Review Appendix for explanation of the yaml file within the Github Repository

## Acknowledgements
* **Author** - Jason Yan, Enterprise Cloud Architect; Blake Ramos, Enterprise Cloud Architect
* **Last Updated By/Date** - Jason Yan, Enterprise Cloud Architect, February 2025
