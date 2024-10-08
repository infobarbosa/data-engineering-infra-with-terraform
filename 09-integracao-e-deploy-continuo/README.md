# Módulo 9: Integração e Deploy Contínuo

Author: Prof. Barbosa  
Contact: infobarbosa@gmail.com  
Github: [infobarbosa](https://github.com/infobarbosa)

## Atenção aos custos!
A gestão dos custos gerados pelos recursos criados durante os laboratórios é de responsabilidade do aluno. Certifique-se de destruir todos os recursos após a conclusão dos exercícios.

## Teoria

### Integração do Terraform com CI/CD (GitHub Actions)
A integração contínua (CI) e o deploy contínuo (CD) são práticas essenciais no desenvolvimento moderno de software. O Terraform pode ser integrado com pipelines de CI/CD para automatizar a validação e o deploy de infraestrutura. O GitHub Actions é uma ferramenta poderosa para configurar essas pipelines.

### Testes e Validação de Código Terraform
Antes de aplicar mudanças na infraestrutura, é importante validar e testar o código Terraform. Isso pode incluir a execução de comandos como `terraform fmt`, `terraform validate`, e `terraform plan`.

### Deploy Contínuo de Infraestrutura
O deploy contínuo permite que mudanças na infraestrutura sejam aplicadas automaticamente após a validação. Isso garante que a infraestrutura esteja sempre atualizada e em conformidade com o código.

## Laboratório

### Exercício 1: Configurar Pipeline de CI/CD para Validar Código Terraform

1. Crie a estrutura de pastas para o Terraform e GitHub Actions:
    ```
    terraform/
    ├── main.tf
    ├── variables.tf
    ├── outputs.tf
    .github/
    └── workflows/
        └── ci-cd-pipeline.yml
    ```

2. Adicione o seguinte conteúdo ao arquivo `main.tf`:
    ```hcl
    provider "aws" {
      region = "us-east-1"
    }

    resource "aws_s3_bucket" "dataeng_bucket" {
      bucket = "dataeng-${random_string.suffix.result}"
    }

    resource "random_string" "suffix" {
      length  = 10
      special = false
      upper   = false
    }
    ```

3. Adicione o seguinte conteúdo ao arquivo `.github/workflows/ci-cd-pipeline.yml`:
    ```yaml
    name: CI/CD Pipeline

    on:
      push:
        branches:
          - main
      pull_request:
        branches:
          - main
    jobs:
      terraform:
        name: 'Terraform'
        runs-on: ubuntu-latest
        environment: 
          name: "AWS"

        steps:
          - name: 'Checkout code'
            uses: actions/checkout@v3

          - name: 'Set up Terraform'
            uses: hashicorp/setup-terraform@v3
            with:
              terraform_version: 1.9.2

          - name: 'Terraform Format'
            run: terraform fmt -check

          - name: 'Terraform Init'
            run: terraform init

          - name: 'Terraform Validate'
            run: terraform validate

          - name: 'Terraform Plan'
            run: terraform plan
            env:
              AWS_ACCESS_KEY_ID: ${{ secrets.AWS_ACCESS_KEY_ID }}
              AWS_SECRET_ACCESS_KEY: ${{ secrets.AWS_SECRET_ACCESS_KEY }}        

    ```

6. Faça commit e push do código para o repositório GitHub.

### Exercício 2: Implementar Deploy Contínuo de Infraestrutura usando GitHub Actions

1. Adicione o seguinte conteúdo ao arquivo `.github/workflows/ci-cd-pipeline.yml` para incluir o deploy contínuo:
    ```yaml
    name: CI/CD Pipeline

    on:
      push:
        branches:
          - main
      pull_request:
        branches:
          - main

    jobs:
      terraform:
        name: 'Terraform'
        runs-on: ubuntu-latest
        environment: 
          name: "AWS"

        steps:
          - name: 'Checkout code'
            uses: actions/checkout@v4

          - name: 'Set up Terraform'
            uses: hashicorp/setup-terraform@v3
            with:
              terraform_version: 1.9.2

          - name: 'Terraform Format'
            run: terraform fmt -check

          - name: 'Terraform Init'
            run: terraform init

          - name: 'Terraform Validate'
            run: terraform validate

          - name: 'Terraform Plan'
            run: terraform plan
            env:
              AWS_ACCESS_KEY_ID: ${{ secrets.AWS_ACCESS_KEY_ID }}
              AWS_SECRET_ACCESS_KEY: ${{ secrets.AWS_SECRET_ACCESS_KEY }}

          - name: 'Terraform Apply'
            if: github.ref == 'refs/heads/main'
            run: terraform apply -auto-approve
            env:
              AWS_ACCESS_KEY_ID: ${{ secrets.AWS_ACCESS_KEY_ID }}
              AWS_SECRET_ACCESS_KEY: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
    ```

2. Crie um environment no repositório Github chamado `AWS`.
3. Adicione as credenciais AWS como segredos no repositório GitHub (`AWS_ACCESS_KEY_ID` e `AWS_SECRET_ACCESS_KEY`).
4. Faça commit e push do código para o repositório GitHub.

## Parabéns
Você concluiu o módulo! Agora você sabe como configurar uma pipeline de CI/CD para validar e fazer deploy contínuo de infraestrutura na AWS usando GitHub Actions.

## Destruição dos recursos
Para evitar custos adicionais, destrua os recursos criados:
```sh
terraform destroy --auto-approve
```
