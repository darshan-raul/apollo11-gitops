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
```


