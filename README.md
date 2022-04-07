# Kubernetes-Microservices-Helm-Istio
# Microservices 
I employed an open-source benchmark for cloud microservices called DeathStarBench, which includes five end-to-end services, four for cloud \ systems, and one for cloud-edge systems running on drone swarms. The social networking end-to-end service is assumed to run on Google Cloud\ Platform in this repository.For more information about the benchmeack, please follow the link:\
```
https://github.com/delimitrou/DeathStarBench

```
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

## Install kuberentes on Google cloud platform (GCP)
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
