# gitopscon_na_2022_session

### Session: https://sched.co/1AR9K

Scenarios:

  Setup and prerequisites:
  Define Kyverno and the Default PolicySets as add-ons


  ####Clusters-as-a-Service

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
