# Kubernetes-Microservices-Helm-Istio-Grafan-Prometheus
# Microservices 
I used an open-source benchmark for cloud microservices called DeathStarBench, which includes five end-to-end services, four for cloud  systems, and one for cloud-edge systems running on drone swarms. The social networking end-to-end service is assumed to run on Google Cloud Platform in this repository.For more information about the benchmeack, please follow the link:

https://github.com/delimitrou/DeathStarBench

## Social Network Microservices

A social network with unidirectional follow relationships built with loosely coupled microservices that communicate with one another via Thrift RPCs.

### Application Structure
![microservice](https://user-images.githubusercontent.com/87664653/162190639-44f4ae4e-6fbc-4cf4-9158-f333ecfc0723.png)

To run microservices on GCP, follow the steps outlined below.

1. If you do not have a GCP account, please open one on GCP.

2. Set up Docker and the Kubernetes cluster nodes on Google Cloud Platform.

3. Attach the helm

4. Setup Grafana/Prometheus

5. Set up Istio

6. Execute Microservices

7. Grafana/Prometheus monitoring

## Create a Kubernetes cluster (master node and workers) on the Google Cloud Platform (GCP).

1. Create a new project on GCP.Therefore, I created a project called "Kubernetes". If you do not know how to create a project, please follow the following link:\
https://cloud.google.com/resource-manager/docs/creating-managing-projects

2. To run gcloud commands which will be mentioned in the next steps, there are some options:

3. cloud shell on GCP (this is the one I used).
4. Downloading and installing the gcloud CLI on your host. follow the link if you prefer to install gcloud cli on your own machine and run the gcloud commands:
https://cloud.google.com/sdk/docs/install

5. Click the "activate cloud shell" button, and then run commands in the shell.

6. Set ProjectID in gcloud for example: kubernetes-342214.

    How to find Project ID:
    https://support.google.com/googleapi/answer/7014113?hl=en

    ```
    gcloud config set project <your_ProjectID>

    ```
    Example:
    ```
    gcloud config set project kubernetes-342214
    ```

7. Set the zone property in the compute section
    ```
    gcloud config set compute/zone <zone name >
    ```
    Exmaple:
    ```
     gcloud config set compute/zone us-central1-a
    ```
    How to find the available regions and zones:
    https://cloud.google.com/compute/docs/regions-zones#:~:text=You%20can%20use%20the%20Google,a%20specific%20region%20or%20zone.


8. Build the VPC
    ```
    gcloud compute networks create k8s-cluster --subnet-mode custom
    ```


9. In the k8s-cluster VPC network, create the k8s-nodes subnet. To set IP ranges, go to the VPC network section on GCP to find subnets, or please follow the link to find your region's IP address. For example, the address of us-central1 is 10.128.0.0/20:
https://cloud.google.com/vpc/docs/subnets
    ```
    gcloud compute networks subnets create k8s-nodes --network k8s-cluster --range <your region's IP address>
    ```
    Example:
    ```
    gcloud compute networks subnets create k8s-nodes --network k8s-cluster --range 10.128.0.0/24
    ```

10. Set a firewall rule that enables internal communication through TCP, UDP, ICMP, and IP.
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
      --source-ranges 10.128.0.0/11
    ```

11. Make a firewall rule that enables external SSH, ICMP, and HTTPS connections.
    ```
    gcloud compute firewall-rules create k8s-cluster-allow-external \
      --allow tcp:22,tcp:6443,icmp \
      --network k8s-cluster \
      --source-ranges 0.0.0.0/0
    ```

12. Create a Master Node Instance on GCP:
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
13. Create Two worker Instances on GCP:
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


13. On Worker Nodes Execute the Join Command


14. Verify the Cluster Status

kubectl get nodes


15. On the controller, install Calico from the manifest:

curl https://docs.projectcalico.org/manifests/calico.yaml -O

kubectl apply -f calico.yaml 

```


1.
Download Microservice 
```
git clone https://github.com/delimitrou/DeathStarBench.git
```
Step 1: Head over to the Github helm release page and copy the Linux amd64 link for the required version.

https://github.com/helm/helm/releases
2.
```
wget -O helm.tar.gz  https://get.helm.sh/helm-v3.8.1-linux-amd64.tar.gz
```
3.
```
wget -O helm.tar.gz  https://get.helm.sh/helm-v3.8.1-linux-amd64.tar.gz

```
4.
```
tar -zxvf helm.tar.gz
```
5.
```
sudo mv linux-amd64/helm /usr/local/bin/helm
```
6.
```
helm
```
7.
```
helm repo add stable https://charts.helm.sh/stable

```
https://devopscube.com/install-configure-helm-kubernetes/

## lstio
https://istio.io/latest/docs/setup/getting-started/
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
https://istio.io/latest/docs/tasks/observability/metrics/using-istio-dashboard/
## Grafana
```
kubectl apply -f https://raw.githubusercontent.com/istio/istio/release-1.13/samples/addons/grafana.yaml
```
## prometheus
```
kubectl apply -f https://raw.githubusercontent.com/istio/istio/release-1.13/samples/addons/prometheus.yaml
```


Error: INSTALLATION FAILED: failed post-install: warning: Hook post-install social-network/templates/mongodb-sharded-init/post-install-hook.yaml failed: Internal error occurred: failed calling webhook "namespace.sidecar-injector.istio.io": failed to call webhook: Post "https://istiod.istio-system.svc:443/inject?timeout=10s": context deadline exceeded
helm.go:84: [debug] failed post-install: warning: Hook post-install social-network/templates/mongodb-sharded-init/post-install-hook.yaml failed: Internal error occurred: failed calling webhook "namespace.sidecar-injector.istio.io": failed to call webhook: Post "https://istiod.istio-system.svc:443/inject?timeout=10s": context deadline exceeded
INSTALLATION FAILED
main.newInstallCmd.func2

```
sudo apt install build-essential make openssl libssl-dev zlib1g-dev luarocks
```
luarocks install luasocket
```

```
