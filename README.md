<<<<<<< HEAD
# azure01



## Getting started

To make it easy for you to get started with GitLab, here's a list of recommended next steps.

Already a pro? Just edit this README.md and make it your own. Want to make it easy? [Use the template at the bottom](#editing-this-readme)!

## Add your files

* [Create](https://docs.gitlab.com/user/project/repository/web_editor/#create-a-file) or [upload](https://docs.gitlab.com/user/project/repository/web_editor/#upload-a-file) files
* [Add files using the command line](https://docs.gitlab.com/topics/git/add_files/#add-files-to-a-git-repository) or push an existing Git repository with the following command:

```
cd existing_repo
git remote add origin https://gitlab.com/kazu02/azure01.git
git branch -M main
git push -uf origin main
```

## Integrate with your tools

* [Set up project integrations](https://gitlab.com/kazu02/azure01/-/settings/integrations)

## Collaborate with your team

* [Invite team members and collaborators](https://docs.gitlab.com/user/project/members/)
* [Create a new merge request](https://docs.gitlab.com/user/project/merge_requests/creating_merge_requests/)
* [Automatically close issues from merge requests](https://docs.gitlab.com/user/project/issues/managing_issues/#closing-issues-automatically)
* [Enable merge request approvals](https://docs.gitlab.com/user/project/merge_requests/approvals/)
* [Set auto-merge](https://docs.gitlab.com/user/project/merge_requests/auto_merge/)

## Test and Deploy

Use the built-in continuous integration in GitLab.

* [Get started with GitLab CI/CD](https://docs.gitlab.com/ci/quick_start/)
* [Analyze your code for known vulnerabilities with Static Application Security Testing (SAST)](https://docs.gitlab.com/user/application_security/sast/)
* [Deploy to Kubernetes, Amazon EC2, or Amazon ECS using Auto Deploy](https://docs.gitlab.com/topics/autodevops/requirements/)
* [Use pull-based deployments for improved Kubernetes management](https://docs.gitlab.com/user/clusters/agent/)
* [Set up protected environments](https://docs.gitlab.com/ci/environments/protected_environments/)

***

# Editing this README

When you're ready to make this README your own, just edit this file and use the handy template below (or feel free to structure it however you want - this is just a starting point!). Thanks to [makeareadme.com](https://www.makeareadme.com/) for this template.

## Suggestions for a good README

Every project is different, so consider which of these sections apply to yours. The sections used in the template are suggestions for most open source projects. Also keep in mind that while a README can be too long and detailed, too long is better than too short. If you think your README is too long, consider utilizing another form of documentation rather than cutting out information.

## Name
Choose a self-explaining name for your project.

## Description
Let people know what your project can do specifically. Provide context and add a link to any reference visitors might be unfamiliar with. A list of Features or a Background subsection can also be added here. If there are alternatives to your project, this is a good place to list differentiating factors.

## Badges
On some READMEs, you may see small images that convey metadata, such as whether or not all the tests are passing for the project. You can use Shields to add some to your README. Many services also have instructions for adding a badge.

## Visuals
Depending on what you are making, it can be a good idea to include screenshots or even a video (you'll frequently see GIFs rather than actual videos). Tools like ttygif can help, but check out Asciinema for a more sophisticated method.

## Installation
Within a particular ecosystem, there may be a common way of installing things, such as using Yarn, NuGet, or Homebrew. However, consider the possibility that whoever is reading your README is a novice and would like more guidance. Listing specific steps helps remove ambiguity and gets people to using your project as quickly as possible. If it only runs in a specific context like a particular programming language version or operating system or has dependencies that have to be installed manually, also add a Requirements subsection.

## Usage
Use examples liberally, and show the expected output if you can. It's helpful to have inline the smallest example of usage that you can demonstrate, while providing links to more sophisticated examples if they are too long to reasonably include in the README.

## Support
Tell people where they can go to for help. It can be any combination of an issue tracker, a chat room, an email address, etc.

## Roadmap
If you have ideas for releases in the future, it is a good idea to list them in the README.

## Contributing
State if you are open to contributions and what your requirements are for accepting them.

For people who want to make changes to your project, it's helpful to have some documentation on how to get started. Perhaps there is a script that they should run or some environment variables that they need to set. Make these steps explicit. These instructions could also be useful to your future self.

You can also document commands to lint the code or run tests. These steps help to ensure high code quality and reduce the likelihood that the changes inadvertently break something. Having instructions for running tests is especially helpful if it requires external setup, such as starting a Selenium server for testing in a browser.

## Authors and acknowledgment
Show your appreciation to those who have contributed to the project.

## License
For open source projects, say how it is licensed.

## Project status
If you have run out of energy or time for your project, put a note at the top of the README saying that development has slowed down or stopped completely. Someone may choose to fork your project or volunteer to step in as a maintainer or owner, allowing your project to keep going. You can also make an explicit request for maintainers.
=======
# GitLab CI/CD com AWS Fargate — Self-hosted Runner

Projeto de estudo de CI/CD utilizando GitLab Runner hospedado em EC2 na AWS, com deploy automatizado no Amazon ECS Fargate.

---

## Visão Geral

```
git push
  └── GitLab detecta o push
        └── pipeline dispara no runner (EC2)
              └── build: docker build + push pro ECR
                    └── deploy: atualiza task definition + rolling update no Fargate
                          └── aplicação rodando no Fargate
```

---

## Infraestrutura (Terraform)

### Recursos criados

- **VPC + Subnet pública** — rede sem NAT Gateway (economia de ~$32/mês)
- **Security Group** — porta 3000 aberta para acesso externo
- **IAM Role (`runner-ecr-role`)** — permissão para a EC2 acessar ECR e ECS sem access key
- **EC2 (`t2.micro`)** — self-hosted runner do GitLab
- **ECR (`codepipeline-repo`)** — repositório de imagens Docker
- **ECS Cluster (`fargate-cluster-kazu`)** — ambiente serverless Fargate
- **Task Definition (`kazu-app-task`)** — receita do container (0.25 vCPU, 0.5GB RAM, porta 3000)
- **ECS Service (`kazu-app-service`)** — gerencia e mantém a task rodando

### Arquivos

```
terraform/
├── main.tf       # EC2, Security Group, VPC
├── iam.tf        # IAM Role e Instance Profile
└── outputs.tf    # IP público da EC2
```

### Comandos

```bash
terraform init
terraform apply
terraform destroy
```

---

## Pipeline CI/CD (GitLab)

### Estrutura do `.gitlab-ci.yml`

```yaml
stages:
  - test
  - build
  - deploy

variables:
  ECR_REGISTRY: $AWS_ACCOUNT_ID.dkr.ecr.us-east-1.amazonaws.com
  ECR_REPO: codepipeline-repo
  REGION: us-east-1
  ECS_CLUSTER: fargate-cluster-kazu
  ECS_SERVICE: kazu-app-service
  ECS_TASK_FAMILY: "kazu-app-task"
```

### Stages

**test** — roda os testes automatizados
```yaml
realizar_testes:
  stage: test
  tags:
    - aws
  before_script:
    - cd app/
    - npm install
  script:
    - npm test
  artifacts:
    when: always
    reports:
      junit: app/junit.xml
```

**build** — gera a imagem Docker e faz push pro ECR
```yaml
build_images:
  stage: build
  tags:
    - aws
  script:
    - aws ecr get-login-password --region $REGION | docker login --username AWS --password-stdin $ECR_REGISTRY
    - docker build -t $ECR_REGISTRY/$ECR_REPO:$CI_COMMIT_SHORT_SHA .
    - docker push $ECR_REGISTRY/$ECR_REPO:$CI_COMMIT_SHORT_SHA
```

**deploy** — atualiza a task definition e faz rolling update no Fargate
```yaml
deploy_fargate:
  stage: deploy
  tags:
    - aws
  script:
    - # busca task definition atual
    - # registra nova task com imagem do commit
    - aws ecs update-service --cluster $ECS_CLUSTER --service $ECS_SERVICE --force-new-deployment
    - aws ecs wait services-stable --cluster $ECS_CLUSTER --services $ECS_SERVICE
```

---

## Configuração do Runner

### Instalação na EC2

```bash
# instala o gitlab-runner
curl -L "https://packages.gitlab.com/install/repositories/runner/gitlab-runner/script.deb.sh" | sudo bash
sudo apt install -y gitlab-runner

# registra o runner
sudo gitlab-runner register \
  --url https://gitlab.com \
  --token glrt-SEU_TOKEN
```

### Config (`/etc/gitlab-runner/config.toml`)

```toml
concurrent = 1
[[runners]]
  name = "ip-172-31-2-63"
  url = "https://gitlab.com"
  executor = "shell"
  tag_list = ["aws", "cloud", "linux"]
  request_concurrency = 2
```

### Comandos úteis

```bash
sudo gitlab-runner start
sudo gitlab-runner restart
sudo gitlab-runner status
sudo journalctl -u gitlab-runner -n 50
```

---

## Variáveis do GitLab

Configurar em `Settings → CI/CD → Variables`:

| Variável | Descrição | Protegida |
|----------|-----------|-----------|
| `AWS_ACCOUNT_ID` | ID da conta AWS (ex: 776743135454) | Masked |

---

## IAM Role

A EC2 usa uma IAM Role para autenticar na AWS **sem access key**.

Policies anexadas:
- `AmazonEC2ContainerRegistryFullAccess` — acesso ao ECR
- `AmazonECS_FullAccess` — acesso ao ECS/Fargate

Como verificar se está funcionando:
```bash
aws ecr get-login-password --region us-east-1
```

---

## Rolling Update (Zero Downtime)

O Fargate faz o deploy sem derrubar a aplicação:

```
1. v1 rodando (100% tráfego)
2. Fargate sobe v2 junto
3. v2 passa no health check
4. tráfego migra para v2
5. v1 é removida
```

O pipeline aguarda a estabilização antes de concluir:
```bash
aws ecs wait services-stable --cluster $ECS_CLUSTER --services $ECS_SERVICE
```

---

## Estrutura do Projeto

```
.
├── .gitlab-ci.yml       # pipeline CI/CD
├── Dockerfile           # imagem da aplicação
├── README.md            # este arquivo
├── app/                 # código da aplicação Node.js
│   ├── index.js
│   ├── package.json
│   └── tests/
└── terraform/           # infraestrutura como código
    ├── main.tf
    ├── iam.tf
    └── outputs.tf
```

---

## Divisão de Responsabilidades

| Ferramenta | Responsabilidade |
|------------|-----------------|
| **Terraform** | Infraestrutura (VPC, EC2, ECS, IAM) — roda uma vez ou quando muda a estrutura |
| **GitLab CI/CD** | Código e deploy — roda a cada `git push` |
| **ECR** | Armazena as imagens Docker |
| **Fargate** | Executa os containers sem gerenciar servidores |

---

## Comparativo com outras ferramentas

| GitLab CI/CD | Azure DevOps | AWS CodePipeline |
|-------------|--------------|------------------|
| Runner | Agent | CodeBuild |
| `.gitlab-ci.yml` | `azure-pipelines.yml` | buildspec.yml |
| ECR | ACR | ECR |
| ECS/Fargate | ACI / AKS | ECS/Fargate |

> O conceito de self-hosted runner é idêntico no GitLab e no Azure DevOps.
> Com CodePipeline seriam necessários 4 serviços AWS (CodeCommit + CodeBuild + CodePipeline + CodeDeploy).

---

## Melhorias para Produção

- [ ] Ambientes separados (dev, homolog, prod)
- [ ] `when: manual` no deploy de produção
- [ ] Notificação no Slack após deploy
- [ ] Variáveis protegidas por ambiente
- [ ] Load Balancer para IP fixo no Fargate
- [ ] Auto scaling no ECS Service
>>>>>>> aa6954d (Initial commit: Infra e Pipeline)
