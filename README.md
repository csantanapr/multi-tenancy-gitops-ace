# Cloud Native Toolkit Deployment Guides


## Apply demo sealedsecret key to all clusters
Download [sealed-secrets-ibm-demo-key.yaml](https://bit.ly/demo-sealed-master) and apply it to the cluster.
```
oc new-project sealed-secrets

oc apply -f ~/Downloads/sealed-secrets-ibm-demo-key.yaml

```
# DO NOT CHECK INTO GIT.
```
rm ~/Downloads/sealed-secrets-ibm-demo-key.yaml
```

## Install OpenShfit GitOps (ArgoCD)
To get started setup gitops operator and rbac on each cluster

- For OpenShift 4.7+ use the following:
```
oc apply -f setup/ocp47/
while ! kubectl wait --for=condition=Established crd applications.argoproj.io; do sleep 30; done
oc apply -n openshift-operators -f https://raw.githubusercontent.com/cloud-native-toolkit/multi-tenancy-gitops-services/master/operators/openshift-pipelines/operator.yaml
while ! oc extract secrets/openshift-gitops-cluster --keys=admin.password -n openshift-gitops --to=- ; do sleep 30; done
```

- For OpenShift 4.6 use the following:
```
oc apply -f setup/ocp46/
while ! kubectl wait --for=condition=Established crd applications.argoproj.io; do sleep 30; done
while ! oc extract secrets/argocd-cluster-cluster --keys=admin.password -n openshift-gitops --to=- ; do sleep 30; done
```

The `admin` and password should have printed with the previous command


## Install the ArgoCD Application Bootstrap for Single Cluster Profile
Apply the bootstrap profile, to use the default `single-cluster` scenario use the following command:
```
oc apply -n openshift-gitops -f 0-bootstrap/argocd/bootstrap.yaml
```


## Other Cluster profiles
For other profile clusters set environment variable `TARGET_CLUSTER` then apply the profile

**shared-cluster**:
```
TARGET_CLUSTER=0-bootstrap/argocd/others/1-shared-cluster/bootstrap-cluster-1-cicd-dev-stage-prod.yaml

TARGET_CLUSTER=0-bootstrap/argocd/others/1-shared-cluster/bootstrap-cluster-n-prod.yaml
```
Now apply the profile
```
echo TARGET_CLUSTER=${TARGET_CLUSTER}
oc apply -n openshift-gitops -f ${TARGET_CLUSTER}
```


This repository shows the reference architecture for gitops directory structure for more info https://cloudnativetoolkit.dev/learning/gitops-int/gitops-with-cloud-native-toolkit

