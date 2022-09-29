# apolo11-go-api-k8s

This will be the plan for this project:

![workflow](./images/apollo11-gitops.png)


# Setup steps:

- Install K8s

```
# Install k0s on the system
curl -sSLf https://get.k0s.sh | sudo sh
sudo k0s install controller --single
sudo k0s start
sudo k0s status

# Check status of nodes

sudo k0s kubectl get nodes
## wait till the nodes are in ready state	
## Here we used the kubectl which comes bundled with k0s, but its better to use the official kubectl client 
```

- Copy the kubeconfig to local user

```
cd ~
## copy the config from k0s admin conf and add in your users .kube folder. 
## note: the user name and group name can be same based on your machine
mkdir .kube
sudo cp /var/lib/k0s/pki/admin.conf .kube/config && sudo chmod 600 .kube/config && sudo chown <user_name>:<group_name> .kube/config

```
- Install native kubectl 

```
# More instuctions here: https://kubernetes.io/docs/tasks/tools/install-kubectl-linux/

curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
curl -LO "https://dl.k8s.io/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl.sha256"
echo "$(cat kubectl.sha256)  kubectl" | sha256sum --check ## Verify that the output is `kubectl: OK`
sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl

# Check if you can access the cluster using native kubectl
kubectl get nodes
# you should be able to see t

```

- Test by creating a pod:

```
kubectl run nginx --image=nginx
kubeclt get pods
kubectl port-forward nginx 1200:80  # because in some servers ports below 1024 may be blocked due to privilege port restrictions (below 1024)
## in a new terminal
curl localhost:1200 
## you should be able to see the  nginx page html in the output
## once confirmed, clean up the above resources
## Crtl+c the port forward and then delete the pod
kubectl delete pod nginx
```

- Install Traefik

```
## Install helm
curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3
chmod 700 get_helm.sh
./get_helm.sh
helm version # confirms that helm is installed

## Now lets install traefik
## This will isntall a replicaset(and hence a pod) for the traefik ingress controller in default namespace

helm repo add traefik https://helm.traefik.io/traefik
helm repo update
helm install traefik traefik/traefik

## Confirm that the required components are created
helm status traefik
kubectl get all -A # confirm the required traefik resources are created

## port forward to visit the traefik dashboard

## if running from localhost
kubectl port-forward $(kubectl get pods --selector "app.kubernetes.io/name=traefik" --output=name) 9000:9000
OR
## if running on a server, make sure that the server firewall allows connection from your machine
kubectl port-forward $(kubectl get pods --selector "app.kubernetes.io/name=traefik" --output=name) --address 0.0.0.0 9000:9000

## Access the traefik dashboard

## NOTE: Ensure that there is a `/` at the end of the url mentioned below, else the dashboard wont load


## if running from localhost
Go to your browser : http://localhost:9000/dashboard/

OR
## if running on a server, make sure that the server firewall allows connection from your machine

Go to your browser : http://<server_ip>:9000/dashboard/

```

- Create a ingress route in Traefik

```
kubectl apply -f manifests/traefik/ingressroutes/nginx.yaml
# this will create a test nginx pod and expose it as a clusterip service
# Then it will create a traefik ingressroute to this nginx service

# in a new terminal or tmux, open a port-forward towards the traefik web endpoint 8000
## if running from localhost
kubectl port-forward $(kubectl get pods --selector "app.kubernetes.io/name=traefik" --output=name) 8000:8000
OR
## if running on a server, make sure that the server firewall allows connection from your machine
kubectl port-forward $(kubectl get pods --selector "app.kubernetes.io/name=traefik" --output=name) --address 0.0.0.0 8000:8000


## if running from localhost
Go to your browser : http://localhost:8000/nginx

OR
## if running on a server, make sure that the server firewall allows connection from your machine

Go to your browser : http://<server_ip>:8000/nginx

```
