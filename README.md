# gitopscon_na_2022_session

### Session: https://sched.co/1AR9K


Scenarios:

  Setup and prerequisites:
  Define Kyverno and the Default PolicySets as add-ons


  #### Clusters-as-a-Service

  1. A Team requests a cluster 
    a. Create a PR with a namespace and the right label
    b. This triggers Kyverno to generate CAPI resources in the Namespace
    c. This triggers CAPI to provision the cluster
  2. ArgoCD creates a CAPI based cluster 
  3. Kyverno can restrict configurations and counts
  4. The new cluster must be registered back to ArgoCD
    a. Can Kyverno help register the new cluster? https://github.com/kyverno/policies/pull/359/files 
  5. Kyverno updates the ApplicationSet for each shared service / add-on including Kyverno and the default PolicySets 
  6. ArgoCD installs Kyverno in new cluster
  7. ArgoCD installs standard Kyverno policies in the new cluster
  8. Kyverno generates roles, etc. in the management cluster to prevent other teams from accessing it
  9. User creates Namespace on the managed/tenant cluster
  10. Kyverno (in the managed/tenant cluster) generates Namespace role-bindings and network policies 


## Demo

1. Create management cluster

```sh
kind create cluster --config kind-mgmt-cluster.yaml --name mgmt
```

2. Initialize CAPI in the management cluster

```sh
kind export kubeconfig --name mgmt
export CLUSTER_TOPOLOGY=true
clusterctl init --infrastructure docker
```

3. Install Kyverno

```sh
kubectl create -f https://raw.githubusercontent.com/kyverno/kyverno/main/config/install.yaml
```

4. Install policies

```sh
kubectl apply -f policies/
```

NOTE: Currently, we need to copy the ClusterClass and all related template object to the target namespace. See: https://github.com/kubernetes-sigs/cluster-api/issues/5673 which will allow using a ClusterClass across namespaces.

5. Create a CAPI cluster by creating a new namespace

```sh
kubectl create ns t1
```

6. Install a CNI (this will be automated)

```sh
kind export kubeconfig --name t1
kubectl apply -f https://raw.githubusercontent.com/projectcalico/calico/v3.24.1/manifests/calico.yaml
```

7. Check tenant cluster nodes

```sh
kubectl get nodes
```

8. Check the cluster status

```sh
kubectl config use kind-mgmt
clusterctl describe cluster t1 -n t1
```
