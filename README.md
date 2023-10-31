# DataHub Accelerator

This is an accelerator that can be used to generate a Kubernetes deployment for [DataHub](https://datahubproject.io/).

* Install App Accelerator: (see https://docs.vmware.com/en/Tanzu-Application-Platform/1.0/tap/GUID-cert-mgr-contour-fcd-install-cert-mgr.html)
```
tanzu package available list accelerator.apps.tanzu.vmware.com --namespace tap-install
tanzu package install accelerator -p accelerator.apps.tanzu.vmware.com -v 1.0.1 -n tap-install -f resources/app-accelerator-values.yaml
Verify that package is running: tanzu package installed get accelerator -n tap-install
Get the IP address for the App Accelerator API: kubectl get service -n accelerator-system
```

Publish Accelerators:
```
tanzu plugin install --local <path-to-tanzu-cli> all
tanzu acc create argo-pipelines-kapp-acc --git-repository https://github.com/agapebondservant/sample-argo-workflow-kappcontroller-accelerator.git --git-branch main
```

## Contents
1. [Pre-requisites](#prereq)
2. [Deploy Argo Training Pipeline](#deploy)

### Deploy Pre-requisites <a name="prereq">
* Set up the following pre-requisites on your Kubernetes cluster:
[ x ] Argo Workflows

* Set up permissions:
```
kubectl apply -f resources/tap-rbac.yaml
kubectl apply -f resources/tap-rbac-2.yaml
```

### Deploy Argo Training Pipeline on TAP <a name="deploy">
* cd to </root/of/branch/directory/with/appropriate/model/stage>
  (Example: the **main** github branch represents the "main" environment, the **staging** github branch represents the "staging" environment, etc)

* Deploy the pipeline:
```
ytt -f config/cifar/pipeline_app.yaml -f config/cifar/values.yaml | kapp deploy -a image-procesor-pipeline-<THE PIPELINE ENVIRONMENT> --logs -y  -nargo -f -
```

* View progress:
```
kubectl get app ml-image-processing-pipeline-<THE PIPELINE ENVIRONMENT> -oyaml  -nargo
```

* View the pipeline in the browser by navigating to https://argo-workflows.<your-domain-name> -
  access the Login token by running
```
kubectl -n argo exec $(kubectl get pod -n argo -l 'app=argo-server' -o jsonpath='{.items[0].metadata.name}') -- argo auth token
```

* To delete the pipeline:
```
kapp delete -a image-procesor-pipeline-<THE PIPELINE ENVIRONMENT> -y -nargo
```
