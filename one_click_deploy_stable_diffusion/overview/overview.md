# Codebase Overview:

This section provides a structured breakdown of essential files and directories, helping you understand their contents and purpose. Read before proceeding to the next section.

### **infra/terraform/main.tf**:
Primary Terraform configuration file that defines the infrastructure resources to be deployed. Specifically, it includes OCI provider info, resource definitions, existing data sources, variables (for ssh public key and compartment id) and output (for displaying instance's public IP after provisioning).

### **infra/terraform/terraform.tfvars.example**:
A template file that defines Terraform variables, specifically compartment_id and ssh_public_key. Users should copy and rename it to terraform.tfvars, then update values accordingly.

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

### Acknowledgements
* **Author** - Jason Yan, Enterprise Cloud Architect; Blake Ramos, Enterprise Cloud Architect
* **Last Updated By/Date** - Jason Yan, Enterprise Cloud Architect, February 2025
