
# Origins

I started the present `git` repository from https://github.com/bibinwilson/kubernetes-prometheus, cmmit id `b1f2e73ee00bea1234b59f40fea81a3cbbf9e6b1`


# Run it against your own cluster

Assuming your `KUBECONFIG` context is set, to access your cluster for example using `kubectl proxy`

```bash
export THIS_REPO=git@github.com:pegasus-io/kubernetes-prometheus.git
export OPS_HOME=~/.k8s.prometheus

git clone ${THIS_REPO} ${OPS_HOME}

cd ${OPS_HOME}

kubectl create namespace monitoring
kubectl create --save-config -f ./
kubectl apply -f ./
```

* example output :

```bash
jbl@poste-devops-jbl-16gbram:~/kubernetes-prometheus$ kubectl create namespace monitoring
namespace/monitoring created
jbl@poste-devops-jbl-16gbram:~/kubernetes-prometheus$ kubectl create --save-config -f ./
clusterrole.rbac.authorization.k8s.io/prometheus created
clusterrolebinding.rbac.authorization.k8s.io/prometheus created
configmap/prometheus-server-conf created
deployment.apps/prometheus-deployment created
ingress.extensions/prometheus-ui created
secret/prometheus-secret created
service/prometheus-service created
jbl@poste-devops-jbl-16gbram:~/kubernetes-prometheus$ kubectl apply -f ./
clusterrole.rbac.authorization.k8s.io/prometheus unchanged
clusterrolebinding.rbac.authorization.k8s.io/prometheus unchanged
configmap/prometheus-server-conf unchanged
deployment.apps/prometheus-deployment unchanged
ingress.extensions/prometheus-ui unchanged
secret/prometheus-secret unchanged
service/prometheus-service unchanged
jbl@poste-devops-jbl-16gbram:~/kubernetes-prometheus$
```
* For testing purposes on this repo, I deployed a `minio` S3 storage service in the default namespace, and the `prometheus` in the `monitoring` namespace, and the output reveals exposition :

```bash
jbl@poste-devops-jbl-16gbram:~/kubernetes-prometheus$ kubectl get pods,svc
NAME          READY   STATUS    RESTARTS   AGE
pod/minio-0   1/1     Running   0          25h
pod/minio-1   1/1     Running   0          25h
pod/minio-2   1/1     Running   0          25h
pod/minio-3   1/1     Running   0          25h

NAME                    TYPE           CLUSTER-IP       EXTERNAL-IP   PORT(S)          AGE
service/kubernetes      ClusterIP      10.96.0.1        <none>        443/TCP          25h
service/minio           ClusterIP      None             <none>        9000/TCP         25h
service/minio-service   LoadBalancer   10.102.197.151   <pending>     9000:30138/TCP   25h
jbl@poste-devops-jbl-16gbram:~/kubernetes-prometheus$ kubectl get pods,svc -n monitoring
NAME                                         READY   STATUS    RESTARTS   AGE
pod/prometheus-deployment-77cb49fb5d-kjpbr   1/1     Running   0          2m4s

NAME                         TYPE       CLUSTER-IP      EXTERNAL-IP   PORT(S)          AGE
service/prometheus-service   NodePort   10.102.135.95   <none>        8080:30000/TCP   2m4s
jbl@poste-devops-jbl-16gbram:~/kubernetes-prometheus$
```

* And now your `prometheus`, k8s deployed, is reachable through nodePort Service exposition at :

```bash
export K8S_CLUSTER_API_SERVER_HOST=minikube.pegasusio.io
export K8S_PROMETHEUS_PORT=30000

firefox http://${K8S_CLUSTER_API_SERVER_HOST}:${K8S_PROMETHEUS_PORT}/graph

```


# kubernetes-prometheus

Configuration files for setting up prometheus monitoring on Kubernetes cluster.

You can find the full tutorial from here https://devopscube.com/setup-prometheus-monitoring-on-kubernetes/
