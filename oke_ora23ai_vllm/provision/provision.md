# Provision of resources to run JupyterHub Notebook

## Introduction

This lab will guide you through the steps needed to provision the infrastructure using the Resource Manager.

Estimated Time: 45 minutes

### **Objectives**

Provisioning of the infrastructure using OCI Resource Manager.

### **Prerequisites**

This lab assumes you have:

* An Oracle Cloud account
* Administrator privileges or access rights to the OCI tenancy
* Ability to provision A10 instances in OCI
* Ability to provision an Oracle 23ai Autonomous Database
* Ability to create resources with Public IP addresses (Load Balancer, Instances, OKE API Endpoint)
* Access to Hugging Face and acceptance of selected model license agreements
* A Hugging Face token with at least read permissions (Go to Profile -> Settings -> Access Tokens)
* Database admin password stored in a vault secret.

## Task 1: Provision resources

1. Go to Resource manager -> Stacks -> Create Stack. Choose My configuration and upload the provided zip file and click Next: [livelab-llm-23ai.zip](https://objectstorage.eu-frankfurt-1.oraclecloud.com/p/8jwmISGYydt3QpwellsJN29NwKU8r4PHyA_UhjQQ1UBaWhZ0tMkzeDr40au1qoGP/n/ocisateam/b/LiveLabs/o/livelab-llm-23ai.zip)

    ![Resource Manager](images/resource_manager.png)

    Or you could use the single click deployment button shown below

    [![Deploy to Oracle Cloud](https://oci-resourcemanager-plugin.plugins.oci.oraclecloud.com/latest/deploy-to-oracle-cloud.svg)](https://cloud.oracle.com/resourcemanager/stacks/create?zipUrl=https://objectstorage.eu-frankfurt-1.oraclecloud.com/p/8jwmISGYydt3QpwellsJN29NwKU8r4PHyA_UhjQQ1UBaWhZ0tMkzeDr40au1qoGP/n/ocisateam/b/LiveLabs/o/livelab-llm-23ai.zip)

2. Provide the following information: 

**Select Compartment**: Choose the appropriate compartment for the Kubernetes cluster.
**Cluster Name**: Enter a descriptive name for the cluster.
**Create New VCN**: Check the option to create a new VCN, provide a name, and leave other networking details as default.
**Node Pool Configuration**: Set the GPU node pool size to two; leave other settings as default or adjust as needed.

3. Next, check all 3 options in Access to the Kubernetes cluster and provide your ssh key to connect to the bastion and operator hosts.

    You can deploy the resources in two ways. Either you create a bastion and operator hosts. These are two VMs and operator host would be configured with kubectl so you can directly execute command from operator against the kubernetes cluster or commands can be executed from oracle resource manager runner.

    If you do not select this option then it is mandatory to have a public OKE endpoint. But if you create bastion and operator hosts then creating a public oke endpoint is optional. For this workshop we will check all 3 and leave as default. Provide your public key to connect to the bastion and operator hosts.

4. In the Autonomous Database configuration provide the OCID of the secret where the ADMIN password for the DB is stored. The rest of the configurations can be changed as needed or leave them as default.

5. Next, in Helm Chart deployments section check all the 3 boxes and leave them as default. No need to provide the optional file. JupyterHub provides the access to the application using the browser. Provide a suitable username and password to access the application in the next lab.

6. Leave JupyterHub - Playbooks Git Repo as default.

7. Check Helm | Deploy vLLM box

8. The model that is being downloaded is from HuggingFace so you have have an account on HuggingFace. The default model is 'meta-llama/Llama-3.2-1B-Instruct'. You will need to go to HuggingFace website huggingface.co, create an account, accept the T&C for the model you'd like to use and generate a token to use here.

    Go to Profile -> Settings -> Access Tokens and generate a token as shown below

    ![Token](images/huggingface_token.png)

9. vLLM API Key is used to secure your LLM deployment. It could be anything you want according to your preference or you can remember. Choose any value for the key according to your preference, such as "dummy" or "live-lab."

10. The HuggingFace model deployed is 'meta-llama/Llama-3.2-1B-Instruct'.

11. Leave the maximum context length as default '-1'.

12. Click Next and then select Run Apply and finally click on Create as shown below.

    ![Apply Stack](images/run_apply.png)

13. Wait for the job to complete, which may take 15-20 minutes before the infrastructure is fully provisioned.

You may now proceed to the next lab.

## Acknowledgements

**Authors**

* **Dragos Nicu**, Senior Cloud Engineer, NACIE
* **Guido Alejandro Ferreyra**, Principal Cloud Architect,NACIE
* **Last Updated By/Date** - Dragos Nicu, October 2024
