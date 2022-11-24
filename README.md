# tigergraph-deployment
Deploy TigerGraph on Red Hat OpenShift

### TigerGraph Kubectl Plugin

```shell
curl https://dl.tigergraph.com/k8s/0.0.2/kubectl-tg -o kubectl-tg
sudo install kubectl-tg /usr/local/bin/

kubectl tg help
```

### Cert Manager operator

Install the cert manager operator from the Operator Catalog

https://github.com/cert-manager/cert-manager

Current version: 1.10.1
Namespace: openshift-operators

Verify installation of cert-manager

```shell
kubectl wait deployment -n openshift-operators cert-manager --for condition=Available=True --timeout=90s
kubectl wait deployment -n openshift-operators cert-manager-cainjector --for condition=Available=True --timeout=90s
kubectl wait deployment -n openshift-operators cert-manager-webhook --for condition=Available=True --timeout=90s
```


### Install TigerGraph Operator with Kubectl Plugin

```shell
oc new-project tigergraph

kubectl tg init --operator-watch-namespace tigergraph -n tigergraph
```

Wait for operator to be deployed:

```shell
kubectl wait deployment tigergraph-operator-controller-manager --for condition=Available=True --timeout=90s --namespace tigergraph
```


### Remove the default Limit Range

Get the name of the default limit range, if defined:

```shell
oc get limitranges
```

Gives something like this:

```shell
NAME                              CREATED AT
tigergraph-core-resource-limits   2022-11-24T14:45:18Z
```

Delete the limit range:

```shell
oc delete limitrange tigergraph-core-resource-limits
```


### Query the storageclass from the OpenShift cluster

```shell
kubectl get storageclass
```

Gives something like this:

```shell
NAME            PROVISIONER             RECLAIMPOLICY   VOLUMEBINDINGMODE      ALLOWVOLUMEEXPANSION   AGE
gp2 (default)   kubernetes.io/aws-ebs   Delete          WaitForFirstConsumer   true                   8h
gp2-csi         ebs.csi.aws.com         Delete          WaitForFirstConsumer   true                   8h
gp3-csi         ebs.csi.aws.com         Delete          WaitForFirstConsumer   true                   8h
```


### Deploy Tigergraph cluster

```shell
kubectl tg create --namespace tigergraph --cluster-name test-tigergraph-cluster --size 1 --ha 1 --storage-class gp2 --storage-size 50G --cpu 4000m --memory 16Gi
```

**Note:** There is an issue with the UI service and the HA setup which is why the above command only creates a single node cluster ! see [Quickstart with EKS](https://docs.tigergraph.com/tigergraph-server/3.7/kubernetes/quickstart-with-eks) for more details.


### Check the initialization of Tigergraph cluster

```shell
kubectl wait --for=condition=complete --timeout=600s job/test-tigergraph-cluster-init-job --namespace tigergraph
```


### Create the route

Query the available services:

```shell
oc get services
```

Gives something like this:


```shell
NAME                                                     TYPE           CLUSTER-IP       EXTERNAL-IP                                                               PORT(S)           AGE
test-tigergraph-cluster-gui-external-service             LoadBalancer   172.30.153.225   a949aa9b43b5b4acfa75d1b3423d1e0e-43424929.eu-west-1.elb.amazonaws.com     14240:30819/TCP   6m41s
test-tigergraph-cluster-internal-service                 ClusterIP      None             <none>                                                                    9000/TCP,22/TCP   6m40s
test-tigergraph-cluster-rest-external-service            LoadBalancer   172.30.234.251   ad0727914e2724a548b942ffd32a434a-1043887603.eu-west-1.elb.amazonaws.com   9000:31948/TCP    6m41s
tigergraph-operator-controller-manager-metrics-service   ClusterIP      172.30.23.160    <none>                                                                    8443/TCP          10m
tigergraph-operator-webhook-service                      ClusterIP      172.30.194.175   <none>                                                                    443/TCP           10m
```

```shell
oc apply -f tigergraph-ui-route.yaml -n tigergraph
oc apply -f tigergraph-rest-route.yaml -n tigergraph
```


### Checking Cluster Status

```shell
kubectl tg status --cluster-name test-tigergraph-cluster --namespace tigergraph
```


### Delete Clusters

```shell
kubectl tg delete --cluster-name test-tigergraph-cluster
```


### Uninstall TigerGraph Operator

```shell
kubectl tg uninstall
```
