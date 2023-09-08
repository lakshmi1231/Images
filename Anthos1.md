# Introduction

Anthos Bare Metal (ABM), a vital component of the Anthos portfolio, revolutionizes hybrid cloud and workload management. Unlike traditional cloud platforms, Anthos Bare Metal enables seamless workload management across diverse environments, including Google Kubernetes Engine (GKE), third-party clouds such as AWS and Azure, and private clusters.

![Anthos architecture](./Images/Anthos%20architecture%201.png)

The deployment options of Anthos can be primarily categorized into the following three categories:

- ## Hybrid cluster deployment

Enterprises have built their infrastructure on-premises by employing virtualization technologies like VMware or having a host of physical servers due to various compliance and regulatory requirements. Currently, such enterprises are looking towards being more agile and delivering modern applications quickly. Based on what technologies the customer uses and where they are in their cloud journey, the following 2 Anthos offerings can be adopted:

- Anthos on VMware
- Anthos on Bare Metal

![Hybrid cluster](https://github.com/lakshmi1231/Images/raw/main/Images/Hybrid%20cluster%201.png)

- ***Edge Deployment***

This deployment is for customers who are looking to build and deploy cloud-native applications that require data and compute to be located closer to end-users to provide real-time, ultra-low latency, and immersive experiences or on resource-constrained hardware. They can deploy Anthos applications directly on their hardware infrastructure.

![Edge deployment](https://github.com/lakshmi1231/Images/raw/main/Images/edge%201.png)

- ***Admin-User (Multi) Cluster Deployment***

This is a full-blown model where enterprises would like to take so that they can be cloud-agnostic due to various drivers that include cost models, customer affinity towards a particular cloud vendor, availability of cloud regions, disaster recovery strategy, or compliance and regulations. Even though multicloud support may not be a current priority for your enterprises, enterprises should adopt cloud-native applications and principles, so it can help transition to a multicloud environment seamlessly based on future requirements. In this way, enterprises can be multicloud ready.

![Admin-user](https://github.com/lakshmi1231/Images/raw/main/Images/admin%20user%202.png)

Anthos clusters on the bare metal support multiple deployment strategies to accommodate varying requirements for resource footprint, availability, and isolation:

- **User Clusters**: Containerized workloads are executed in a Kubernetes cluster called a user cluster. It consists of worker nodes and control plane nodes. Bare metal Anthos clusters can support one or more user clusters. Worker nodes that run user workloads must be present in either one or more user clusters.

- **Admin Clusters**: A Kubernetes cluster that oversees one or more user clusters is known as an admin cluster. The admin cluster can carry out the following tasks: Construct user clusters, Boost user clusters, Refresh user clusters, and get rid of user clusters.


# 2. Architecture

In this section, we provide an overview of the architecture for deploying Anthos Bare Metal for CQASP deployment. The architecture diagram below illustrates the key components and their interactions in an Anthos Bare Metal DEPLOYMENT.

![Architecture](https://github.com/lakshmi1231/Images/raw/main/Images/Architecture%202.png)

## 2.1. Key Components and Interactions

- **Control Plane Node**: The control plane node houses the Anthos management components and the bundled load balancer. The control plane is responsible for managing and orchestrating cluster operations.

- **Worker Nodes**: Two worker nodes are provisioned for running applications and workloads. These worker nodes ensure high availability for your workloads.

- **Workstation Node**: The workstation node is equipped with essential software tools like `kubectl`, `docker`, and `bmctl` (Anthos Bare Metal Controller). It plays a pivotal role in managing the cluster, initiating deployments, and monitoring cluster health.

An HA deployment topology for Anthos cluster should have three control planes **_ceqcluster1-cp1-001, ceqcluster1-cp2-001, ceqcluster1-cp3-001_** running in the cluster. The control plane, apart from running Anthos management component, runs the Load balancer component. An external Load balancer can also be used and configure it to run with the Anthos clusters. The configuration of choosing an internal (i.e., bundled) v/s external Load balancer is specified via the configuration file during deployment.

The bundled load balancer has a prerequisite of load balancer node(s) to be in the same L2 subnet for its internal working (due to dependency on Address Resolution Protocol for announcing node addresses across the network). It is required to configure Virtual IPs for Control Plane to send traffic to GKE Kubernetes API server and for Ingress for internal service invocation.

The topology has two worker nodes for running your application/workloads. The two worker nodes **_(ceqcluster1-w1-001, ceqcluster1-w2-001)_** are configured for high availability. The worker nodes don’t have any restriction on L2 subnet and can reside in the regular L3 subnet.

## 2.2. Deployment Workflow

- **Admin Workstation VM**: You initiate the deployment process from the Admin Workstation VM. Here, you run Terraform configurations and deployment scripts.

- **Terraform Deployment (Not shown in the diagram)**: Terraform orchestrates the creation of infrastructure components on Google Cloud, including Control Plane Nodes, Worker Nodes, networking, and storage.

- **Control Plane Initialization**: Control Plane Nodes are responsible for initializing the Anthos Bare Metal control plane services, ensuring that Kubernetes and related services are up and running.

- **Worker Node Deployment**: Worker Nodes are provisioned based on the specifications you define. They receive workloads from the Control Plane Nodes.

- **Service Account Setup**: Service Accounts and authentication keys are configured to enable secure communication between cluster components and Google Cloud services.

- **Application Deployment**: With the Anthos Bare Metal cluster fully operational, you can deploy your CQASP.

The below-mentioned deployment steps cover installation of Anthos Bare Metal on GCP and subsequently deploying the CQASP software. The below install guide will focus on using the terraform templates adopted from Google repository (https://github.com/GoogleCloudPlatform/anthos-samples/tree/main/anthos-bm-gcp-terraform) and tweaked for Cequence requirements. The documentation will deploy 3 control-plane virtual machines and 2 worker-node virtual machines (default variables). The deployment of bare metal will follow the hybrid deployment model.

## 2.3 Terraform Variables

This section provides an essential overview of the Terraform variables used in your Anthos Bare Metal deployment. It outlines the requirements, provider versions, modules, resources, inputs, and outputs required for configuring and deploying Anthos Bare Metal on your infrastructure.

**Deployment Requirements:**

| Deployment Requirements | Version         |
|------------------------|-----------------|
| terraform               | >= v0.15.5, < 1.2 |
| google                 | >= 3.68.0       |
| google-beta            | >= 3.68.0       |

**Providers:**

| Name     | Version |
|----------|---------|
| google   | 4.36.0  |
| local    | 2.2.3   |


**Modules:**

| Name                           | Source                                         | Version         |
|--------------------------------|------------------------------------------------|-----------------|
| admin_vm_hosts                 | ./modules/vm                                   | No specified    |
| configure_controlplane_lb      | ./modules/loadbalancer                         | No specified    |
| configure_ingress_lb           | ./modules/loadbalancer                         | No specified    |
| controlplane_vm_hosts          | ./modules/vm                                   | No specified    |
| create_service_accounts        | terraform-google-modules/service-accounts/google | Version ~> 4.0 |
| enable_google_apis_primary     | terraform-google-modules/project-factory/google//modules/project_services | Version 13.0.0 |
| enable_google_apis_secondary   | terraform-google-modules/project-factory/google//modules/project_services | Version 13.0.0 |
| gke_hub_membership             | terraform-google-modules/gcloud/google          | Version ~> 3.1.1 |
| init_hosts                     | ./modules/init                                  | No specified    |
| install_abm                    | ./modules/install                               | No specified    |
| instance_template              | terraform-google-modules/vm/google//modules/instance_template | Version ~> 7.8.0 |
| worker_vm_hosts                | ./modules/vm                                   | No specified    |


**Resources:**

| Name                                      | Type                                    |
|-------------------------------------------|-----------------------------------------|
| google_compute_firewall.lb-firewall-rule  | resource                                |
| google_filestore_instance.cluster-abm-nfs | resource                                |
| local_file.cluster_yaml_bundledlb          | resource                                |
| local_file.cluster_yaml_manuallb          | resource                                |
| local_file.init_args_file                 | resource                                |
| local_file.nfs_yaml                       | resource                                |


**Inputs:**

| Parameter                      | Description                                                                     |
|--------------------------------|---------------------------------------------------------------------------------|
| abm_cluster_id                 | Unique id to represent the Anthos Cluster to be created.                        |
| access_scopes                  | The IAM access scopes associated with the Compute Engine VM Service Accounts.    |
| anthos_service_account_name    | Name given to the Service account that will be used by the Anthos cluster components. |
| boot_disk_size                 | Size of the primary boot disk to be attached to the Compute Engine VMs in GBs.  |
| boot_disk_type                 | Type of the boot disk to be attached to the Compute Engine VMs.                  |
| connect_agent_account          | GCP account email address to use with Connect Agent for logging into the cluster using Google Cloud identity. |
| credentials_file               | Path to the Google Cloud Service Account key file (required).                   |
| enable_nested_virtualization   | Enable nested virtualization on the Compute Engine VMs to be scheduled.          |
| gce_vm_service_account         | Service Account to use for GCE instances.                                       |
| gpu                            | GPU information to be attached to the provisioned GCE instances.                |
| image                          | The source image to use when provisioning the Compute Engine VMs.                |
| image_family                   | Source image to use when provisioning the Compute Engine VMs.                   |
| image_project                  | Project name of the source image to use when provisioning the Compute Engine VMs. |
| instance_count                 | Number of instances to provision per layer (Control plane and Worker nodes) of the cluster. |
| machine_type                   | Google Cloud machine type to use when provisioning the Compute Engine VMs.      |
| min_cpu_platform               | Minimum CPU architecture upon which the Compute Engine VMs are to be scheduled. |
| mode                           | Indication of the execution mode (setup, install, manuallb).                     |
| network                        | VPC network to which the provisioned Compute Engine VMs are to be connected.    |
| nfs_server                     | Provision a Google Filestore instance for NFS shared storage.                    |
| primary_apis                   | List of primary Google Cloud APIs to be enabled for this deployment.             |
| project_id                     | Unique identifier of the Google Cloud Project that is to be used (required).    |
| region                         | Google Cloud Region in which the Compute Engine VMs should be provisioned.      |
| resources_path                 | Path to the resources folder with the template files (required).                |
| secondary_apis                 | List of secondary Google Cloud APIs to be enabled for this deployment.           |
| tags                           | List of tags to be associated with the provisioned Compute Engine VMs.            |
| username                       | The name of the user to be created on each Compute Engine VM to execute the init script. |
| zone                           | Zone within the selected Google Cloud Region that is to be used.                 |


**Outputs:**

| Name               | Description                                                                                       |
|--------------------|---------------------------------------------------------------------------------------------------|
| admin_vm_ssh       | Run the following command to provision the Anthos cluster.                                       |
| controlplane_ip    | You may access the control plane nodes of the Anthos on bare metal cluster by accessing this IP address. You need to copy the kubeconfig file for the cluster from the admin workstation to access using the kubectl CLI. |
| ingress_ip         | You may access the application deployed in the Anthos on bare metal cluster by accessing this IP address. |
| installation_check | Run the following command to check the Anthos Bare Metal installation status.                   |


## 2.4 Installation Guide (Hybrid Mode)

This section provides a step-by-step guide to installing Anthos Bare Metal using the All-in-One Install method. Follow these instructions to set up your Anthos Bare Metal cluster:

#### Step 1: Prerequisites

Before you begin the installation process, ensure you have the following prerequisites in place:
- An admin workstation VM with the necessary permissions.
- Terraform version >= v0.15.5 and < 1.2.
- Google Cloud CLI (gcloud) installed and configured.
- Google Cloud Service Account key file (JSON) with the appropriate permissions.
- A Google Cloud Project where you want to deploy Anthos Bare Metal.
- Properly configured Terraform variables (as explained in section 3.1).

#### Step 2: Clone the Anthos on Bare Metal Repository

[Clone the Anthos on Bare Metal GitHub repository to your admin workstation VM](https://github.com/your-organization/anthos-on-bare-metal.git)

#### Step 3: Change to the Anthos Repository Directory

Navigate to the cloned repository directory:

```bash
terraform init
terraform plan
terraform apply
tail -f ./terraform.log
```

#### Step 4: Initialize terraform
```bash
terraform init
```

#### Step 4: Create a terraform plan
```bash
terraform plan
```

#### Step 5: Apply the terraform configuration
```bash
terraform apply
```

#### Step 6: Monitor the Installation
```bash
tail -f ./terraform.log
```

The apply command (in step 6) sets up the Compute Engine VM based bare metal infrastructure. This can take a few minutes (approx. 3-5 mins) for the entire bare-metal cluster to be setup. Deploy an Anthos cluster.


# 3. Anthos Bare Metal Deployment

In this section, we cover the installation of Anthos Bare Metal on GCP (Google Cloud Platform) and subsequently deploying the CQASP (Cequence Application Security Platform) software. The installation guide focuses on using Terraform templates adapted from the [Google repository](https://github.com/GoogleCloudPlatform/anthos-samples/tree/main/anthos-bm-gcp-terraform) and customized for Cequence requirements. The deployment includes 3 control-plane virtual machines and 2 worker-node virtual machines (default variables), following the hybrid deployment model.


## 3.1. Deploy an anthos cluster

After the Terraform, execution completes you are ready to deploy an Anthos cluster.

#### a. SSH into admin host

```bash
gcloud compute ssh tfadmin@cluster1-ceq-ws0-001 --project=<YOUR_PROJECT> --zone=<YOUR_ZONE>
```

#### b.	Install the Anthos cluster on the provisioned Compute Engine VM based bare metal infrastructure.

```bash
sudo ./run_initialization_checks.sh && \
sudo bmctl create config -c cluster1 && \
sudo cp ~/cluster1.yaml bmctl-workspace/cluster1 && \
sudo bmctl create cluster -c cluster1
```
Running the commands from the Terraform output starts setting up a new Anthos cluster. This includes checking the initialization state of the nodes, creating the admin and user clusters and also registering the cluster with Google Cloud using Connect. The whole setup can take up to 15 minutes. You see the following output as the cluster is being created:

 ![Architecture](https://github.com/lakshmi1231/Images/raw/main/Images/3.3%20b.png)

#### c.	Verify and interacting with the Baremetal cluster.
You can find your cluster's kubeconfig file on the admin machine in the bmctl-workspace directory. To verify your deployment, complete the following steps. SSH into the admin host (if you are not already inside it):
```bash
gcloud compute ssh tfadmin@cluster1-abm-ws0-001 --project=<YOUR_PROJECT> --zone=<YOUR_ZONE>
```
Set the KUBECONFIG environment variable with the path to the cluster's configuration file to run kubectl commands on the cluster


```bash
export CLUSTER_ID=cluster1
export KUBECONFIG=$HOME/bmctl-workspace/$CLUSTER_ID/$CLUSTER_ID-kubeconfig
kubectl get nodes
```

You should see the nodes of the cluster printed, like the output below:

![Architecture](https://github.com/lakshmi1231/Images/raw/main/Images/3.3%20c.png)

## 3.2. Logging into anthos cluster

Upon successfully deploying Anthos Bare Metal, it's essential to understand how to access and manage cluster resources. This section explains how to log in to the Anthos cluster and can view and interact with cluster resources (like Pods, Services etc) via the Google Cloud Platform (GCP) UI.
To log in, access the GKE Console and choose your preferred method of authentication. In this example, we will create a Kubernetes Service Account and use the generated token for login.

#### Step 1: To enter the auth we can go to GKE console and click on 3 dots then login

There are multiple authentication options available, but for this demonstration, we will create a Kubernetes Service Account (KSA) for logging in. Please note that we are using the cluster-admin role for this test

![Architecture](https://github.com/lakshmi1231/Images/raw/main/Images/3.4%20a.png)

```bash
apiVersion: v1
kind: ServiceAccount
metadata:
  name: anthos
  namespace: kube-system
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: anthos-admin
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-admin
subjects:
  - kind: ServiceAccount
    name: anthos
    namespace: kube-system
```

Apply the sa.yaml file to retrive the token, which can be done by: 
```bash
kubectl apply -f sa.yaml
kubectl get secret -n kube-system | grep anthos
kubectl describe -n kube-system secret anthos-token-<generated id>
```
#### Step 2: Input the token on the console

![Architecture](https://github.com/lakshmi1231/Images/raw/main/Images/3.4%20b.png)

Now the information is seen on GCP console (GKE UI)

![Architecture](https://github.com/lakshmi1231/Images/raw/main/Images/3.4%20bb.png)

Also it's provided on Anthos UI

![Architecture](https://github.com/lakshmi1231/Images/raw/main/Images/3.4%20bbb.png)

The cluster is now ready. With the kubeconfig one can proceed to deploy workloads and use it as kubernetes cluster.

## 3.3	Creating google cloud service account keys.

To set up Anthos Bare Metal effectively, you'll need a Google Cloud Service Account key with the appropriate permissions. This section guides you through creating a service account and generating the necessary key:

#### Step 1: Access the Google Cloud Console

Open your web browser and navigate to the Google Cloud Console.

#### Step 2: Choose your project

In the upper-right corner of the console, ensure that you have selected the Google Cloud project where you intend to deploy Anthos Bare Metal. If not, click on the project dropdown and choose your project

#### Step 3: Open the Service Accounts Page

In the left-hand navigation pane, go to "IAM & Admin" and select "Service accounts."

#### Step 4: Create a new service account

Click the "+ CREATE SERVICE ACCOUNT" button at the top of the page.

#### Step 5: Configure the Service Account

Provide a name for your service account, e.g., "AnthosServiceAccount."
Optionally, provide a description.
Choose the role for your service account. To deploy Anthos Bare Metal, the service account should have sufficient permissions, such as "Editor" or custom roles with necessary privileges.
Click the "Continue" button.


#### Step 7: Grant User Access (Optional)

You can assign users or groups access to this service account if needed. This step is optional.

#### Step 8: Add Service Account Key

Click on the "Keys" tab.
Click the "+ CREATE KEY" button.

#### Step 9: Choose Key Type

Select the key type: JSON or P12. For Anthos Bare Metal, choose JSON as it's more commonly used.

#### Step 10: Create the Key

Click the "CREATE" button. A JSON key file will be generated and downloaded to your computer.

#### Step 11: Save the Key Securely

Ensure that you save the JSON key file in a secure location, as it contains sensitive information required for Anthos Bare Metal deployment.

#### Step 12: Configure Anthos with the Key

During the Anthos Bare Metal deployment (as explained in section 3.3), you'll need to provide the path to this JSON key file. Make sure you have the key file accessible when running the Terraform configuration.

This completes the process of creating a Google Cloud Service Account and generating a key for Anthos Bare Metal deployment.

## 3.4 Istio service mesh

As a result of increased adoption of cloud platforms, developers have to architect for portability using services, while operators have to manage large, distributed deployments that span hybrid and multi-cloud deployments.
Istio reduces complexity of managing services deployments by providing a uniform way to secure, connect, and monitor microservices across environments.
Istio on Anthos Bare Metal provides all the benefits of open-source Istio, without the complexity of configuration, installation, upgrade, and certificate authority setup.

### 3.4.1	Why use Istio

-	Automatic load balancing for HTTP, gRPC, WebSocket, and TCP traffic.
-	Fine-grained control of traffic behaviour with routing rules, retries, failovers, and fault injection.
-	A pluggable policy layer and configuration API supporting access controls, rate limits and quotas.
-	Automatic metrics, logs, and traces for all traffic within a cluster, ingress and egress.
-	Secure service-to-service communication in a cluster with strong identity-based authentication and authorization.

### 3.4.2 Istio - Install instructions

#### Step 1: Connect to cluster

#### Step 2: Download the latest version of istio

```bash
curl -L https://istio.io/downloadIstio | sh -
cd istio-<<VERSION>>
```
#### Step 3: Install istio with the demo configuration profile

```bash
./bin/istioctl install --set profile=demo -y
```

#### Step 4: VERIFy the installation and ensure the istio components are running without any errors
```bash
#verify the pod status
kubectl get pods -n istio-system
```


