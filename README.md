# Setting-Up-Kubernetes-Clusters-on-AWS-EC2

![image](https://github.com/user-attachments/assets/6bade0c9-907f-4cea-bbd6-d5e5a95de9e6)

# Introduction:
Kubernetes (K8s) is an open-source container orchestration platform that automates the deployment, scaling, and management of containerized applications. Developed by Google, it enables developers to define application states through configuration files, allowing for seamless management across various environments.

# Key Features:
Automated Deployment: Simplifies application deployment and management.
Scaling: Automatically adjusts the number of running containers based on demand.
Self-Healing: Replaces and reschedules containers that fail or become unresponsive.
Service Discovery: Facilitates communication between services within the cluster.
Storage Management: Dynamically allocates storage resources for containers.

Kubernetes is widely adopted for its robustness, flexibility, and strong community support, making it a go-to solution for modern application architectures.

# Architecture Overview:

![image](https://github.com/user-attachments/assets/167d324e-12a5-4232-92a2-151806c71bc6)

Kubernetes architecture consists of several key components that work together to manage containerized applications:

# 1. Master Node
The control plane of the Kubernetes cluster, responsible for managing the cluster’s state.

API Server: The front-end that exposes the Kubernetes API and handles REST commands.
etcd: A distributed key-value store that holds cluster configuration and state data.
Scheduler: Assigns pods to worker nodes based on resource availability.
Controller Manager: Monitors the cluster and ensures the desired state is maintained.
Cloud Controller Manager: Integrates with cloud provider APIs (optional).

# 2. Worker Node
Where application workloads run, hosting necessary services to execute pods.

kubelet: Ensures containers are running as specified by the API server.
Container Runtime: Software for running containers (e.g., Docker).
kube-proxy: Manages network routing and load balancing between services and pods.
Pods: The smallest deployable units containing one or more containers.

# 3. Networking
Facilitates communication within the cluster and with external clients.

Cluster Networking: Enables pod communication across nodes.
Services: Abstracts access to sets of pods for load balancing and service discovery.

# 4. Add-ons
Extensions to enhance Kubernetes functionality.

Dashboard: A web-based UI for managing the cluster.
Monitoring Tools: Tools like Prometheus for tracking performance.
Ingress Controllers: Manage external access to services.
These components work together to provide a robust platform for deploying and managing containerized applications.

# Environment Setup:
To set up a Kubernetes cluster on AWS EC2, follow these steps to prepare your environment:

# 1. Choosing EC2 Instance Types
Recommended Instance Type: Use t2.medium instances for both master and worker nodes to balance cost and performance.
Resource Allocation: Each t2.medium instance provides 2 vCPUs and 4 GB of memory, suitable for a small Kubernetes cluster.
# 2. Selecting the Operating System
Operating System: Use Ubuntu (preferably the latest LTS version) for a stable and widely supported environment.

# Preparing the EC2 Instances:
# Launch EC2 Instances:
Log into the AWS Management Console.
Navigate to the EC2 Dashboard and click “Launch Instance.”
Choose the t2.medium instance type.
Select Ubuntu from the Amazon Machine Images (AMIs) section.

# Configure Instance Details:
Set the number of instances: 2 {1 for the master node and at least 1 for the worker node.}
Configure VPC and subnet settings as needed (use default settings for simplicity).

![image](https://github.com/user-attachments/assets/866d27db-c15a-49fa-9690-d59cb784b86d)

# Configuring Security Groups:
As we launched two instances with same security group ,we have to edit the inbound rules for the worker-node instance.

![image](https://github.com/user-attachments/assets/c15ce4e6-bd51-4dd4-949b-b8aa701613bb)

![image](https://github.com/user-attachments/assets/c54f1f43-cd83-44af-b45c-63089366c80f)

![image](https://github.com/user-attachments/assets/34f53874-fa56-44b7-9192-fd186821d858)

# Add rule : Type- All traffic and source-Anywhere .

![image](https://github.com/user-attachments/assets/63dfb0e5-92a6-4ba6-bd70-4c03c654279f)

# EC2 Connect
Connect both your Master-node and Worker-node using EC2 Connect and establish connection.

![image](https://github.com/user-attachments/assets/46cbc00a-d88a-4ec4-b507-ae13bdbdce1e)

# Basic steps to be done in both Master & worker node after connection.
 “NOTE: I HAVE ONLY ADDED THE SCREENSHOT OF MASTER-NODE BUT SAME STEPS SHOULD BE DONE IN WORKER-NODE ALSO”

# Step 1: Login as root user:
sudo su

# Step 2: Get the updates
apt-get update

# Step 3: Install HTTPS:
Installing an HTTP server enhances the functionality of a Kubernetes cluster, allowing for better traffic management, service exposure, and application health monitoring, making it essential for deploying web-based applications.

apt-get install apt-transport-https

# Installing Docker: (Both in master and worker node)
apt install docker.io -y

# To check docker is installed :
docker — version

# To start the docker:
systemctl start docker
systemctl enable docker

# Install Open GPG Key:
Installing a GPG key during Kubernetes cluster setup is a crucial security measure that ensures the authenticity and integrity of the packages being installed, protecting the cluster from potential threats.

sudo curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add

After installing use command apt-get update

# Edit source list file:
nano /etc/apt/sources.list.d/kubernetes.list
Add the below line in the above file:
deb http://apt.kubernetes.io/ kubernetes-xenial main
Press (Ctrl X Y Enter)

# Setting Up Kubernetes with kubeadm:
Installing kubeadm, kubelet, and kubectl:
sudo apt-get update
 apt-transport-https may be a dummy package; if so, you can skip that package

sudo apt-get install -y apt-transport-https ca-certificates curl gpg

2.  If the directory `/etc/apt/keyrings` does not exist, it should be created before the curl command, read the note below.

 sudo mkdir -p -m 755 /etc/apt/keyrings

curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.31/deb/Release.key | sudo gpg — dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg

3.  This overwrites any existing configuration in /etc/apt/sources.list.d/kubernetes.list

echo ‘deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.31/deb/ /’ | sudo tee /etc/apt/sources.list.d/kubernetes.list

4. sudo apt-get update
sudo apt-get install -y kubelet kubeadm kubectl
sudo apt-mark hold kubelet kubeadm kubectl

5. sudo systemctl enable — now kubelet
Use these above commands to install kubeadm, kubelet, and kubectl

# Initializing the Kubernetes Cluster:
kubeadm init
This command should be given in only MASTER-NODE

After k8’s control-plane successfully initialized it gives you few commands to start the cluster, copy and paste

mkdir -p $HOME/.kube Copy configuration to kube directory (in config file) cp -i /etc/kubernetes/admin.conf HOME/.kube/configProvideuserpermissionstoconfigfilechown(id -u):$(id -g) $HOME/.kube/config

Also copy ,paste the root user command also.

Keep the kubeadm join command as it is we will install network tool and paste te join command in worker-node.

# Networking Setup with Flannel:
Flannel is an open-source networking solution for Kubernetes that provides pod-to-pod communication across nodes in a cluster. It is a CNI (Container Network Interface) plugin, and its main purpose is to set up a reliable overlay network between Kubernetes nodes so that containers (pods) can communicate with each other, even if they are running on different nodes.

# To install flannel
kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml

# After installing networking tool give
kubectl get node

# Joining Worker Node to the Cluster:
Copy the join command in the master-node and paste it in the worker-node to join all the nodes with cluster.

COPY LONG CODE PROVIDED MY MASTER IN NODE NOW LIKE CODE GIVEN BELOW kubeadm join 172.31.14.32:6443 — token mcvd2n.aqvyp59vq3inhtks — discovery-token-ca-cert-hash sha256:9b6b9558ef85d46e805d9a24c4186d7918bced4986cf4503769f46be758e5dc

This command should be give in WORKER-NODE

# Verifying Cluster Status:
Checking Node Status:
To check node status use the below command:

kubectl get nodes

The both master-node and worker-node should show status as Ready.

# Conclusion:
In this hands-on guide, we set up a basic Kubernetes cluster on AWS EC2 using Ubuntu, Docker, and kubeadm, with Flannel for networking. We successfully configured one master node and one worker node, both in ready status. This cluster is now fully operational and ready for deploying containerized applications or scaling with additional nodes. It provides a solid foundation for further Kubernetes exploration on cloud infrastructure.
















