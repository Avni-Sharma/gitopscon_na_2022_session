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

3. Install and configure ArgoCD

```sh
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```

```sh
kubectl patch svc argocd-server -n argocd -p '{"spec": {"type": "LoadBalancer"}}'
```

```sh
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d; echo
```

```sh
kubectl port-forward svc/argocd-server -n argocd 8080:443
```

```sh
 argocd login 127.0.0.1:8080 --username admin --password <SECRET>
```

Navigate to: https://127.0.0.1:8080/

Login through argocd cli

```sh
argocd login localhost:8080 --username admin --password <SECRET>
```

4. Install AppSets

```sh
kubectl create -f appsets
```

```sh
kubectl get appsets
```


5. Install Kyverno

```sh
kubectl create -f https://raw.githubusercontent.com/kyverno/kyverno/main/config/install.yaml
```

6. Install policies

```sh
kubectl apply -f policies/
```

NOTE: Currently, we use policies to copy the ClusterClass and all related template object to the target namespace. See: https://github.com/kubernetes-sigs/cluster-api/issues/5673 which will allow using a ClusterClass across namespaces.

7. Create a CAPI cluster by creating a new namespace

```sh
kubectl create ns cluster1
```

8. Check for the tenant cluster to be created:

```sh
clusterctl describe cluster cluster1 -n cluster1
```

The output should match:

```sh
NAME                                                    READY  SEVERITY  REASON  SINCE  MESSAGE 
Cluster/cluster1                                        True                     6h34m           
├─ClusterInfrastructure - DockerCluster/cluster1-tdtcd  True                     6h35m           
├─ControlPlane - KubeadmControlPlane/cluster1-td584     True                     6h34m           
│ └─Machine/cluster1-td584-lpn4t                        True                     6h34m           
└─Workers                                                                                        
  └─MachineDeployment/cluster1-md-0-s9mv8               True                     6h18m           
    └─Machine/cluster1-md-0-s9mv8-d69fc6bb9-hgp5p       True                     6h34m           

New clusterctl version available: v1.2.2 -> v1.2.3
https://github.com/kubernetes-sigs/cluster-api/releases/tag/v1.2.3                                                        
```

9. Check the ArgoCD UI to verify that the three applications (Calico, Kyverno, and Kyverno policies) are deployed. You may need to refresh to trigger the deploy.

10. Switch to the tenant cluster and try to create a pod:

```sh
kind export kubeconfig --name cluster1
```

```sh
kubectl run test --image=nginx
```

You should see an error stating that the image is not signed.

11. Try running a signed image:

```sh
kubectl run test --image ghcr.io/kyverno/test-verify-image:signed
```

This will now show other policy violation errors.

12. Try running a signed image with proper configuration:

```sh
kubectl create ns test
kubectl apply -n test -f resources/good-pod.yaml
```


## Cleanup

```sh
kind export kubeconfig --name mgmt
kubectl delete cluster cluster1 -n cluster1
kubectl delete ns cluster1
kubectl delete secret cluster1 -n argocd
kubectl delete ur --all -n kyverno
```

