# k8s-tls-lab

kubectl create clusterrolebinding cluster-admin-binding --clusterrole cluster-admin --user kubernetes-admin
helm repo add nginx-stable https://helm.nginx.com/stable
helm repo update
kubectl get nodes -o json | jq .items[0].status.addresses[0].address
MYSTATICIP=$(kubectl get nodes -o json | jq .items[0].status.addresses[0].address)
echo $MYSTATICIP
helm install nginx-stable/nginx-ingress --set controller.service.loadBalancerIP="$MYSTATICIP",rbac.create=true --generate-name
kubectl apply -f k8s-tls-lab/configmap.yaml
kubectl apply -f k8s-tls-lab/deployment.yaml
kubectl create -f k8s-tls-lab/ingress.yaml
