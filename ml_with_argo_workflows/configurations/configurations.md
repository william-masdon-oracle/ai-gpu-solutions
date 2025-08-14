# Configure the pipeline

## Introduction

This lab will guide you through the steps needed to set up the pipeline.

Estimated Time: 1 hour 30 minutes

### Objectives

This lab contains an example on how to set up and configure the automated Argo Workflow pipeline.

### Prerequisites

This lab assumes you have:

* An Oracle Cloud account
* Administrator privileges or access rights to the OCI tenancy
* Ability to create resources with Public IP addresses (Load Balancer, Instances, OKE API Endpoint)
* Ability to create OCI Functions
* Ability to create Dynamic Groups
* An OCI Auth token generated

## Task 1: Create the OCIR images

1. Once the job has succeeded go to _Stack details_ -> _Output_ and copy the ssh\_to\_operator value.

    ![Obtain ssh command](images/ssh.png)

2. With the ssh command copied, connect to the operator instance from the machine that you provided the ssh key during the provisioning of the infrastructure.

3. Once logged into the operator, go into the **ml\_training\_medical\_images** directory.

    ```
    <copy>
    cd  ml_training_medical_images
    </copy>
    ```

4. Now we need to login to OCIR. For this you need to have an OCI Auth Token generated as mentioned in the prerequisites. Copy, edit and execute the following commands in order to login into OCIR, create the OCIR image and push it.

    ```
    <copy>
    podman login -u '<NAMESPACE>/oracleidentitycloudservice/<USERNAME>' <OCI REGION PREFIX>.ocir.io
    </copy>
    ```

    ```
    <copy>
    podman build -t <OCI REGION PREFIX>.ocir.io/<NAMESPACE>/ml-train:latest -f Dockerfile.train
    </copy>
    ```

    ```
    <copy>
    podman push <OCI REGION PREFIX>.ocir.io/<NAMESPACE>/ml-train:latest
    </copy>
    ```

    The **NAMESPACE** is usually the name of the Tenancy and the **OCI REGION PREFIX** is the prefix in which you are running the livelab(for example phx for Phoenix). 


    Once the image has been pushed you need to go to OCIR in the web console and set the repository from private to public. The image is created in the ROOT compartment.

    ![OCIR](images/ocir.png)

    **NOTE**: _If the login command does not work use `podman login <OCI REGION PREFIX>.ocir.io` and provide the username as the format above._


## Task 2: Create an OCI Function

1. Download the function body from [here](https://objectstorage.eu-frankfurt-1.oraclecloud.com/p/gYkjyFZZxImrAqhCl2UkQ2_XGh9RXk23KMge-yxyZOeLtZUGK5Hvmj96XpOeKvam/n/ocisateam/b/LiveLabs/o/argo-wf-webhook.zip).

2. In the web console go to _Developer Services_ -> _Applications_ and create a new application. Set an appropriate name for it and then select a VCN which has a public subnet with **internet access**.

    ![Create application](images/application.png)

3. After the application is created follow the steps **1 through 7** from _Getting started_ -> _Cloud Shell_ setup.

4. Once you reach step 8 use the following command instead:

    ```
    <copy>
    fn init --runtime python ml-webhook
    </copy>
    ```

    And **DO NOT** execute steps **9 through 11**.

5. Upload to the Cloud Shell the function that was downloaded at step 2. You can find the upload option on the right corner of the cloud shell window. 

    ![Upload function](images/cloud_shell.png)

6. After the upload to Cloud Shell finishes, unzip it using this command:

    ```
    <copy>
    unzip -j argo-wf-webhook.zip -d ml-webhook
    </copy>
    ```

    If you named your function differently than put the correct directory name. Approve the replacement of the files.

7. Now we are going to deploy the function in our application:

    ```
    <copy>
    cd ml-webhook
    </copy>
    ```

    ```
    <copy>
    fn -v deploy --app argo-livelab
    </copy>
    ```

    (use the correct application name here if you have not used the same name)

8. While the function is being deployed go back to the operator and execute the following command to obtain the ARGO_TOKEN used for login:

    ```
    <copy>
    ARGO_TOKEN="$(kubectl get secret oracle.service-account-token -o=jsonpath='{.data.token}' | base64 --decode)"
    </copy>
    ```
    ```
    <copy>
    echo $ARGO_TOKEN
    </copy>
    ```

Then copy the value returned by the echo command.

9. Once the deploy has finished, in the web console go to the _Configuration_ section of the Application and create the following two **key:value** pairs:

    **ARGO\_TOKEN**: the value you copied at step 8  
    **ARGO\_SERVER\_URL**: go to _Stack details_ -> _Output_ and you can find it there

    ![Function Config](images/func_conf.png)

10. In order for the function to work we also need to create the following Dynamic Group and Policies:

    Dynamic Group rule: `resource.compartment.id = '< ocid where you created the function >'`    
    Policy rule: `Allow dynamic-group <name of the DG> to use functions-family in compartment <name of the compartment from the DG>`

## Task 3: Create an Event and link it to the Function

1. In the webconsole search for _Events Service_ and select _Rules_ in the Services section.

    ![Events](images/events.png)

2. Create a Rule that triggers the execution of the function we just created every time an image is uploaded in the Object Storage. Bellow you can find an example on how to setup the rule. During the provision step we also created 3 Object Storage buckets and we need to trigger the function only when uploading to raw bucket.

    ![Rule](images/rules.png)

You may proceed to the next lab.

## Acknowledgements

**Authors**

* **Dragos Nicu**, Senior Cloud Engineer, NACIE
* **Last Updated By/Date** - Dragos Nicu, January 2025
