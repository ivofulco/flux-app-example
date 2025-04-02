- [GitOps Simplificado com Flux](#gitops-simplificado-com-flux)
- [Introdução](#introdução)
- [O que é GitOps?](#o-que-é-gitops)
- [Preparação do Ambiente](#preparação-do-ambiente)
  - [Instalando flux cli](#instalando-flux-cli)
- [Passo a Passo: Configurando a Implantação Contínua](#passo-a-passo-configurando-a-implantação-contínua)
  - [Configurando o Repositório Git e a Aplicação de Exemplo](#configurando-o-repositório-git-e-a-aplicação-de-exemplo)
  - [Criando os Manifests Kubernetes da Aplicação](#criando-os-manifests-kubernetes-da-aplicação)
  - [Instalando o Flux no Cluster Kubernetes](#instalando-o-flux-no-cluster-kubernetes)
  - [Configurando Acesso do Flux ao Repositório Git](#configurando-acesso-do-flux-ao-repositório-git)
  - [Deploy Automático com Flux](#deploy-automático-com-flux)
  - [Verificando o Deploy no Cluster](#verificando-o-deploy-no-cluster)
- [Conclusão](#conclusão)
- [Fonte](#fonte)

# GitOps Simplificado com Flux

Pipeline Completo de Implantação Contínua em Kubernetes

# Introdução
Este guia prático demonstrará como configurar uma pipeline de implantação contínua completa com GitOps usando o Flux em Kubernetes. Vamos abordar desde a configuração inicial do Git, criação de uma aplicação de exemplo, até a configuração e o deploy automático. Ao final, seu ambiente estará pronto para sincronizar atualizações do código no Git com seu cluster Kubernetes, garantindo implantações rápidas e confiáveis.

# O que é GitOps?
GitOps é uma abordagem que usa repositórios Git como a “fonte única de verdade” para as operações e configuração de infraestrutura. Com ele, a automação de deploys é feita de forma segura, consistente e rastreável, ideal para ambientes Kubernetes que requerem escalabilidade e facilidade de gerenciamento.

# Preparação do Ambiente
Para seguir este tutorial, você precisará de:

- Um cluster Kubernetes
- kubectl
- Helm
- Um repositório Git, onde armazenaremos o código e as configurações da aplicação.
  

## Instalando flux cli

Para instalar o flux cli basta seguir este [passo a passo](https://fluxcd.io/flux/cmd/) de acordo com seu SO.

# Passo a Passo: Configurando a Implantação Contínua

## Configurando o Repositório Git e a Aplicação de Exemplo

Crie um novo repositório Git no GitHub (ou outro serviço de hospedagem Git).

```
flux-app-example/
├── deploy/
│   ├── deployment.yaml
│   └── service.yaml
└── Dockerfile
```

Clone o repositório para sua máquina local:

```shell
git clone https://github.com/ivofulco/fluxcd.git 
cd fluxcd
```

Adicione uma aplicação de exemplo. Crie uma estrutura básica para uma aplicação Node.js ou NGINX. Aqui vamos usar o NGINX para simplificar.

No arquivo Dockerfile, adicione o seguinte conteúdo para criar uma imagem NGINX personalizada:

```Dockerfile
FROM nginx:alpine 
COPY index.html /usr/share/nginx/html/index.html
```

Crie um arquivo index.html com um conteúdo simples:

```HTML5
<!DOCTYPE html>
<html>
<head>
    <title>GitOps com Flux</title>
</head>
<body>
    <h1>Deploy Contínuo com Flux e GitOps</h1>
    <p>Aplicação NGINX de exemplo.</p>
</body>
</html>
```


## Criando os Manifests Kubernetes da Aplicação

Dentro do diretório deploy/, crie os manifests de Kubernetes para o Deployment e o Service:

- deployment.yaml
- service.yaml
  
Faça o commit e o push dessas mudanças para o repositório:

```shell
git add .
git commit -m "Adiciona aplicação de exemplo e manifests Kubernetes"
git push origin main
```

## Instalando o Flux no Cluster Kubernetes
Agora, vamos instalar o Flux para que ele possa monitorar o repositório Git e sincronizar as alterações com o cluster.

Adicione o repositório Helm do Flux e atualize:

```shell
helm repo add fluxcd https://charts.fluxcd.io
helm repo update
```

Instale o Flux no namespace flux:

```shell
helm install flux fluxcd/flux --namespace flux --create-namespace
```

Verifique se o Flux foi instalado corretamente:

```shell
kubectl get pods -n flux
```

## Configurando Acesso do Flux ao Repositório Git
Para que o Flux monitore seu repositório, é necessário configurá-lo com uma chave SSH que permita o acesso ao repositório Git.

Gere um par de chaves SSH:

```shell
ssh-keygen -t rsa -b 4096 -C "ivofulco@outlook.com" -f flux-key -N ""
```

Adicione a chave pública (flux-key.pub) nas configurações de deploy keys do repositório Git.

Crie um segredo Kubernetes com a chave privada para o Flux:

```shell
kubectl create secret generic flux-git-key --from-file=identity=flux-key --namespace flux
```

Atualize o Flux para usar o segredo criado, configurando o Git repository e branch no namespace flux:


```yaml
apiVersion: fluxcd.io/v1
kind: HelmRelease
metadata:
  name: flux
  namespace: flux
spec:
  git:
    url: git@github.com:ivofulco/flux-app-example.git
    branch: main
    secretRef:
      name: flux-git-key
```

## Deploy Automático com Flux
Com o Flux configurado, ele começará a monitorar o repositório Git. Assim que qualquer alteração for detectada nos manifests de Kubernetes (dentro da pasta deploy/), o Flux aplicará automaticamente as mudanças ao cluster.

Faça uma alteração no index.html ou nos arquivos do Kubernetes para simular um ajuste de configuração.

Faça o commit e o push das alterações:

```shell
git add .
git commit -m "Atualiza conteúdo da aplicação de exemplo"
git push origin main
```

O Flux detectará essa mudança e atualizará o cluster de acordo com os manifests mais recentes.

## Verificando o Deploy no Cluster
Verifique se o Deployment e o Service foram aplicados corretamente:

```shell```
kubectl get deployments kubectl get services
```

Localize o IP externo do serviço nginx-service (em um cluster real ou no Minikube com o minikube service nginx-service --url) e abra no navegador para ver a aplicação de exemplo rodando.

# Conclusão
Neste tutorial, configuramos um pipeline completo de implantação contínua com GitOps usando Flux e Kubernetes. Com essa configuração, qualquer alteração no repositório Git é automaticamente sincronizada com o cluster, facilitando o controle de versões e a rastreabilidade das alterações.

# Fonte

[Artigo original neste link](https://medium.com/@habbema/gitops-simplificado-com-flux-d5b2916d8078)