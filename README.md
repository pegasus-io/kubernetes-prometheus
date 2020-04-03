
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

# kubernetes-prometheus

Configuration files for setting up prometheus monitoring on Kubernetes cluster.

You can find the full tutorial from here https://devopscube.com/setup-prometheus-monitoring-on-kubernetes/
