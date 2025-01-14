# Train and validate the model

### Objectives

In this lab we will train a ML model on medical images and validate that the model was trained succesfully.

Estimated Time: 1 hour

### Prerequisites

This lab assumes you have:

* An Oracle Cloud account
* Administrator privileges or access rights to the OCI tenancy
* Ability to create resources with Public IP addresses (Load Balancer, Instances, OKE API Endpoint)
* Ability to create OCI Functions
* Ability to create Dynamic Groups
* An OCI Auth token generated

## Task 1: Upload medical images to the Object Storage bucket

1. In the web console got to Storage -> Object Storage -> Buckets and select the medical-images-raw bucket.

2. In this bucket you can upload any medical images that you want but they need to be of the same type(x-ray of brest cancer for example) and you should upload at least 10 files at first.

3. You can verify that the model was trainied once the model.keras file appears in the trained-model bucket

## Task 2: Validate the model

1. SSH back into the operator and into the **ml_training_medical_images** directory.

2. Execute the validate_model.py file using the following command:

    `python3.10 validate_model.py`

3. The script validates against the images that were used for training and will return how accurate was the training.

This concludes our ML pipeline using OCI and Argo workflows.

## Acknowledgements

**Authors**

* **Dragos Nicu**, Senior Cloud Engineer, NACIE
* **Last Updated By/Date** - Dragos Nicu, January 2025
