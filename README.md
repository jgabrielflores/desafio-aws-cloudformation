<div align="center">

# Infraestrutura como Código com AWS CloudFormation

[![AWS](https://img.shields.io/badge/Amazon_AWS-232F3E?style=for-the-badge&logo=amazonaws&logoColor=white)](https://aws.amazon.com/)
[![CloudFormation](https://img.shields.io/badge/AWS_CloudFormation-FF4F8B?style=for-the-badge&logo=amazonaws&logoColor=FF4F8B)](https://aws.amazon.com/cloudformation/)
[![EC2](https://img.shields.io/badge/Amazon_EC2-FF9900?style=for-the-badge&logo=amazonec2&logoColor=white)](https://aws.amazon.com/ec2/)
[![S3](https://img.shields.io/badge/Amazon_S3-569A31?style=for-the-badge&logo=amazons3&logoColor=white)](https://aws.amazon.com/s3/)
[![IAM](https://img.shields.io/badge/AWS_IAM-DD344C?style=for-the-badge&logo=amazonaws&logoColor=DD344C)](https://aws.amazon.com/iam/)
[![Status](https://img.shields.io/badge/Status-Conclu%C3%ADdo-brightgreen?style=for-the-badge)](.)

**Bootcamp GFT — Fundamentos de Cloud com AWS** · [DIO](https://www.dio.me/)

</div>

---

## 🎯 Objetivo

Este laboratório tem como objetivo implementar a primeira Stack com AWS CloudFormation, aplicando na prática os conceitos de **Infraestrutura como Código (IaC)**. O entregável é um repositório organizado contendo o template utilizado, o registro do deploy real na AWS e os aprendizados da prática.

---

## ⚙️ Tecnologias Utilizadas

| Serviço | Uso |
|---|---|
| **AWS CloudFormation** | Provisionamento de infraestrutura via IaC (template YAML) |
| **Amazon EC2** | Instância Linux com servidor Apache configurado via UserData |
| **Amazon S3** | Bucket de armazenamento de objetos |
| **AWS IAM** | Grupo e usuário com permissões gerenciadas |
| **AWS Systems Manager** | Resolução automática da AMI mais recente (SSM Parameter) |

---

## 🏗️ Arquitetura da Solução

A stack cria **5 recursos** em uma única execução:

```
                Stack: desafio-cloudformation
   ┌───────────────────────────────────────────────┐
   │                                               │
   │   ┌─────────────────────────────┐             │
   │   │  Security Group             │             │
   │   │  ┌───────────────────────┐  │             │
   │   │  │  EC2 Instance         │  │             │
   │   │  │  + Apache (UserData)  │  │             │
   │   │  └───────────────────────┘  │             │
   │   └─────────────────────────────┘             │
   │                                               │
   │   ┌──────────┐  ┌────────────┐  ┌──────────┐  │
   │   │ Bucket S3│  │ Grupo IAM  │←─│ User IAM │  │
   │   └──────────┘  └────────────┘  └──────────┘  │
   │                                               │
   └───────────────────────────────────────────────┘
```

| Recurso Lógico | Tipo AWS | Descrição |
|---|---|---|
| `GrupoSeguranca` | `AWS::EC2::SecurityGroup` | Firewall liberando portas **80** (HTTP) e **22** (SSH) |
| `EC2Instance` | `AWS::EC2::Instance` | Instância Linux com Apache instalado via `UserData` |
| `S3Bucket` | `AWS::S3::Bucket` | Bucket com nome único por conta (`${AWS::AccountId}`) |
| `IAMGroup` | `AWS::IAM::Group` | Grupo IAM `GPO-ADMIN-LAB` |
| `IAMUser` | `AWS::IAM::User` | Usuário IAM associado ao grupo |

### Parâmetros do Template

| Parâmetro | Função |
|---|---|
| `InstanceType` | Tipo da EC2 (`t2.micro` ou `t3.micro`) |
| `KeyName` | Par de chaves para acesso SSH |
| `LatestAmiId` | Resolve a AMI mais recente do Amazon Linux 2023 via SSM automaticamente |

### Outputs

A stack expõe: ID da instância, **URL do site Apache**, nome do bucket S3 e nome do usuário IAM.

---

## O que é AWS CloudFormation?

O **AWS CloudFormation** é o serviço de **Infraestrutura como Código (IaC)** da AWS. Em vez de criar recursos manualmente no console, você **descreve a infraestrutura em um arquivo de texto** (YAML ou JSON) e o CloudFormation provisiona tudo automaticamente.

| Conceito | Descrição |
|---|---|
| **Template** | Arquivo YAML/JSON que descreve os recursos a criar |
| **Stack** | Conjunto de recursos criados a partir de um template — gerenciados juntos |
| **Recurso** | Cada componente AWS declarado no template (EC2, bucket S3, etc.) |

**Funções intrínsecas utilizadas:**

- `!Ref` — referencia outro recurso ou parâmetro
- `!Sub` — substitui variáveis dentro de um texto (ex.: `${AWS::AccountId}`)
- `!FindInMap` — busca um valor na seção `Mappings`
- `Fn::Base64` — codifica texto em Base64 (exigido pelo `UserData` da EC2)

---

## 📋 Passo a Passo

Deploy real executado na região **us-east-1 (Norte da Virgínia)**.

### Pré-requisito — Criar o Key Pair

Em **EC2 → Key Pairs → Create key pair**, crie a chave `key-desafio-cloudformation` (tipo **RSA**, formato **`.pem`**).

![Criação do Key Pair](images/01-criar-key-pair.png)

### Etapa 1 — Upload do Template

Em **CloudFormation → Create stack → With new resources**, escolha **Upload a template file** e envie o arquivo `projeto-cloudformation.yaml`.

![Upload do template](images/02-upload-template.png)

### Etapa 2 — Parâmetros da Stack

Defina o nome da stack como `desafio-cloudformation` e preencha os parâmetros: `InstanceType = t2.micro`, `KeyName = key-desafio-cloudformation` e `LatestAmiId` no valor padrão.

![Parâmetros da stack](images/03-parametros-stack.png)

### Etapa 3 — Opções e Capabilities

Adicione a tag `Projeto = Desafio-DIO-GFT`. Marque o checkbox de **Capabilities** (`CAPABILITY_IAM`) — obrigatório pois a stack cria recursos IAM.

![Opções da stack](images/04-stack-options.png)

### Etapa 4 — Revisão e Criação

Revise todas as configurações e clique em **Submit**.

![Revisão da stack](images/05-review-stack.png)

### Etapa 5 — Stack Criada com Sucesso

Após o provisionamento, todos os **5 recursos** aparecem com status `CREATE_COMPLETE`.

![Recursos criados](images/06-recursos-criados.png)

### Etapa 6 — Outputs da Stack

A aba **Outputs** retorna os valores úteis: ID da instância, `WebsiteURL`, nome do bucket e nome do usuário IAM.

![Outputs da stack](images/07-outputs.png)

---

## 📸 Evidências

### Site Apache Funcionando

Acessando a `WebsiteURL` dos Outputs, o servidor Apache responde — instalado automaticamente pelo `UserData` na criação da instância.

![Página do Apache](images/08-pagina-apache.png)

### Recursos Criados na AWS

**Instância EC2** — `t2.micro`, status *Running*, com a tag `Name = Webserver-CloudFormation`:

![Instância EC2](images/09-ec2-instance.png)

**Security Group** — regras de entrada liberando as portas 80 (HTTP) e 22 (SSH):

![Security Group](images/10-security-group.png)

**Bucket S3** — sufixo com ID da conta para garantir nome globalmente único:

![Bucket S3](images/11-bucket-s3.png)

**Usuário IAM** — associado ao grupo `GPO-ADMIN-LAB`:

![Usuário IAM](images/12-usuario-iam.png)

**Template interpretado pelo CloudFormation:**

![Template no console](images/13-template-yaml.png)

---

## 💡 Aprendizados

- **IaC muda a forma de pensar infraestrutura.** Em vez de clicar para criar, você *descreve o estado desejado* e a AWS reconcilia. A infraestrutura vira código revisável, versionável e repetível.
- **A stack é uma unidade de ciclo de vida.** Criar, atualizar e — principalmente — **destruir** tudo de uma vez evita recursos órfãos gerando custo.
- **Templates de materiais didáticos precisam de revisão crítica.** AMIs e VPCs são específicas de conta/região/data; copiar sem adaptar leva a falhas no deploy. Resolver a AMI via **SSM Parameter** foi o aprendizado técnico mais valioso desta prática.
- **Nomes de recursos têm regras.** Buckets S3 precisam de nome **globalmente único e em minúsculas** — usar `${AWS::AccountId}` no `!Sub` garante isso de forma elegante.
- **Recursos IAM exigem `CAPABILITY_IAM`** — uma trava de segurança da AWS para evitar criação acidental de permissões.
- **`UserData` automatiza a configuração do servidor** — a EC2 já nasce com Apache instalado e rodando, sem intervenção manual.

---

## 🧹 Limpeza dos Recursos

Para não gerar custos após a prática:

1. No CloudFormation, selecione a stack `desafio-cloudformation`
2. Clique em **Delete** e confirme
3. O CloudFormation remove **todos os recursos** automaticamente
4. Confirme que a EC2 foi para o estado *terminated*

> Se o bucket S3 contiver objetos, esvazie-o antes de deletar a stack.

---

## 🗂️ Estrutura do Repositório

```
desafio-aws-cloudformation/
├── README.md
├── templates/
│   ├── projeto-cloudformation.yaml    # Template consolidado (deploy na AWS)
│   ├── 01-EC2.yaml                    # Material da aula — referência
│   ├── 02-Apache.yaml                 # Material da aula — referência
│   ├── 03-Firewall.yaml               # Material da aula — referência
│   └── 04-EC2_S3_UserGroup.yaml       # Material da aula — referência
└── images/                            # Screenshots do deploy real (01 a 13)
    ├── 01-criar-key-pair.png
    ├── 02-upload-template.png
    ├── 03-parametros-stack.png
    ├── 04-stack-options.png
    ├── 05-review-stack.png
    ├── 06-recursos-criados.png
    ├── 07-outputs.png
    ├── 08-pagina-apache.png
    ├── 09-ec2-instance.png
    ├── 10-security-group.png
    ├── 11-bucket-s3.png
    ├── 12-usuario-iam.png
    └── 13-template-yaml.png
```

---

## ✅ Conclusão

Este laboratório consolidou na prática o conceito de **Infraestrutura como Código** com AWS CloudFormation. A principal descoberta foi que templates de materiais didáticos frequentemente precisam de adaptações para funcionar em contas reais — AMIs fixas envelhecem, VPCs são específicas por conta e nomes de recursos seguem regras que o material nem sempre documenta. Desenvolver o template consolidado desde o zero, corrigindo cada problema encontrado, foi a experiência de aprendizado mais significativa desta prática.

---

## 📚 Referências

- [Documentação AWS CloudFormation](https://docs.aws.amazon.com/cloudformation/)
- [Referência de Tipos de Recurso](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-template-resource-type-ref.html)
- [SSM Parameter Store — AMIs](https://docs.aws.amazon.com/systems-manager/latest/userguide/parameter-store-public-parameters-ami.html)
- [DIO — GFT Fundamentos de Cloud com AWS](https://www.dio.me/)

---

<div align="center">

Desenvolvido como parte do bootcamp **GFT — Fundamentos de Cloud com AWS** na [DIO](https://www.dio.me/)

</div>
