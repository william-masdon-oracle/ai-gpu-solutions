# DCGM installation on Oracle Linux and Ubuntu VMs

## Introduction

This lab will walk you through the steps for installing and configuring NVIDIA Data Center GPU Manager (DCGM) on the deployed A10 Oracle Linux and Ubuntu virtual machines. Using these instructions, you will set up the environment required to monitor GPU metrics and enable efficient GPU management for your workloads.

Estimated Time: 40 minutes

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

1. To install the required packages and set up Docker, follow these steps.

    * First, install necessary utilities and configure the Docker repository. Then, install Docker and enable it:

        ```
        <copy>
        sudo su
        dnf install -y dnf-utils zip unzip gcc
        dnf config-manager --add-repo=https://download.docker.com/linux/centos/docker-ce.repo
        dnf remove -y runc
        dnf install -y docker-ce --nobest
        systemctl enable docker.service
        systemctl start docker.service
        usermod -a -G docker opc
        </copy>
        ```

2. To install DCGM and configure it for your environment, follow these steps.

    *  First, install the NVIDIA Container Toolkit and set up the NVIDIA repository. Then, install DCGM and enable the service:

        ```
        <copy>
        dnf install -y nvidia-container-toolkit
        dnf config-manager --add-repo https://developer.download.nvidia.com/compute/cuda/repos/rhel8/x86_64/cuda-rhel8.repo
        dnf clean expire-cache
        dnf install -y datacenter-gpu-manager
        systemctl --now enable nvidia-dcgm
        </copy>
        ```

3. Configure the Firewall.

    * To configure the firewall, stop the firewall service temporarily, open the required port for DCGM metrics, and restart the firewall to apply the changes:

        ```
        <copy>
        systemctl stop firewalld
        firewall-offline-cmd --zone=public --add-port=9400/tcp
        systemctl start firewalld
        exit
        </copy>
        ```

4. Set Up Docker Compose for DCGM Exporter, while connected as `opc`:

    * Create a directory for the Docker Compose file:

        ```
        <copy>
        mkdir /home/opc/dcgm-exporter-composer
        cd /home/opc/dcgm-exporter-composer
        </copy>
        ```

    * Create the Docker Compose file:

        ```
        <copy>
        cat <<EOF > dcgm-exporter-composer.yaml
        version: '3.8'
        services:
        dcgm-exporter:
            image: nvcr.io/nvidia/k8s/dcgm-exporter:3.3.7-3.5.0-ubuntu22.04
            deploy:
            restart_policy:
                condition: always
            environment:
            - DCGM_EXPORTER_VERSION=3.3.7-3.5.0
            network_mode: host
            cap_add:
            - SYS_ADMIN
            command: ["-r", "localhost:5555", "-f", "/etc/dcgm-exporter/dcp-metrics-included.csv"]
            container_name: dcgm-exporter
        EOF
        </copy>
        ```

5. Create a Systemd Service for Docker Compose:

    * Create the service daemon unit file while connected as `root`:

        ```
        <copy>
        sudo su
        cat <<EOF > /etc/systemd/system/docker-compose-dcgm.service
        [Service]
        Restart=always
        WorkingDirectory=/home/opc/dcgm-exporter-composer
        ExecStart=/usr/bin/docker compose -f dcgm-exporter-composer.yaml up
        ExecStop=/usr/bin/docker compose -f dcgm-exporter-composer.yaml down
        TimeoutStartSec=30

        [Install]
        WantedBy=multi-user.target
        EOF
        </copy>
        ```

    * Enable and start the service:

        ```
        <copy>
        systemctl daemon-reload
        systemctl enable docker-compose-dcgm.service
        systemctl start docker-compose-dcgm.service
        systemctl status docker-compose-dcgm.service
        </copy>
        ```

6. Verify the Setup:

    * Check the running Docker containers, still connected as `root`(it may take a few seconds until the container is started):

        ```
        <copy>
        docker ps
        </copy>
        ```

        You should see a running container similar to the following:

        ```
        <copy>
        CONTAINER ID   IMAGE                                                      COMMAND                  CREATED          STATUS          PORTS                                       NAMES
        5eb6522f4f06   nvcr.io/nvidia/k8s/dcgm-exporter:3.3.7-3.5.0-ubuntu22.04   "/usr/local/dcgm/dcg…"   41 seconds ago   Up 34 seconds   9400/tcp                                    dcgm-exporter
        </copy>
        ```

    * Check the container logs.

        ```
        <copy>
        docker logs dcgm-exporter
        </copy>
        ```


    * Verify the DCGM metrics present on the VM:

        ```
        <copy>
        curl <host IP>:9400/metrics
        OR
        curl localhost:9400/metrics
        </copy>
        ```

## Task 2: Install DCGM on the Ubuntu A10 GPU VM

1. Download the meta-package to set up the CUDA network repository.

    * Use the following commands:

        ```
        <copy>
        distribution=$(. /etc/os-release;echo $ID$VERSION_ID | sed -e 's/\.//g')
        wget https://developer.download.nvidia.com/compute/cuda/repos/$distribution/x86_64/cuda-keyring_1.1-1_all.deb
        sudo dpkg -i cuda-keyring_1.1-1_all.deb
        sudo apt-get update
        </copy>
        ```

2. To install DCGM, start by using the package manager to set up the Datacenter GPU Manager.
    
    * Run the following commands to install both DCGM and the NVIDIA driver:

        ```
        <copy>
        sudo apt-get install -y datacenter-gpu-manager
        sudo apt-get install -y nvidia-driver-555
        </copy>
        ```
    
3. Enable the DCGM service

    * To enable the DCGM service, use the systemctl command to start and enable it immediately. Verify the service status to ensure it is running

        ```
        <copy>
        sudo systemctl --now enable nvidia-dcgm
        sudo systemctl status nvidia-dcgm
        </copy>
        ```
    
    * You should see an output similar to:

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

4. To install the NVIDIA Container Toolkit, use the package manager to set up the required toolkit for containerized GPU support.

    * Run the following command:

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

    * Check Docker service status:

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

    * Check the running Docker containers (it may take a few seconds until the container is started):

        ```
        <copy>
        sudo docker ps
        </copy>
        ```

    * You should see output similar to:

        ```
        <copy>
        CONTAINER ID   IMAGE                                                      COMMAND                  CREATED          STATUS          PORTS                                       NAMES
        5eb6522f4f06   nvcr.io/nvidia/k8s/dcgm-exporter:3.3.7-3.5.0-ubuntu22.04   "/usr/local/dcgm/dcg…"   41 seconds ago   Up 34 seconds   0.0.0.0:9400->9400/tcp, :::9400->9400/tcp   hungry_lovelace
        </copy>
        ```

    * Check DCGM metrics(wait a few seconds if the page does not return results initially):

        ```
        <copy>
        curl <host IP>:9400/metrics
        OR
        curl localhost:9400/metrics
        </copy>
        ```

## Task 3: Configure Prometheus to get GPU metrics from the A10 targets

1. On the Grafana server **demo_grafana** edit the `prometheus.yml` file:

    * You can safely delete the previous content of the file and edit it as root. Make sure to replace the `<server_ip>` with the **private IPs** of the A10 targets

        ```
        <copy>
        sudo su -
        > /etc/prometheus/prometheus.yml
        vi /etc/prometheus/prometheus.yml
        </copy>
        ```

    * The file /etc/prometheus/prometheus.yml should look like below.

        ```
        <copy>
        global:
            scrape_interval: 15s # Set the scrape interval to every 15 seconds. Default is every 1 minute.
            evaluation_interval: 15s # Evaluate rules every 15 seconds. The default is every 1 minute.
            # scrape_timeout is set to the global default (10s).

        # Alertmanager configuration
        alerting:
            alertmanagers:
                - static_configs:
                    - targets:
                    # - alertmanager:9093

        # Load rules once and periodically evaluate them according to the global 'evaluation_interval'.
        rule_files:
            # - "first_rules.yml"
            # - "second_rules.yml"

        # A scrape configuration containing exactly one endpoint to scrape:
        # Here it's Prometheus itself.
        scrape_configs:
            # The job name is added as a label `job=<job_name>` to any timeseries scraped from this config.
            - job_name: "prometheus"
              static_configs:
                - targets: ["localhost:9090"]
            - job_name: 'dgcm-exporter'
              static_configs:
                - targets: ["<server_ip>:9400","<server_ip>:9400"]
        </copy>
        ```

    * restart Prometheus to pick up the new targets

        ```
        <copy>
        sudo systemctl daemon-reload
        sudo systemctl stop prometheus
        sudo systemctl start prometheus
        sudo systemctl status prometheus
        </copy>
        ```

2. Check that the target metrics are accessible on the Grafana VM.

    * On the **demo_grafana* VM run the below command, using the private IPs of the GPU VMs:

        ```
        <copy>
        curl <GPU_private_ip>:9400/metrics
        </copy>
        ```

You may now **proceed to the next lab**.

## Acknowledgements

**Authors** 
* Adina Nicolescu, Senior Cloud Engineer, NACIE
* Francisc Vass, Principal Cloud Architect, NACIE

**Last Updated By/Date**
* Adina Nicolescu - Senior Cloud Engineer, NACIE - Dec 2024
