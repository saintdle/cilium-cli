kind create cluster --config kind-config-1.yaml --name cluster1
kind create cluster --config kind-config-2.yaml --name cluster2

kubectl --context kind-cluster1 apply -f https://github.com/cert-manager/cert-manager/releases/download/v1.11.0/cert-manager.crds.yaml
kubectl --context kind-cluster2 apply -f https://github.com/cert-manager/cert-manager/releases/download/v1.11.0/cert-manager.crds.yaml

kubectl apply --context kind-cluster1 -f issuer.yaml
kubectl apply --context kind-cluster2 -f issuer.yaml

kubectl --context kind-cluster1 -n kube-system create secret tls cilium-ca --cert=cilium-ca-crt.pem --key=cilium-ca-key.pem
kubectl --context kind-cluster2 -n kube-system create secret tls cilium-ca --cert=cilium-ca-crt.pem --key=cilium-ca-key.pem

./cilium install --context kind-cluster1 --chart-directory ~/go/src/github.com/cilium/cilium/install/kubernetes/cilium --helm-set cluster.id=1 --helm-values ./myval.yaml
./cilium install --context kind-cluster2 --chart-directory ~/go/src/github.com/cilium/cilium/install/kubernetes/cilium --helm-set cluster.id=2 --helm-values ./myval.yaml

kubectl apply --context kind-cluster1 -f https://github.com/cert-manager/cert-manager/releases/download/v1.11.0/cert-manager.yaml
kubectl apply --context kind-cluster2 -f https://github.com/cert-manager/cert-manager/releases/download/v1.11.0/cert-manager.yaml

./cilium --context kind-cluster1 clustermesh status --wait
./cilium --context kind-cluster2 clustermesh status --wait
./cilium --context kind-cluster1 clustermesh enable
./cilium --context kind-cluster2 clustermesh enable
kubectl --context kind-cluster1 get service -n kube-system clustermesh-apiserver
kubectl --context kind-cluster2 get service -n kube-system clustermesh-apiserver
./cilium clustermesh connect --context kind-cluster1 --destination-context kind-cluster2
./cilium uninstall --context kind-cluster1
./cilium uninstall --context kind-cluster2
kind delete cluster -n cluster1
kind delete cluster -n cluster2
