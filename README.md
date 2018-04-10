
### Authorization
Create a cluster level role binding so that you can manipulate the system level namespace

```
kubectl create clusterrolebinding cluster-admin-binding \
 --clusterrole=cluster-admin --user=dan.roscigno@gmail.com
```

### Clone the YAML files
```
git clone git@github.com:DanRoscigno/GuestbookExample.git
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

