# Introduction

## About this Workshop

Demonstration of running an agent workflow using Lllama3 8B(NIM) as main agent, CodeLlama7B(vLLM) and CuOpt(NIM) as subsidiary agents in OKE using Gradio for the UI.

Running Agent Workflow on Oracle Kubernetes Engine (OKE) is easier than you think. Oracle Cloud Infrastructure (OCI) provides a wide variety of GPU-enabled nodes, both virtual and bare metal, tailored to run as worker nodes in your Kubernetes cluster. These come pre-installed with NVIDIA drivers and device plugin daemonset, simplifying setup.
Agent Workflow running on OCI helps you to configure different AI models securely and automate different workflows in order to simplify them.

Estimated Workshop Time: 3 hours

### Objectives

Objective of this workshop is to deploy a LLM in OKE and setting up an agent workflow.

In this workshop, you will learn how to:

* Configure & set-up JupyterHub Notebook
* Run Agent Workflow on Oracle Kubernetes Engine (OKE)

### Prerequisites

This lab assumes you have:

* An Oracle Cloud account
* Administrator permissions or permissions to use the OCI tenancy
* Ability to spin-up A10 instances in OCI
* Ability to create resources with Public IP addresses (Load Balancer, Instances, OKE API Endpoint)
* Access to HuggingFace, accept selected HuggingFace model license agreement.
* Accept CodeLlama7B HuggingFace model license agreement.
* Access to NGC Catalog for Llama3 and CuOpt NIMs.

## Learn More

* [CodeLlama7B](https://huggingface.co/codellama/CodeLlama-7b-hf)
* [Lllama3 8B](https://huggingface.co/meta-llama/Meta-Llama-3-8B)
* [Nvidia CuOpt](https://build.nvidia.com/nvidia/nvidia-cuopt)
* [HuggingFace](https://huggingface.co/)
* [NGC API Key](https://docs.nvidia.com/ai-enterprise/deployment-guide-spark-rapids-accelerator/0.1.0/appendix-ngc.html)

You may now proceed to the next lab.

## Acknowledgements

**Authors**

* **Ionut Sturzu**, Principal Cloud Architect, NACIE
* **Abhinav Jain**, Senior Cloud Engineer, NACIE