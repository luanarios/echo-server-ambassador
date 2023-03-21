# Disclaimer

Esse é um repositório para realização dos testes da implementação do Rate Limit disponível em [envoyproxy/ratelimit](https://github.com/envoyproxy/ratelimit). Nessa stack vamos utilizar o [Emissary Ingress](https://www.getambassador.io/docs/emissary) como serviço de API-Gateway e a aplicação [Bookinfo](https://istio.io/v1.0/docs/examples/bookinfo/) que é composta por quatro microserviços e será utilizada para demonstrar o funcionamento das features do Emissary Ingress e da aplicação de Rate Limit do Envoy.

# Preparando o ambiente:

## Criação do Cluster

Para criação do cluster local, vamos utilizar a ferramenta [k3d](https://k3d.io/v5.4.9/).

Executamos o comando abaixo para desabilitar o traefik (pois pode causar conflito com a instalação do Emissary Ingress) e utilizar a versão 1.24.9 do Kubernetes.

`k3d cluster create master-rl --k3s-arg="--disable=traefik@server:0" --image=rancher/k3s:v1.24.9-k3s2`

## Adicionando o Helm

Adicionando e atualizando o repositório:

`helm repo add datawire https://app.getambassador.io
helm repo update`

## Instalação do Emissary Ingress

Os comandos abaixo criarão o namespace para o emissary, configuração dos CRDs e instalação.

```kubectl create namespace emissary && \
kubectl apply -f https://app.getambassador.io/yaml/emissary/3.2.0/emissary-crds.yaml
kubectl wait --timeout=90s --for=condition=available deployment emissary-apiext -n emissary-system
```
```
helm install emissary-ingress --namespace emissary datawire/emissary-ingress && \
kubectl -n emissary wait --for condition=available --timeout=90s deploy -lapp.kubernetes.io/instance=emissary-ingress
```

## Instalação do Redis

Adicionando o repositório e realizando a instalação de uma instância do Redis.

helm repo add bitnami https://charts.bitnami.com/bitnami
helm install redis bitnami/redis --values values/redis-values.yaml

# Deploy da Aplicação

## Bookinfo

Realize o deploy da stack do Bookinfo:

`kubectl apply -f bookinfo/bookinfo.yaml`

## Emissary Ingress

Configure o `Host`, `Listener` e `Mappings` do Emissary Ingress para acessar a aplicação do Bookinfo:

`kubectl apply -f ambassador/`

## Acessando o Bookinfo

Verifique o IP do LoadBalancer do Emissary Ingress com o comando:

`kubectl get svc -n emissary`

Nessa stack mapeamos o ``EXTERNAL-IP`` para o host **test.bookinfo.com**.

`sudo nano /etc/hosts`

Acesse o endereço no seu browser e verifique a stack do Bookinfo funcionando.

## Rate Limit

Faça o deploy do rate limit:

`kubectl apply -f ratelimit/`

# Testando o Rate Limit

## Configuração

O Rate Limit deve ser configurado de acordo com a [documentação do Emissary Ingress](https://www.getambassador.io/docs/emissary/latest/topics/using/rate-limits). Dentro do Mapping inserimos o label specifier da configuração que criamos no 
`ratelimit/configmap.yaml`. No exemplo dessa stack, passamos um `request_headers` e limitamos o serviço productpage para 5 requisições por minuto.
 
 ## Validando o Rate Limit

 Realizando a solicitação abaixo, é esperado receber 5 respostas a cada um minuto.

 `curl -H "teste: foo" test.bookinfo.com/`

 Verificando os logs do pod do serviço de ratelimit é possível verificar as quantidade de solicitações atuais e as que estão sendo barradas.

 `kubectl get pods -A`

 `kubectl get logs -f <nome-do-pod>`
