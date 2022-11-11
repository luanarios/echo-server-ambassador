Criação do cluster:

k3d cluster create bookinfo --k3s-arg="--disable=traefik@server:0" --image=rancher/k3s:v1.21.14-k3s1

Instalação via Helm

helm install ambassador datawire/ambassador --namespace=ambassador --values values.yaml && \
kubectl wait --for condition=available --timeout=90s deploy -lproduct=aes