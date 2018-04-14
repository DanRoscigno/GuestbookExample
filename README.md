#### Note: These instructions may be GCP specific, this is where I have experience with Docker and Kubernetes.

### Create an Elastic Cloud deployment
You can use Elastic Cloud ( http://cloud.elastic.co ), or a local deployment, or deploy containers from https://www.elastic.co/guide/en/elasticsearch/reference/current/docker.html

If this is your first experience with the Elastic stack I would recommend Elastic Cloud; and don't worry, you do not need a credit card.

Make sure that you take note of the CLOUD ID and Elastic Password if you use Elastic Cloud or Elastic Cloud Enterprise.

### Connect to your Kubernetes environment
In my case this is the Google Cloud Kubernetes Engine.  I like this environment, and I love the free quota that they give out.  If you use a local environment open a terminal, I will use the console provided by Google.

### Authorization
Create a cluster level role binding so that you can manipulate the system level namespace

```
kubectl create clusterrolebinding cluster-admin-binding \
 --clusterrole=cluster-admin --user=<your email associated with GCP>
```

### Clone the YAML files
```
git clone https://github.com/DanRoscigno/GuestbookExample.git
cd GuestbookExample
```
### Set the credentials
Set these with the values from the http://cloud.elastic.co deployment

```
vi ELASTIC_PASSWORD
vi CLOUD_ID
```
and create a secret in the Kubernetes system level namespace

```
kubectl create secret generic dynamic-logging \
--from-file=./ELASTIC_PASSWORD --from-file=./CLOUD_ID \
--namespace=kube-system
```

### Check to see if kube-state-metrics is running
```
kubectl get pods --namespace=kube-system | grep kube-state
```
and create it if needed (by default it will not be there)

```
go get k8s.io/kube-state-metrics
cd /home/dan_roscigno/gopath/src/k8s.io/kube-state-metrics
make container
kubectl create -f kubernetes
kubectl get pods --namespace=kube-system | grep kube-state 
```

### Deploy the Guestbook example
Note: This is mostly the default Guestbook example from https://github.com/kubernetes/examples/blob/master/guestbook/all-in-one/guestbook-all-in-one.yaml

I added an ingress that preserves source IPs and added ConfigMaps for the Apache2 and Mod-Status configs so that I could block the /server-status endpoint from outside the internal network (actually apache2.conf is unedited, but I may need it later).

```
kubectl create -f guestbook.yaml 
```
Verify the external IP is assigned

```
kubectl get service frontend -w
```
### Deploy the Elastic Beats
```
kubectl create -f filebeat-kubernetes.yaml 
kubectl create -f metricbeat-kubernetes.yaml 
kubectl create -f packetbeat-kubernetes.yaml 
```

