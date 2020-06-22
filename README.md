# rcc-portals-k8 deployments

This README file has step-by-step guides to bringup and set up RCC portals in kubernetes running in NeCTAR.


## Bring up Kubernetes in NeCTAR
Follow the NeCTAR tutorial on how to bring up Kubernetes cluster: https://tutorials.rc.nectar.org.au/kubernetes/01-overview

## Ingeress and certificates
The Nectar tutorial gives an example using nginx via port 80. Here, I am using cert-manager with nginx ingress and letsencrypt to create a https load balancer. 


I am using helm (version 3) for this

### install cert-manager

Follow some steps from https://cert-manager.io/docs/installation/kubernetes/


create cert-manaer namespace

    kubectl create namespace cert-manager

Add the Jetstack Helm repository.

    helm repo add jetstack https://charts.jetstack.io

Update your local Helm chart repository cache.

    helm repo update

Install

    helm install cert-manager jetstack/cert-manager --namespace cert-manager --version v0.14.3

verify

    kubectl get pods --namespace cert-manager

Install nginx-ingress

    helm install nginx stable/nginx-ingress --set rbac.create=true --set controller.publishService.enabled=true

Then create security group with http, https open 


install custom resource definition (kub version>=1.15)

    kubectl apply --validate=false -f https://github.com/jetstack/cert-manager/releases/download/v0.15.1/cert-manager.crds.yaml

this is important, otherwise step 6 throws errors like this:

    unable to recognize "issuer.yaml": no matches for kind "ClusterIssuer" in version "cert-manager.io/v1alpha2"
    unable to recognize "issuer.yaml": no matches for kind "ClusterIssuer" in version "cert-manager.io/v1alpha2"
    unable to recognize "issuer.yaml": no matches for kind "ClusterIssuer" in version "cert-manager.io/v1alpha2"


for kubernetest < 1.15

    kubectl apply --validate=false -f https://github.com/jetstack/cert-manager/releases/download/v0.15.1/cert-manager-legacy.yaml

    kubectl delete mutatingwebhookconfiguration.admissionregistration.k8s.io cert-manager-webhook
    kubectl delete validatingwebhookconfigurations.admissionregistration.k8s.io cert-manager-webhook


follow this to verify: https://cert-manager.io/docs/installation/kubernetes/#verifying-the-installation

    kubectl apply -f cert-manager/test-resources.yaml
    kubectl describe certificate -n cert-manager-test
    kubectl delete -f cert-manager/test-resources.yaml

create issuer
    
    kubectl apply -f cert-manager/issuer.yaml


### Create a test site
    
    First you need to make sure create a DNS name to the IP address of the load balacer. For instance, I have k8test.rcc.uq.edu.au points to IP address of the load balancer.
 

Create test app and service

    kubectl apply -f test-service/testk8.yaml

Create the ingress, this will issue a new cert as well
    
    kubectl apply -f test-service/testk8-ingress.yaml

It will take sometimes till the new cert is created

    kubectl get certs