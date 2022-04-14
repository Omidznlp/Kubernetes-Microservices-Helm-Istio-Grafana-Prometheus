# Kubernetes-Microservices-Helm-Istio-Grafan-Prometheus

## Microservices 

I used an open-source benchmark for cloud microservices called DeathStarBench, which includes five end-to-end services, four for cloud  systems, and one for cloud-edge systems running on drone swarms. The social networking end-to-end service is assumed to run on Google Cloud Platform in this repository.For more information about the benchmeack, please follow the link:

https://github.com/delimitrou/DeathStarBench

## Social Network Microservices

A social network with unidirectional follow relationships built with loosely coupled microservices that communicate with one another via Thrift RPCs.

### Application Structure

![microservice](https://user-images.githubusercontent.com/87664653/162190639-44f4ae4e-6fbc-4cf4-9158-f333ecfc0723.png)

To run microservices on GCP, follow the steps outlined below.

1. If you do not have a GCP account, please open one on GCP.
2. Set up master and worker nodes on GCP
3. Set up Docker and the Kubernetes cluster nodes on GCP.

4. Attach the helm

5. Setup Grafana/Prometheus

6. Set up Istio

7. Execute Microservices

8. Grafana/Prometheus monitoring

## Create a Kubernetes cluster (master node and workers) on the Google Cloud Platform (GCP).

1. Create a new project on GCP.Therefore, I created a project called "Kubernetes". If you do not know how to create a project, please follow the following link:\
https://cloud.google.com/resource-manager/docs/creating-managing-projects

2. To run gcloud commands which will be mentioned in the next steps, there are some options:

    2.1 cloud shell on GCP (this is the one I used).\
    2.2 Downloading and installing the gcloud CLI on your host. follow the link if you prefer to install gcloud cli on your own machine and run the gcloud commands:
    https://cloud.google.com/sdk/docs/install

3. Click the "activate cloud shell" button, and then run the following commands in the shell.

4. Set ProjectID in gcloud for example: kubernetes-342214.

    How to find Project ID:
    https://support.google.com/googleapi/answer/7014113?hl=en

    ```
    gcloud config set project <your_ProjectID>

    ```
    Example:
    ```
    gcloud config set project kubernetes-342214
    ```

5. Set the zone property in the compute section
    ```
    gcloud config set compute/zone <zone name >
    ```
    Exmaple:
    ```
     gcloud config set compute/zone us-central1-a
    ```
    How to find the available regions and zones:
    https://cloud.google.com/compute/docs/regions-zones#:~:text=You%20can%20use%20the%20Google,a%20specific%20region%20or%20zone.


6. Build the VPC
    ```
    gcloud compute networks create k8s-cluster --subnet-mode custom
    ```


7. In the k8s-cluster VPC network, create the k8s-nodes subnet. To set IP ranges, go to the VPC network section on GCP to find subnets, or please follow the link to find your region's IP address. For example, the address of us-central1 is 10.128.0.0/20:
https://cloud.google.com/vpc/docs/subnets
    ```
    gcloud compute networks subnets create k8s-nodes --network k8s-cluster --range <your region's IP address>
    ```
    Example:
    ```
    gcloud compute networks subnets create k8s-nodes --network k8s-cluster --range 10.128.0.0/20
    ```

8. Set a firewall rule that enables internal communication through TCP, UDP, ICMP, and IP.
    ```
    gcloud compute firewall-rules create k8s-cluster-allow-internal \
      --allow tcp,udp,icmp,ipip \
      --network k8s-cluster \
      --source-ranges <your region's IP address>
    ```
    Example
    ```
    gcloud compute firewall-rules create k8s-cluster-allow-internal \
      --allow tcp,udp,icmp,ipip \
      --network k8s-cluster \
      --source-ranges 10.128.0.11/20
    ```

9. Make a firewall rule that enables external SSH, ICMP, and HTTPS connections.
    ```
    gcloud compute firewall-rules create k8s-cluster-allow-external \
      --allow tcp:22,tcp:6443,icmp \
      --network k8s-cluster \
      --source-ranges 0.0.0.0/0
    ```

10. Create a Master Node Instance on GCP:
    ```
    gcloud compute instances create master-node \
        --async \
        --boot-disk-size 100GB \
        --can-ip-forward \
        --image-family ubuntu-1804-lts \
        --image-project ubuntu-os-cloud \
        --machine-type n1-standard-2 \
        --private-network-ip 10.128.0.11 \
        --scopes compute-rw,storage-ro,service-management,service-control,logging-write,monitoring \
        --subnet k8s-nodes \
        --zone us-central1-a \
        --tags k8s-cluster,master-node,controller
    ```
11. Create Two worker Instances on GCP:
    ```
    for i in 0 1; do
      gcloud compute instances create workernode-${i} \
        --async \
        --boot-disk-size 100GB \
        --can-ip-forward \
        --image-family ubuntu-1804-lts \
        --image-project ubuntu-os-cloud \
        --machine-type n1-standard-2 \
        --private-network-ip 10.128.0.2${i} \
        --scopes compute-rw,storage-ro,service-management,service-control,logging-write,monitoring \
        --subnet k8s-nodes \
        --zone us-central1-a \
        --tags k8s-cluster,worker
    done
    ```

## Install Docker and kuberentes on Google cloud platform (GCP)

Connect to the master and worker nodes via ssh.
How to connect via ssh:\
https://cloud.google.com/compute/docs/instances/connecting-to-instance

### Docker Installation on Master and Workers nodes

Install Docker on the master and worker nodes.\

1. Update pakages

    ```
    sudo apt-get update
    ```

2. Install dependencies

    ```
    sudo apt-get install ca-certificates curl gnupg lsb-release    
    ```

3. Add Docker’s official GPG key:

    ```
    curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
    ```

4. Use the following command to set up the stable repository.

    ```
    echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu \
    $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
    ```

5. Update the apt package index.

    ```
    sudo apt-get update
    ```
6. Install docker.

    ```
    sudo apt-get install docker-ce docker-ce-cli containerd.io
    ```

7. Enable docker service

    ```
    sudo systemctl enable docker 
    ```

8. Verfiy installation

    ```
    docker ––version
    ```

Read more information about the commands at the following link:

<https://docs.docker.com/engine/install/ubuntu/>

### kubernetes Installation on Master and Workers nodes

Install kubernetes on the master and worker nodes.\
1.
```
cat <<EOF | sudo tee /etc/modules-load.d/k8s.conf
br_netfilter
EOF

cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
EOF
```
```
sudo sysctl --system
```
2.
```
sudo apt-get update
sudo apt-get install -y apt-transport-https ca-certificates curl
```
3.
```
sudo curl -fsSLo /usr/share/keyrings/kubernetes-archive-keyring.gpg https://packages.cloud.google.com/apt/doc/apt-key.gpg
```
4.
```
echo "deb [signed-by=/usr/share/keyrings/kubernetes-archive-keyring.gpg] https://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee /etc/apt/sources.list.d/kubernetes.list

```
5.
```
sudo apt-get update
sudo apt-get install -y kubelet kubeadm kubectl
sudo apt-mark hold kubelet kubeadm kubectl
```
6.
```
sudo swapoff -a
sudo sed -i '/ swap / s/^/#/' /etc/fstab
```
7.Create this file "daemon.json" in the directory "/etc/docker" and add the following
```
{
"exec-opts": ["native.cgroupdriver=systemd"]
}
```
8.
```
sudo systemctl restart docker
sudo kubeadm reset
```

### Kuberenetes Configuration on Master Node

1. The following procedure should only be applied on the master node, and only the "kubeadm join" command should be executed on worker nodes.

```
sudo kubeadm init --pod-network-cidr=10.10.0.0/16 
```
Seeing the following lines at the terminal
```
[init] Using Kubernetes version: v1.23.5
[preflight] Running pre-flight checks
[preflight] Pulling images required for setting up a Kubernetes cluster
[preflight] This might take a minute or two, depending on the speed of your internet connection
[preflight] You can also perform this action in beforehand using 'kubeadm config images pull'
[certs] Using certificateDir folder "/etc/kubernetes/pki"
[certs] Generating "ca" certificate and key
[certs] Generating "apiserver" certificate and key
[certs] apiserver serving cert is signed for DNS names [jenkins-agent kubernetes kubernetes.default kubernetes.default.svc kubernetes.default.svc.cluster.local] and IPs [10.96.0.1 10.0.2.15]
[certs] Generating "apiserver-kubelet-client" certificate and key
[certs] Generating "front-proxy-ca" certificate and key
[certs] Generating "front-proxy-client" certificate and key
[certs] Generating "etcd/ca" certificate and key
[certs] Generating "etcd/server" certificate and key
[certs] etcd/server serving cert is signed for DNS names [jenkins-agent localhost] and IPs [10.0.2.15 127.0.0.1 ::1]
[certs] Generating "etcd/peer" certificate and key
[certs] etcd/peer serving cert is signed for DNS names [jenkins-agent localhost] and IPs [10.0.2.15 127.0.0.1 ::1]
[certs] Generating "etcd/healthcheck-client" certificate and key
[certs] Generating "apiserver-etcd-client" certificate and key
[certs] Generating "sa" key and public key
[kubeconfig] Using kubeconfig folder "/etc/kubernetes"
[kubeconfig] Writing "admin.conf" kubeconfig file
[kubeconfig] Writing "kubelet.conf" kubeconfig file
[kubeconfig] Writing "controller-manager.conf" kubeconfig file
[kubeconfig] Writing "scheduler.conf" kubeconfig file
[kubelet-start] Writing kubelet environment file with flags to file "/var/lib/kubelet/kubeadm-flags.env"
[kubelet-start] Writing kubelet configuration to file "/var/lib/kubelet/config.yaml"
[kubelet-start] Starting the kubelet
[control-plane] Using manifest folder "/etc/kubernetes/manifests"
[control-plane] Creating static Pod manifest for "kube-apiserver"
[control-plane] Creating static Pod manifest for "kube-controller-manager"
[control-plane] Creating static Pod manifest for "kube-scheduler"
[etcd] Creating static Pod manifest for local etcd in "/etc/kubernetes/manifests"
[wait-control-plane] Waiting for the kubelet to boot up the control plane as static Pods from directory "/etc/kubernetes/manifests". This can take up to 4m0s
[apiclient] All control plane components are healthy after 10.509242 seconds
[upload-config] Storing the configuration used in ConfigMap "kubeadm-config" in the "kube-system" Namespace
[kubelet] Creating a ConfigMap "kubelet-config-1.23" in namespace kube-system with the configuration for the kubelets in the cluster
NOTE: The "kubelet-config-1.23" naming of the kubelet ConfigMap is deprecated. Once the UnversionedKubeletConfigMap feature gate graduates to Beta the default name will become just "kubelet-config". Kubeadm upgrade will handle this transition transparently.
[upload-certs] Skipping phase. Please see --upload-certs
[mark-control-plane] Marking the node jenkins-agent as control-plane by adding the labels: [node-role.kubernetes.io/master(deprecated) node-role.kubernetes.io/control-plane node.kubernetes.io/exclude-from-external-load-balancers]
[mark-control-plane] Marking the node jenkins-agent as control-plane by adding the taints [node-role.kubernetes.io/master:NoSchedule]
[bootstrap-token] Using token: qp794c.xbsi5nanw2u9sn9x
[bootstrap-token] Configuring bootstrap tokens, cluster-info ConfigMap, RBAC Roles
[bootstrap-token] configured RBAC rules to allow Node Bootstrap tokens to get nodes
[bootstrap-token] configured RBAC rules to allow Node Bootstrap tokens to post CSRs in order for nodes to get long term certificate credentials
[bootstrap-token] configured RBAC rules to allow the csrapprover controller automatically approve CSRs from a Node Bootstrap Token
[bootstrap-token] configured RBAC rules to allow certificate rotation for all node client certificates in the cluster
[bootstrap-token] Creating the "cluster-info" ConfigMap in the "kube-public" namespace
[kubelet-finalize] Updating "/etc/kubernetes/kubelet.conf" to point to a rotatable kubelet client certificate and key
[addons] Applied essential addon: CoreDNS
[addons] Applied essential addon: kube-proxy

Your Kubernetes control-plane has initialized successfully!

To start using your cluster, you need to run the following as a regular user:

  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config

Alternatively, if you are the root user, you can run:

  export KUBECONFIG=/etc/kubernetes/admin.conf

You should now deploy a pod network to the cluster.
Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
  https://kubernetes.io/docs/concepts/cluster-administration/addons/

Then you can join any number of worker nodes by running the following on each as root:
kubeadm join 10.128.0.11:6443 --token qp794c.xbsi5nanw2u9sn9x \
	--discovery-token-ca-cert-hash sha256:355a4ca26e908ddc939b8476377f17d1133a80132eab7db07655f6fa2bacd6e2 

```

2. To start using your cluster, you need to run the following as a regular user

```
  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config
```
3. To deploy the Network model

```
sudo kubectl apply -f https://docs.projectcalico.org/manifests/calico.yaml
```
The other network model can be found at the following link.\
 
 https://kubernetes.io/docs/concepts/cluster-administration/networking/

### Kuberenetes Configuration on Worker Nodes

To join worker nodes to the cluster, please insert the following command in the worker nodes. (Please run your own command which is printed onscreen in the last step.) 

```
kubeadm join 10.128.0.11:6443 --token qp794c.xbsi5nanw2u9sn9x \
	--discovery-token-ca-cert-hash sha256:355a4ca26e908ddc939b8476377f17d1133a80132eab7db07655f6fa2bacd6e2 
```

## Configurations of Master Node

### Helm Installation

Head over to the Github helm release page and copy the Linux amd64 link for the required version.
link:\
<https://github.com/helm/helm/releases>
1.
```
wget -O helm.tar.gz  https://get.helm.sh/helm-v3.8.1-linux-amd64.tar.gz
```
2.
```
wget -O helm.tar.gz  https://get.helm.sh/helm-v3.8.1-linux-amd64.tar.gz

```
3.
```
tar -zxvf helm.tar.gz
```
4.
```
sudo mv linux-amd64/helm /usr/local/bin/helm
```

5.verify helm installation

```
helm version 
```
7.
```
helm repo add stable https://charts.helm.sh/stable

```

### Istio Installation


1.
```
curl -L https://istio.io/downloadIstio | sh -

```
2.
```
cd istio-1.13.2
```
3.
```
export PATH=$PWD/bin:$PATH

```
3.
```
istioctl install --set profile=demo -y
```
```
✔ Istio core installed                                                          
✔ Istiod installed                                                              
✔ Ingress gateways installed                                                    
✔ Egress gateways installed                                                     
✔ Installation complete                                                        
Making this installation the default for injection and validation.

Thank you for installing Istio 1.13.  Please take a few minutes to tell us about your install/upgrade experience!  https://forms.gle/pzWZpAvMVBecaQ9h9

```
4.
```
kubectl label namespace default istio-injection=enabled
```
For more information:
How to set up Istio: \
<https://istio.io/latest/docs/setup/getting-started/>

### Grafana Installation

```
kubectl apply -f https://raw.githubusercontent.com/istio/istio/release-1.13/samples/addons/grafana.yaml
```
### Prometheus Installation
```
kubectl apply -f https://raw.githubusercontent.com/istio/istio/release-1.13/samples/addons/prometheus.yaml
```

### Download Microservices Repository

```
git clone https://github.com/Omidznlp/DeathStarBench.git
```

### Install Microservices via Helm

```
cd DeathStarBench/socialNetwork/helm-chart
```

```
helm install social-media socialnetwork/ --values socialnetwork/values.yaml
```

### Remote Desktop for Master node

There are two ways to run Dashboard in a browser:

1. On the master node, install Chrome Remote Desktop (I used this one). Please follow the "Install Chrome Remote Desktop on the VM instance" section at the following link to run Chrome Remote Desktop on the master node.\
 
FYI: https://ubuntu.com/blog/launch-ubuntu-desktop-on-google-cloud/

2. Configure the forwarding rule to route the output port to the input ports.\
FYI:
<https://cloud.google.com/load-balancing/docs/protocol-forwarding>
    ```
    <public ip address of master node:port> -> (map to) the ip addresss of nginx-thrift service:8080
    ```
    ```
    <public ip address of master node:port> -> (map to) 127.0.0.1:3000 (grafana dashboard)
    ```

### Run Death Star Social Media

See the ip addresss of Social Media App(the nginx-thrift service on port 8080). Now, you can visit <http://nginx-thrift_service_ip_address:8080> to see the front end.

```
kubectl get services
```

![see ip address](https://user-images.githubusercontent.com/87664653/162442193-02fdc954-c576-446e-b69c-e0bdcce35492.png)

![deathstart](https://user-images.githubusercontent.com/87664653/162442072-f728e43c-e294-470c-bafb-ec0388c15488.png)

![deathstar_in](https://user-images.githubusercontent.com/87664653/162442086-dc9e63f1-e00b-4437-9574-08e6e8a9340f.png)

### Run Grafana Dashboard

Open the master node in remote desktop mode and run the following command.
```
cd istio-1.13.2
```
```
istioctl dashboard grafana
```
The Grafana dashboard appears in the browser automatically.

### Running HTTP Workload Generator

#### Install Dependencies

```
cd DeathStarBench/socialNetwork
```
```
sudo apt install build-essential make openssl libssl-dev zlib1g-dev luarocks
```
```
luarocks install luasocket
```
```
cd wrk2
make
```
```
cd wrk2
./wrk -D exp -t <num-threads> -c <num-conns> -d <duration> -L -s ./scripts/social-network/compose-post.lua http://localhost:8080/wrk2-api/post/compose -R <reqs-per-sec>
```
Example:
```
./wrk -D exp -t 3 -c 3 -d 100 -L -s ./scripts/social-network/compose-post.lua http://10.110.246.16:8080/wrk2-api/post/compose -R 10
```

### Monitoring

Choose the nginx-thrift service for monitoring.
![service](https://user-images.githubusercontent.com/87664653/162442391-971410ec-aa4b-4727-802c-c6c31afa5672.png)

![grafana](https://user-images.githubusercontent.com/87664653/162441959-f56f1af6-e15e-444f-ac9c-8d24041d5d13.png)


## Troubelshooting

If the following error happens in the helm install step:
```
Error: INSTALLATION FAILED: failed post-install: warning: Hook post-install social-network/templates/mongodb-sharded-init/post-install-hook.yaml failed: Internal error occurred: failed calling webhook "namespace.sidecar-injector.istio.io": failed to call webhook: Post "https://istiod.istio-system.svc:443/inject?timeout=10s": context deadline exceeded
helm.go:84: [debug] failed post-install: warning: Hook post-install social-network/templates/mongodb-sharded-init/post-install-hook.yaml failed: Internal error occurred: failed calling webhook "namespace.sidecar-injector.istio.io": failed to call webhook: Post "https://istiod.istio-system.svc:443/inject?timeout=10s": context deadline exceeded
INSTALLATION FAILED
main.newInstallCmd.func2

```
Please insert the following command:

```
cd istio-1.13.2
export PATH=$PWD/bin:$PATH
```
