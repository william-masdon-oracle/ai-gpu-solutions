# Deploy VM infrastructure (GPU shape); install & configure Prometheus and Grafana

## Introduction

In this lab, you will deploy the foundational infrastructure required for GPU performance monitoring using Oracle Resource Manager (ORM). The ORM stack will provision two A10 GPU-enabled virtual machines (one with Oracle Linux and one with Ubuntu) in a public subnet for generating and monitoring GPU activity. Additionally, a separate VM will be deployed to host Prometheus and Grafana. While the ORM stack sets up the infrastructure, Prometheus and Grafana will be installed and configured manually in later steps. This lab lays the groundwork for the subsequent stages of the workshop, where GPU activity and metrics will be monitored and visualized.

Estimated Time: 30 minutes

### Objectives

* Provision infrastructure using ORM to deploy two A10 GPU-enabled virtual machines (Oracle Linux and Ubuntu) and a separate VM for Prometheus and Grafana.
* Manually install Prometheus on the designated VM.
* Manually install Grafana on the designated VM.

### Prerequisites

This lab assumes you have:

* An Oracle Cloud account
* Administrator permissions or sufficient permissions to use OCI Compute and Networking services.
* Access to A10 GPU-enabled shapes, specifically the VM.GPU.A10.1 shape
* Access to the Oracle Resource Manager(ORM).

## Task 1: Ensure Networking Prerequisites

Before deploying the infrastructure, ensure the following networking prerequisites are met to allow proper functionality of the lab:

1. **Existing VCN and Public Subnet**:

    The ORM stack requires an existing VCN with a public subnet. The public subnet simplifies access to:
    * The Grafana dashboard from your laptop.
    * The Prometheus dashboard from your laptop.s

2. **Subnet Requirements**:

    The subnet must be public and associated with:
    * A Route Table containing an Internet Gateway to enable outbound Internet access.
    
    **Note** - The ORM stack includes a validation step that will test the subnet and raise an error if it is not public.

3. **Security rules**:

    The public subnet must have the following security rules configured:
    * Allow ingress for TCP/22 from your laptop or server to SSH into the VMs and for Ansible to work.
    * Allow ingress for TCP/3000 from your laptop or server to access the Grafana dashboard.
    * Allow ingress for TCP/9090 from your laptop or server to access the Prometheus dashboard.
    * Allow ingress/egress for TCP/9400 and TCP/9100 within the subnet where the VMs are created (for DCGM and Node Exporter).

Ensure these configurations are in place before proceeding to deploy the ORM stack.

## Task 2: Provision GPUs and Standard VM with the ORM Stack

To deploy the infrastructure for this lab, you will create and configure a stack in Oracle Resource Manager (ORM) to automate the provisioning of GPU-enabled and standard virtual machines.

1. **Create the ORM stack**
    
    * Navigate to the Resource Manager section in the Oracle Cloud Console:  _Developer Services_ -> _Resource manager_ -> _Stacks_ -> _Create Stack_.
    * Choose _My configuration_, upload this [infrastructure automation stack](???) and click **Next**.

        ![Resource Manager](./../../dcgm_gpu_monitoring/infra/images/resource_manager.png " ")

    OR

    * You can use the single click deployment button below to launch the stack creation directly, accept the terms and click **Next**.

        [![Deploy to Oracle Cloud](https://oci-resourcemanager-plugin.plugins.oci.oraclecloud.com/latest/deploy-to-oracle-cloud.svg)](https://cloud.oracle.com/resourcemanager/stacks/create?zipUrl=???)

2. Choose or fill in the deployment options, including:

    * **Compartment**, **Availability domain**, **VCN**, **Public Subnet**, **Public SSH Key**

        ![Deployment options](./../../dcgm_gpu_monitoring/infra/images/orm_stack_config.png " ")

    * Click **Next**.

3. Create and run the stack:
    
    * Review the options selected in the previous steps and then select _Run Apply_ and finally click on **Create** as shown below.

        ![Apply Stack](./../../dcgm_gpu_monitoring/infra/images/apply_stack.png " ")

4. Wait for the job to complete, which should only take a few minutes. 
    
    * Once the stack creation is complete, you will have 3 new VMs:
        * **demo_linux_1**: to be monitored with DCGM
        * **demo_ubuntu_1**: to be monitored with DCGM
        * **demo_grafana**: will host the monitoring tools, Prometheus and Grafana.
    
    * The IP addresses of these instances will be displayed in both the stack output and the stack log.
    
       ![Instances](./../../dcgm_gpu_monitoring/infra/images/orm_stack_output.png " ")

    * `**Note** - The ORM stack will deploy two VM.GPU.A10.1 instances. To save costs, ensure to stop or destroy them after completing the lab.`


## Task 3: Install Prometheus

Follow these steps to install Prometheus on the **demo_grafana** VM:

1. Connect to the `demo_grafana` VM:

* In your local terminal, use the SSH private key corresponding to the public key provided during the ORM stack deployment to connect to the demo_grafana VM. Replace <public-ip> with the public IP of the VM:

    ```
    <copy>
    ssh -i <path-to-your-ssh-private-key> opc@<public-ip>
    </copy>
    ```

2. Download and extract Prometheus:

    ```
    <copy>
    wget https://github.com/prometheus/prometheus/releases/download/v2.37.6/prometheus-2.37.6.linux-amd64.tar.gz
    tar xvfz prometheus-*.tar.gz
    </copy>
    ```
3. Create required directories:

* Create two directories for Prometheus:

    * `/etc/prometheus`: To store Prometheus configuration files.
    * `/var/lib/prometheus`: To hold application data.

    ```
    <copy>
    sudo mkdir /etc/prometheus /var/lib/prometheus
    </copy>
    ```
4. Navigate to the extracted folder:

    ```
    <copy>
    cd prometheus-2.37.6.linux-amd64
    </copy>
    ```

5. Move executables:

* Move the `prometheus` and `promtool` binaries to `/usr/local/bin` to make Prometheus accessible globally, to all users:

    ```
    <copy>
    sudo mv prometheus promtool /usr/local/bin/
    </copy>
    ```

6. Move configuration file:

* Move the `prometheus.yml` configuration file to `/etc/prometheus`:

    ```
    <copy>
    sudo mv prometheus.yml /etc/prometheus/prometheus.yml
    </copy>
    ```

7. Move console resources:

* Move the `consoles` and `console_libraries` directories to `/etc/prometheus`. These contain resources for creating custom consoles (not covered in this guide):

    ```
    <copy>
    sudo mv consoles/ console_libraries/ /etc/prometheus/
    </copy>
    ```

8. Verify installation:

* Check the Prometheus version to confirm the installation:

    ```
    <copy>
    prometheus --version
    </copy>
    ```

9. Instructions to disable SELinux:

* Open the SELinux configuration file for editing:

    ```
    <copy>
    sudo vi /etc/selinux/config
    </copy>
    ```

* Locate the line with SELINUX= and change its value to:

    ```
    <copy>
    SELINUX=disabled
    </copy>
    ```

* Save the file and exit.

* Reboot the system to apply the changes:

    ```
    <copy>
    sudo reboot
    </copy>
    ```


## Task 4: Configure Prometheus as a Service

Perform the below steps connected to the `demo_grafana` VM:

1. Create a `prometheus` user:

* Add a system user for Prometheus to run as a service:

    ```
    <copy>
    sudo useradd -rs /bin/false prometheus
    </copy>
    ```

2. Assign ownership of directories:

* Grant the `prometheus` user ownership of the directories created earlier:

    ```
    <copy>
    sudo chown -R prometheus: /etc/prometheus /var/lib/prometheus
    </copy>
    ```

3. Create a systemd service file:

* Configure Prometheus to run as a systemd service by creating a service file:

    ```
    <copy>
    sudo vi /etc/systemd/system/prometheus.service
    </copy>
    ```

    Add the following content to the file:

    ```
    <copy>
    [Unit]
    Description=Prometheus
    Wants=network-online.target
    After=network-online.target

    [Service]
    User=prometheus
    Group=prometheus
    Type=simple
    Restart=on-failure
    RestartSec=5s
    ExecStart=/usr/local/bin/prometheus \
        --config.file /etc/prometheus/prometheus.yml \
        --storage.tsdb.path /var/lib/prometheus/ \
        --web.console.templates=/etc/prometheus/consoles \
        --web.console.libraries=/etc/prometheus/console_libraries \
        --web.listen-address=0.0.0.0:9090 \
        --web.enable-lifecycle \
        --log.level=info

    [Install]
    WantedBy=multi-user.target
    </copy>
    ```

4. Enable and start the service:

* Reload the systemd daemon, enable the Prometheus service, and start it:

    ```
    <copy>
    sudo systemctl daemon-reload
    sudo systemctl enable prometheus
    sudo systemctl start prometheus
    </copy>
    ```

5. Check service status:

* Verify that Prometheus is running:

    ```
    <copy>
    sudo systemctl status prometheus
    </copy>
    ```

6. Open port 9090 in the firewall:

* Allow access to the Prometheus dashboard by opening port 9090:

    ```
    <copy>
    sudo systemctl stop firewalld
    sudo firewall-offline-cmd --zone=public --add-port=9090/tcp
    sudo systemctl start firewalld
    </copy>
    ```

7. Access Prometheus dashboard:

* Open your browser and navigate to:

    ```
    <copy>
    http://<public-ip>:9090
    </copy>
    ```
    Replace `<public-ip>` with the public IP address of the Prometheus server.

## Task 5: Install Grafana

The next steps will walk you through the instructions to install Grafana, configure it, and connect it to Prometheus as a data source. These steps should be performed on the `demo_grafana` VM.

1. Create a new repository file for Grafana:

    ```
    <copy>
    sudo vi /etc/yum.repos.d/grafana.repo
    </copy>
    ```
2. Add the following configuration to the file and save it:
    
    ```
    <copy>
    [grafana]
    name=grafana
    baseurl=https://packages.grafana.com/oss/rpm
    repo_gpgcheck=1
    enabled=1
    gpgcheck=1
    gpgkey=https://packages.grafana.com/gpg.key
    sslverify=1
    sslcacert=/etc/pki/tls/certs/ca-bundle.crt
    </copy>
    ```
3. Install Grafana using the dnf package manager:
    
    ```
    <copy>
    sudo dnf install grafana -y
    </copy>
    ```
4. Enable Grafana to start at boot:
    
    ```
    <copy>
    sudo systemctl enable grafana-server
    </copy>
    ```

5. Start the Grafana service:

    ```
    <copy>
    sudo systemctl start grafana-server
    </copy>
    ```

6. Check Grafana's status to ensure it's running:

    ```
    <copy>
    sudo systemctl status grafana-server
    </copy>
    ```

7. Configure the firewall for Grafana, which operates on port `3000` by default. To allow access, stop the firewall temporarily, open port `3000` and restart the firewall:

    ```
    <copy>
    sudo systemctl stop firewalld
    sudo firewall-offline-cmd --zone=public --add-port=3000/tcp
    sudo systemctl start firewalld
    </copy>
    ```

8. To access Grafana:

* Use the public IP of the `demo_grafana` VM in a browser:

    ```
    <copy>
    http://<public-ip>:3000
    </copy>
    ```

* Log in using the default credentials:

    * **Username**: admin
    * **Password**: admin
    _(Change the password upon first login.)_

    ```
    <copy>
    </copy>
    ```

## Task 6: Add Prometheus as a data source

1. In the Grafana web interface, navigate to _Settings_ > _Data Sources_.

2. Select **Prometheus** as the data source type.

3. Enter the Prometheus server's private IP and port. Prometheus typically runs on port `9000`:

* Example: `http://<private-ip>:9000`

4. Click Save & Test to verify the connection.


You may now proceed to the next lab.

## Acknowledgements

**Authors** 
* Adina Nicolescu, Senior Cloud Engineer, NACIE
* Francisc Vass, Principal Cloud Architect, NACIE

**Last Updated By/Date**
* Adina Nicolescu - Senior Cloud Engineer, NACIE - Dec 2024
