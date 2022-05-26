# ARGOCD DEPLOY ACTION

Action de deploy das aplicações com o  ArgoCD.

## Inputs
Todos os inputs são de definição obrigatória.

- **aws-access-key-id** → secret key da AWS 
- **aws-secret-access-key** → secret da AWS
- **owner** → organização no github
- **environment** → ambiente da aplicação( dev, stg ou prod ). 
- **username** → usuário que comitará as alterações quando a pipeline executar.
- **gh-token** → token do GitHub
- **repository-name** → repositório que será clonado durante a execução da pipeline.
- **app-name** → aplicação
- **platform-name** → plataforma
- **path-name** → caminho da pasta da aplicação no repositório clonado.
- **email** → email do usuário que comitará as alterações.
- **influxdb-token** → token de acesso a instancia do InfluxDB
- **influxdb-url** → url da instancia do InfluxDB
- **slack-hook** → token de acesso ao Slack
- **argocd-domain** → dominío interno do ArgoCD

## Etapas da Action
- Checkout do GitHub
- Configuração das credenciais da AWS
- Login no Amazon ECR
- Obter a versão do app
- Inserir tag na imagem Docker do ECR
- Inserir tag na imagem Docker do GHCR
- Envio da imagem Docker para o ECR
- Login do Docker no GHRC
- Envio da imagem Docker para o GHCR
- Substituição da env $DOCKER_IMAGE pela imagem Docker correspondente, no arquivo deploy/rollout-template-enviroment.yaml
- Clone do repositório de deployments e envio do arquivo deploy/rollout-template-enviroment.yaml para a pasta correspondente no repositório, sobreescrevendo o arquivo rollout.yaml
- Configuração do usuário admin do GitHub( nome e email )
- Commit das alterações do arquivo rollout.yaml no repositório de deployments
- Envio do commit
- Envio da métrica de rollback para InfluxDB. Executa apenas se o ambiente for prod. Execução a partir da action [datapoint-producer-action](https://github.com/contraktor-tech/datapoint-producer-action)
- Envio da métrica de deploy para InfluxDB. Executa apenas se o ambiente for prod. Execução a partir da action [datapoint-producer-action](https://github.com/contraktor-tech/datapoint-producer-action)
- Envio de notificação para Slack. Notificação de tentativa de deploy caso a pipeline execute sem erro. Execução a partir da action [slack-notification-action](https://github.com/contraktor-tech/slack-notification-action)
- Envio de notificação para Slack. Notificação de erro caso a pipeline apresente erro. Execução a partir da action [slack-notification-action](https://github.com/contraktor-tech/slack-notification-action)
- Exclusão de pacotes GHCR não tageados do repositório. Execução a partir da action [delete-untagged-ghcr-action](https://github.com/i9cloud-tech/delete-untagged-ghcr-action)

## Como usar
Acesse a documentação no [Confluence](https://contraktor.atlassian.net/wiki/spaces/CONTRAKTOR/pages/16842753/Actions#argocd-deploy-action) para ver uma pipeline de exemplo.

```
name: deploy-ambiente-argocd
on:
  push:
    branches:
      - 

env:
  APP_NAME: '<nome-da-aplicação>'

jobs:
  publish:
    runs-on: docker
    steps:
      - uses: actions/checkout@v2

      - name: Build docker image
        run: |
          docker build --build-arg <args> -t ${APP_NAME}:latest .

      - name: Deploy to Argo CD
        uses: contraktor-tech/argocd-deploy-action@v3
        with:
          aws-access-key-id: 
          aws-secret-access-key: 
          owner: 
          environment: 
          username: 
          gh-token: 
          repository-name: 
          app-name: ${{ env.APP_NAME }}
          platform-name: 
          path-name: '<ambiente>/<plataforma>/${{ env.APP_NAME }}'
          email: 
          influxdb-token: 
          influxdb-url: 
          slack-hook: 
          argocd-domain: 
```
