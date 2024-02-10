# Tekton-Tutorial
---------------------------------------------------------------------------------------------------------------------------
Talking about the architect Tekton
---------------------------------------------------------------------------------------------------------------------------
Prepair the environment
---------------------------------------------------------------------------------------------------------------------------
https://kind.sigs.k8s.io/docs/user/quick-start/
---------------------------------------------------------------------------------------------------------------------------
Install Kind
# For AMD64 / x86_64
[ $(uname -m) = x86_64 ] && curl -Lo ./kind https://kind.sigs.k8s.io/dl/v0.20.0/kind-linux-amd64
chmod +x ./kind
sudo mv ./kind /usr/local/bin/kind
---------------------------------------------------------------------------------------------------------------------------
Implement the Kubernetes Cluster
cat <<EOF | kind create cluster --config=-
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
networking:
  apiServerAddress: "0.0.0.0"
nodes:
- role: control-plane
  kubeadmConfigPatches:
  - |
    kind: InitConfiguration
    nodeRegistration:
      kubeletExtraArgs:
        node-labels: "ingress-ready=true"
  extraPortMappings:
  - containerPort: 80
    hostPort: 80
    protocol: TCP
  - containerPort: 443
    hostPort: 443
    protocol: TCP
  - containerPort: 6443
    hostPort: 6443
    protocol: TCP

EOF
---------------------------------------------------------------------------------------------------------------------------
kubectl cluster-info --context kind-kind
---------------------------------------------------------------------------------------------------------------------------
Implement Ingress nginx 
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/main/deploy/static/provider/kind/deploy.yaml
---------------------------------------------------------------------------------------------------------------------------
---------------------------------------------------------------------------------------------------------------------------
Apply sample application configuration
kubectl apply -f https://kind.sigs.k8s.io/examples/ingress/usage.yaml
---------------------------------------------------------------------------------------------------------------------------
Test ingress configuration
# should output "foo-app"
curl localhost/foo/hostname
# should output "bar-app"
curl localhost/bar/hostname
---------------------------------------------------------------------------------------------------------------------------
Install Tekton
k apply -f ./tekton-v0.55.0/
kubectl get pods --namespace tekton-pipelines --watch
---------------------------------------------------------------------------------------------------------------------------
Dashboard Link
http://dashboard.my.ir

---------------------------------------------------------------------------------------------------------------------------
I will talk about the Task and Pipeline 
![alt text](https://github.com/iraj-norouzi/Tekton-Tutorial/blob/main/pictures/2.png?raw=true)
![alt text](https://github.com/iraj-norouzi/Tekton-Tutorial/blob/main/pictures/1.png?raw=true)
![alt text](https://github.com/iraj-norouzi/Tekton-Tutorial/blob/main/pictures/3.png?raw=true)
![alt text](https://github.com/iraj-norouzi/Tekton-Tutorial/blob/main/pictures/4.png?raw=true)
![alt text](https://github.com/iraj-norouzi/Tekton-Tutorial/blob/main/pictures/5.png?raw=true)

---------------------------------------------------------------------------------------------------------------------------
---------------------------------------------------------------------------------------------------------------------------
