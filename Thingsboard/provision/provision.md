# Provisioning of Thingsboard instance

## Introduction

This lab will take you through the steps needed to provision Thingsboard image

Estimated Time: 30 minutes

### About Thingsboard

ThingsBoard is an open-source IoT platform for data collection, processing, visualization, and device management

### Objectives

Provisioning of Thingsboard instance

### Prerequisites

This lab assumes you have:

* An Oracle account
* Administrator permissions or permissions to use the OCI Streaming, OCI Compute and Identity Domains

## Task 1: Launch Thingsboard using Marketplace

Go to Marketplace -> All Applications and search for Thingsboard. Choose your compartment and click on Launch Instance.

![Marketplace Thingsboard](images/marketplace.png)

This task will help you to create Thingsboard VM under your choosen compartment.

1. Provide the information for **Compartment**, **Name** , **Availability Domain**, **Image (Keep it as it is)**, **Shape (Keep as it is or select according to your choice)**, **VCN**, **Subnet (It has to be Public)**, **Add SSH Keys (Add keys to access your instance)**

    Click **Create**

    ![VM Thingsboard](images/thingsboard_vm_create.png)

## Task 2: Access Thingsboard UI

1. In few minutes the status of recently created  instance will change from **Provisioning** to **Running**

    ![Running Thingsboard Instance](images/thingsboard_vm_running.png)

Wait for a minute or two before accessing the thingsboard UI using the browser. You can access it by copying the IP address of the instance and adding port 8080 to it. So complete address would be IP_Address:8080. Please find it below in the image

![UI Thingsboard](images/thingsboard_ui.png)

> **Note**
    > Port 8080 should be open so you can access UI using the browser.

2. Login to the Thingsboard UI using the following credentials

* **Username** : tenant@thingsboard.org
* **Password** : tenant

You may now proceed to the next lab.

## Acknowledgements

**Authors**

* **Adina Ion-Nicolescu**, Senior Cloud Engineer, NACIE
* **Abhinav Jain**, Senior Cloud Engineer, NACIE