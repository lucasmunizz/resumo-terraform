# Terraform + AWS – Resumo Rápido

## O que é Terraform?
Terraform é uma ferramenta de IaC (Infrastructure as Code) da HashiCorp que permite definir infra na nuvem usando arquivos de configuração. Ele gera e aplica planos de execução para criar, modificar e deletar recursos de forma declarativa.

## Por que usar Terraform com AWS?
- Infra como código: infraestrutura versionada e replicável.
- Automatização: menos cliques na AWS Console, mais produtividade.
- Controle total: planejamento com `terraform plan` antes de aplicar mudanças.

---

## Estrutura Básica
```hcl
provider "aws" {
  region = "us-east-1"
}

resource "aws_s3_bucket" "meu_bucket" {
  bucket = "meu-bucket-exemplo"
  acl    = "private"
}
```
### Explicando:
- **Provider**: Define qual nuvem usar (AWS, Azure, GCP, etc.).
- **Resource**: Representa um recurso (ex: um S3 bucket, uma instância EC2, etc.).
- **acl**: Define permissão do bucket.

---

## Comandos Essenciais
```sh
terraform init       # Inicializa o diretório e baixa dependências
terraform plan       # Mostra o que vai ser criado/modificado
terraform apply      # Aplica as mudanças
terraform destroy    # Deleta tudo que foi criado
```

---

## Trabalhando com Variáveis
```hcl
variable "region" {
  default = "us-east-1"
}

provider "aws" {
  region = var.region
}
```

### Como passar valores:
- No `terraform.tfvars`
  ```hcl
  region = "us-west-2"
  ```
- Via linha de comando:
  ```sh
  terraform apply -var="region=us-west-2"
  ```

---

## Outputs
Pra pegar informações da infra criada e reutilizar.
```hcl
output "bucket_name" {
  value = aws_s3_bucket.meu_bucket.id
}
```
```sh
terraform output bucket_name
```

---

## Módulos
Módulos ajudam a reutilizar código e organizar infra grande. Eles permitem encapsular configurações reutilizáveis e separar componentes da infraestrutura em arquivos menores.

### Criando um Módulo
1. Crie um diretório para o módulo:
    ```sh
    mkdir -p modules/s3
    ```
2. Dentro do diretório, crie os seguintes arquivos:
    - `main.tf` (definição dos recursos)
    - `variables.tf` (variáveis do módulo)
    - `outputs.tf` (saídas do módulo)

### Exemplo de um módulo para um S3 Bucket:
#### `modules/s3/main.tf`
```hcl
resource "aws_s3_bucket" "this" {
  bucket = var.bucket_name
  acl    = "private"
}
```

#### `modules/s3/variables.tf`
```hcl
variable "bucket_name" {
  description = "Nome do bucket"
  type        = string
}
```

#### `modules/s3/outputs.tf`
```hcl
output "bucket_id" {
  value = aws_s3_bucket.this.id
}
```

### Utilizando o Módulo
```hcl
module "s3" {
  source      = "./modules/s3"
  bucket_name = "meu-bucket"
}
```

### Estrutura Final
```
/
|-- main.tf
|-- variables.tf
|-- outputs.tf
|-- modules/
    |-- s3/
        |-- main.tf
        |-- variables.tf
        |-- outputs.tf
```

Com isso, sua infraestrutura fica mais modularizada e fácil de gerenciar.

---

## Terraform State
Terraform guarda o estado da infra num arquivo `.tfstate`, essencial pra manter controle do que já foi criado.

Se estiver trabalhando em equipe, usar um backend remoto (S3 + DynamoDB) é uma boa pra evitar conflitos.
```hcl
terraform {
  backend "s3" {
    bucket         = "meu-bucket-terraform"
    key            = "terraform.tfstate"
    region         = "us-east-1"
    dynamodb_table = "terraform-lock"
  }
}
```

---

## Conclusão
Isso aqui é só o básico! Terraform tem muito mais coisa legal, tipo:
- Data sources (pra pegar dados de recursos existentes)
- Workspaces (pra separar ambientes tipo dev, staging e prod)
- Provisioners (pra rodar scripts depois que o recurso é criado)

Dá pra explorar bem mais e deixar a infra toda automatizada

---

### Referências
- [Documentação oficial do Terraform](https://developer.hashicorp.com/terraform/docs)
- [Módulos oficiais no Terraform Registry](https://registry.terraform.io/)

