# Introduction

## About this Workshop

This workshop walks you through provisioning the necessary infrastructure on Oracle Cloud Infrastructure (OCI) and deploying virtual machines to configure NVIDIA DCGM (Data Center GPU Manager) with Prometheus and Grafana. The lab involves setting up two GPU-enabled VMs (Oracle Linux and Ubuntu) for GPU activity generation and monitoring, alongside a separate VM for Prometheus and Grafana. Public subnet access is configured to enable connectivity, and GPU activity is generated to visualize performance metrics in real time. By the end of the guide, you will have a fully operational environment for monitoring GPU metrics using Prometheus and Grafana dashboards.

Estimated Workshop Time: 2 hours

### Objectives

The objective of this workshop is to provide hands-on experience with deploying and demonstrating GPU performance monitoring using NVIDIA DCGM, Prometheus, and Grafana on OCI.

By the end of this workshop, you will:

* Provision and configure GPU-enabled and simple VMs on OCI for monitoring with NVIDIA DCGM.
* Set up Prometheus and Grafana to collect and visualize GPU metrics.
* Gain experience using automation tools like Ansible to streamline the setup of NVIDIA DCGM. 
* Generate GPU activity for monitoring dashboards and review these results using Grafana.

### Prerequisites

This lab assumes you have:

* Access to an Oracle Cloud account
* Sufficient permissions to create and manage Compute Instances, especially access to GPU-enabled shapes (shape "VM.GPU.A10.1" must be available).
* Ability to create resources within a public subnet with Internet Gateway access.
* Basic familiarity with Linux commands, Prometheus, and Grafana is recommended but not mandatory.

## Learn More

* [NVIDIA DCGM](https://developer.nvidia.com/dcgm)
* [DCGM Github Repo](https://github.com/NVIDIA/DCGM)

You may now proceed to the next lab.

## Acknowledgements

**Authors** 
* Adina Nicolescu, Senior Cloud Engineer, NACIE
* Francisc Vass, Principal Cloud Architect, NACIE

**Last Updated By/Date**
* Adina Nicolescu - Senior Cloud Engineer, NACIE - Dec 2024