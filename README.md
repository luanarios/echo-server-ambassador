Criação do cluster:

k3d cluster create bookinfo --k3s-arg="--disable=traefik@server:0" --image=rancher/k3s:v1.21.14-k3s1

Instalação via Helm

helm install ambassador datawire/ambassador --namespace=ambassador --values values.yaml && \
kubectl wait --for condition=available --timeout=90s deploy -lproduct=aes


---

Versão 3.2.0 

k3d cluster create bookinfo --k3s-arg="--disable=traefik@server:0"

# Add the Repo:
helm repo add datawire https://app.getambassador.io
helm repo update
 
# Create Namespace and Install:
kubectl create namespace emissary && \
kubectl apply -f https://app.getambassador.io/yaml/emissary/3.2.0/emissary-crds.yaml
kubectl wait --timeout=90s --for=condition=available deployment emissary-apiext -n emissary-system
helm install emissary-ingress --namespace emissary datawire/emissary-ingress && \
kubectl -n emissary wait --for condition=available --timeout=90s deploy -lapp.kubernetes.io/instance=emissary-ingress