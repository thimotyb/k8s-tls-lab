# k8s-tls-lab

## Create endpoint for lets encrypt to perform challenge from outside

gcloud compute addresses create endpoints-ip --region us-central1

gcloud compute addresses list

## Expose public API as public endpoint reacheble from lets encypt (Edit openapi.yamls with static IP assigned at step before)

gcloud endpoints services deploy openapi.yaml

## Create k8s cluster and setup RBAC

gcloud container clusters create cl-cluster --zone us-central1-f

kubectl create clusterrolebinding cluster-admin-binding --clusterrole cluster-admin --user $(gcloud config get-value account)
OR:
kubectl create clusterrolebinding cluster-admin-binding --clusterrole cluster-admin --user kubernetes-admin

## Add nginx ingress via HELM

helm repo add nginx-stable https://helm.nginx.com/stable

helm repo update

--kubectl get nodes -o json | jq .items[0].status.addresses[0].address
--MYSTATICIP=$(kubectl get nodes -o json | jq .items[0].status.addresses[0].address)
--echo $MYSTATICIP

## Populate STATIC IP External and start ingress with that IP

helm install nginx-stable/nginx-ingress --set controller.service.loadBalancerIP="$MYSTATICIP",rbac.create=true --generate-name

kubectl apply -f k8s-tls-lab/configmap.yaml

kubectl apply -f k8s-tls-lab/deployment.yaml

kubectl apply -f k8s-tls-lab/service.yaml

## Edit ingress.yaml with project ID
kubectl create -f k8s-tls-lab/ingress.yaml

## Test with browser that domain/IP is reachable from internet
http://api.endpoints.[MY-PROJECT].cloud.goog

# Setup HTTPS
## Setup Let's Encrypt / Cert Manager
kubectl apply -f https://github.com/cert-manager/cert-manager/releases/download/v1.8.0/cert-manager.crds.yaml

kubectl create namespace cert-manager

helm repo add jetstack https://charts.jetstack.io

helm repo update

helm install cert-manager jetstack/cert-manager --namespace cert-manager  --create-namespace --version v1.8.0

kubectl get pods --namespace cert-manager

export EMAIL=<YOUR_EMAIL>

cat letsencrypt-issuer.yaml | sed -e "s/email: ''/email: $EMAIL/g" | kubectl apply -f-

## Navigate to gke-tls-lab/certificate.yaml. and Replace all instances of [MY-PROJECT] with Project ID.

## Navigate to ingress-tls.yaml. Rename all instances of [MY-PROJECT] with your Project ID. These are adding annotations for TLS with cert manager




