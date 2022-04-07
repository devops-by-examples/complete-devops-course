# Installation and Configuration

https://abhishekveeramalla-av.medium.com/run-argo-cd-using-operator-on-kind-e59f48687d38

## Run Argo CD using operator on Kind

This post will document how to run an Argo CD instance using Argo CD operator.

We will use Kind to create the Kubernetes cluster and Nginx as ingress controller.
Kubernetes Cluster

We can leverage Kind’s extraPortMapping config option when creating a cluster to forward ports from the host to an ingress controller running on a node.
Create a kind cluster with extraPortMappings and node-labels.

```
    extraPortMappings allow the local host to make requests to the Ingress controller over ports 80/443
    node-labels only allow the ingress controller to run on a specific node(s) matching the label selector
```

`$ kubectl create -f https://raw.githubusercontent.com/iam-veeramalla/argocd-clusterconfig-demo/main/kind-argocd-ingress.yaml`

## Install Nginx Ingress Controller

`$ kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/main/deploy/static/provider/kind/deploy.yaml`

The manifests contains kind specific patches to forward the hostPorts to the ingress controller, set taint toleration and schedule it to the custom labelled node.

Now the Ingress is all setup. Wait until is ready to process requests running:

```
$ kubectl wait --namespace ingress-nginx \
  --for=condition=ready pod \
  --selector=app.kubernetes.io/component=controller \
  --timeout=90s
```

## Install Argo CD Operator

Install Operator Lifecycle Manager (OLM), a tool to help manage the Operators running on your cluster.

`$ curl -sL https://github.com/operator-framework/operator-lifecycle-manager/releases/download/v0.20.0/install.sh | bash -s v0.20.0`

or

`$ operator-sdk olm install`

2. Create subscription resource for Argo CD operator using the below command.

`$ kubectl create -f https://operatorhub.io/install/argocd-operator.yaml`

3. After install, watch your operator come up using next command.

`$ kubectl get csv -n operators`

## Create Argo CD Instance

We are done with the creation of a Kubernetes cluster and installing Nginx ingress controller and Argo CD operator. The next step is to create an Argo CD instance in the cluster.

`$ kubectl apply -f https://raw.githubusercontent.com/argoproj-labs/argocd-operator/master/examples/argocd-ingress.yaml`

Watch for the Argo CD workloads to start.

`$ kubectl get pods -w`

## Login to Argo CD

Once the pods are up and running, you can access the Argo CD server using the below URL on your browser.

`https://example-argocd`

**Note: ** The Argo CD admin password is stored in the secret with name<instance-name>-cluster.

`$ oc extract secret/example-argocd-cluster — to=-`

Thanks for reading :). I hope you find this article useful. Go Argo CD !!!

**References:**

 Argo CD Operator: https://github.com/argoproj-labs/argocd-operator
