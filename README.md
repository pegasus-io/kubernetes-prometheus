# Purpose

This is a POC of how to use Prometheus to monitor :

*  **Kubernetes Cluster 1 (production)** : user facing service
  * 2 namespaces : one for the app, one docker oci images app distribution
  * a k8S cluster
  * in the K8S CLuster, a `node` / `mongodb` app is deployed,
  * antoher service, a minio S3 storage cluster, is deployed for use of the node app
  * Prometheus monitors mongo cluster, minio cluster, and app workflows
  * in the K8S Cluster, a Private docker registry with `Portus` service is deployed, (a second POC will replace Portus with Harbor) : users use it to docker pull images as part of theur subscribe plan . This is a Distributionb Channel, and the Kubernetes Cluster IS NOT THE ONE FROM WHICH OCI IMAGES ARE PULLED.
* Cluster 2 : private autodevops service
  * a k8S cluster, with 5 namespaces :
    * dev
    * integration
    * staging (pre-production)
    * devops (sprinkling a bit of magic)
    * production (making money) **Kubernetes Cluster 1 (production)**
  * in default namespace :
    * a Notary Service is deployed, used to sign oci images across all namespaces
    * A Keycloak Service is deployed, to manage Identity across all namespaces
    * a HashiCorp Vault is deployed, for default namespace services secret management, in addition to a `SMACK` rotator.
    * A `Rocketchat` Service is deployed
  * in each namespace :
    * A `HashiCorp Vault` is deployed, for the namespace's services secret management. Root secrets for this vault, are pulled from the vault in the default namespace, to which only high level administrators ahve access, through higly secured, auditable access control.
    * In the K8S Cluster, a Private docker registry with `Portus` service is deployed, (a second POC will replace Portus with Harbor)
    * Antoher service, a minio S3 storage cluster, is deployed for use as storage of the private docker registry
    * A `clair scanner` is deployed with the portus service, to assess security improvement across namespaces
    * Prometheus monitors mongo cluster, minio cluster, and app workflows


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

What you have now, is a Prometheus service, properly deployed to `Kubernetes`. But its config is empty.

We need to configure it now :
* To monitor `minio` :
* To monitor `Mongodb` :

# Scale out Prometheus

* here is the replicaset deployed :

```bash
jbl@poste-devops-jbl-16gbram:~/kubernetes-prometheus$ kubectl get replicasets -n monitoring
NAME                               DESIRED   CURRENT   READY   AGE
prometheus-deployment-77cb49fb5d   1         1         1       49m
jbl@poste-devops-jbl-16gbram:~/kubernetes-prometheus$ kubectl describe replicaset prometheus-deployment-77cb49fb5d -n monitoring
Name:           prometheus-deployment-77cb49fb5d
Namespace:      monitoring
Selector:       app=prometheus-server,pod-template-hash=77cb49fb5d
Labels:         app=prometheus-server
                pod-template-hash=77cb49fb5d
Annotations:    deployment.kubernetes.io/desired-replicas: 1
                deployment.kubernetes.io/max-replicas: 2
                deployment.kubernetes.io/revision: 1
Controlled By:  Deployment/prometheus-deployment
Replicas:       1 current / 1 desired
Pods Status:    1 Running / 0 Waiting / 0 Succeeded / 0 Failed
Pod Template:
  Labels:  app=prometheus-server
           pod-template-hash=77cb49fb5d
  Containers:
   prometheus:
    Image:      prom/prometheus
    Port:       9090/TCP
    Host Port:  0/TCP
    Args:
      --config.file=/etc/prometheus/prometheus.yml
      --storage.tsdb.path=/prometheus/
    Environment:  <none>
    Mounts:
      /etc/prometheus/ from prometheus-config-volume (rw)
      /prometheus/ from prometheus-storage-volume (rw)
  Volumes:
   prometheus-config-volume:
    Type:      ConfigMap (a volume populated by a ConfigMap)
    Name:      prometheus-server-conf
    Optional:  false
   prometheus-storage-volume:
    Type:       EmptyDir (a temporary directory that shares a pod s lifetime)
    Medium:
    SizeLimit:  <unset>
Events:
  Type    Reason            Age   From                   Message
  ----    ------            ----  ----                   -------
  Normal  SuccessfulCreate  51m   replicaset-controller  Created pod: prometheus-deployment-77cb49fb5d-kjpbr
jbl@poste-devops-jbl-16gbram:~/kubernetes-prometheus$
```
* so to scale it out :
```bash
# ---
# kubectl scale replicasets nzmr --replicas=5
export REPLICASET_NAME=""
export REPLICASET_NAME=$(kubectl describe replicaset prometheus-deployment-77cb49fb5d -n monitoring|grep 'Name:'|grep -v 'conf'|awk '{print $2}')
export POD_NUMBER=$(kubectl describe replicaset prometheus-deployment-77cb49fb5d -n monitoring|grep 'Replicas:'|awk '{print $2}')
echo " Your prometheus replicaSet is  : [${REPLICASET_NAME}], has [${POD_NUMBER}] and we'll scale it to [5] "


kubectl scale --replicas=5 rs/${REPLICASET_NAME} -n monitoring
# will scale out, and then automatically scale down due to deployment definition setting replica no to 1
export DEPLOYMENT_NAME=$(kubectl get deployments -n monitoring|tail -n 1|awk '{print $1}')
kubectl scale --replicas=10 deployment/${DEPLOYMENT_NAME} -n monitoring

# And bam now this all scaled out nicely, without errors for me, uisng a 4 vCPU / 8Gb RAM VM
```

# kubernetes-prometheus

Configuration files for setting up prometheus monitoring on Kubernetes cluster.

You can find the full tutorial from here https://devopscube.com/setup-prometheus-monitoring-on-kubernetes/
