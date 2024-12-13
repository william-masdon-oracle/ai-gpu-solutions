# DCGM installation on Oracle Linux and Ubuntu VMs

## Introduction

This lab will walk you through the steps for installing and configuring NVIDIA Data Center GPU Manager (DCGM) on Oracle Linux and Ubuntu virtual machines. Using these instructions, you will set up the environment required to monitor GPU metrics and enable efficient GPU management for your workloads.

Estimated Time: 60 minutes

### Objectives

* Installing and configuring NVIDIA DCGM to monitor GPU metrics and manage resources efficiently on the OEL and Ubuntu A10 provisioned VMs
* Configuration of Prometheus to receive the metrics.

### Prerequisites

This lab assumes you have:

* An Oracle Cloud account
* Administrator permissions or permissions to use the OCI Compute and OCI Networking services
* Access to A10 or GPU shape
* Access to the Oracle Resource Manager(ORM)

## Task 1: Install DCGM on Oracle Linux

## Task 2: Install DCGM on the Ubuntu A10 GPU VM

1. Download the meta-package to set up the CUDA network repository

    ```
    <copy>
    distribution=$(. /etc/os-release;echo $ID$VERSION_ID | sed -e 's/\.//g')
    wget https://developer.download.nvidia.com/compute/cuda/repos/$distribution/x86_64/cuda-keyring_1.1-1_all.deb
    sudo dpkg -i cuda-keyring_1.1-1_all.deb
    sudo apt-get update
    </copy>
    ```

2. Install DCGM:

    ```
    <copy>
    sudo apt-get install -y datacenter-gpu-manager
    sudo apt-get install -y nvidia-driver-555
    </copy>
    ```
    
3. Enable the DCGM service

    ```
    <copy>
    sudo systemctl --now enable nvidia-dcgm
    sudo systemctl status nvidia-dcgm
    </copy>
    ```

    You should see an output similar to:

    ```
    <copy>
    nvidia-dcgm.service - NVIDIA DCGM service
            Loaded: loaded (/lib/systemd/system/nvidia-dcgm.service; enabled; vendor preset: enabled)
            Active: active (running) since Fri 2024-12-13 09:34:33 UTC; 35s ago
        Main PID: 3983 (nv-hostengine)
            Tasks: 6 (limit: 289832)
            Memory: 1.3M
                CPU: 11ms
            CGroup: /system.slice/nvidia-dcgm.service
                    └─3983 /usr/bin/nv-hostengine -n --service-account nvidia-dcgm
    </copy>
    ```

4. Install NVIDIA Container Toolkit

    ```
    <copy>
    sudo apt-get install -y nvidia-container-toolkit
    </copy>
    ```

5. Install Docker:

* Remove old installations, if any:

    ```
    <copy>
    for pkg in docker.io docker-doc docker-compose docker-compose-v2 podman-docker containerd runc; do sudo apt-get remove $pkg; done
    </copy>
    ```

* Add Docker's official GPG key and repository:

    ```
    <copy>
    sudo apt-get update
    sudo apt-get install ca-certificates curl
    sudo install -m 0755 -d /etc/apt/keyrings
    sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
    sudo chmod a+r /etc/apt/keyrings/docker.asc
    echo \
    "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
    $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
    sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
    sudo apt-get update
    </copy>
    ```

* Install Docker packages:

    ```
    <copy>
    sudo apt-get install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
    </copy>
    ```

* Add the Ubuntu user to the Docker group:

    ```
    <copy>
    sudo usermod -aG docker $USER
    </copy>
    ```

* Check Docker service statu

    ```
    <copy>
    sudo systemctl status docker
sudo systemctl status containerd
    </copy>
    ```

6. Configure DCGM as a containerized service:

* Become root:

    ```
    <copy>
    sudo su -
    </copy>
    ```

* Create a systemd service for DCGM:

    ```
    <copy>
    cat <<EOF > /etc/systemd/system/dcgm_service.service
    [Unit]
    Description=NVIDIA DCGM Exporter
    After=docker.service
    Requires=docker.service

    [Service]
    Restart=always
    ExecStart=/usr/bin/docker run --gpus all --cap-add SYS_ADMIN --rm -p 9400:9400 nvcr.io/nvidia/k8s/dcgm-exporter:3.3.7-3.5.0-ubuntu22.04
    ExecStop=/usr/bin/docker stop %n
    ExecStopPost=/usr/bin/docker rm %n

    [Install]
    WantedBy=multi-user.target
    EOF
    </copy>
    ```

* Return to the Ubuntu user:

    ```
    <copy>
    exit
    </copy>
    ```

* Enable and start the DCGM service:

    ```
    <copy>
    sudo systemctl daemon-reload
    sudo systemctl enable dcgm_service.service
    sudo systemctl start dcgm_service.service
    </copy>
    ```

7. Verify the DCGM service

* Check the running Docker containers:

    ```
    <copy>
    sudo docker ps
    </copy>
    ```

    You should see output similar to:

    ```
    <copy>
    CONTAINER ID   IMAGE                                                      COMMAND                  CREATED          STATUS          PORTS                                       NAMES
    5eb6522f4f06   nvcr.io/nvidia/k8s/dcgm-exporter:3.3.7-3.5.0-ubuntu22.04   "/usr/local/dcgm/dcg…"   41 seconds ago   Up 34 seconds   0.0.0.0:9400->9400/tcp, :::9400->9400/tcp   hungry_lovelace
    </copy>
    ```

* Check DCGM metrics:

     ```
    <copy>
    curl localhost:9400/metrics
    </copy>
    ```

You may now proceed to the next lab.

## Acknowledgements

**Authors** 
* Adina Nicolescu, Senior Cloud Engineer, NACIE
* Francisc Vass, Principal Cloud Architect, NACIE

**Last Updated By/Date**
* Adina Nicolescu - Senior Cloud Engineer, NACIE - Dec 2024
