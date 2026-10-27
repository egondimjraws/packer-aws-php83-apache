# Packer AWS — AMI com PHP 8.3 e Apache

Atividade de criação de uma imagem de máquina (AMI) na AWS usando o [HashiCorp Packer](https://www.packer.io/). A AMI gerada já sai com Apache, PHP 8.3 e o código da aplicação instalados, pronta para subir instâncias EC2.

> **Atividade didática:** este projeto tem fins de estudo. O template foi apenas validado (`packer validate`) e o build não foi executado na AWS — nenhuma AMI foi efetivamente criada. As seções de execução e teste abaixo descrevem como seria o processo em uma conta AWS configurada.

## Objetivo

Automatizar a construção de uma imagem imutável de servidor web, em vez de configurar cada instância manualmente após o boot.

## Estrutura

```
.
├── .gitignore       # Ignora cache, logs e arquivos sensíveis do Packer
├── LICENSE          # Licença MIT
├── amazon.pkr.hcl   # Template Packer (HCL)
└── app/
    └── index.php    # Aplicação de exemplo copiada para o webroot
```

## O que o template faz

1. **Fonte (`amazon-ebs`)**: sobe uma instância temporária `t3.micro` a partir da AMI mais recente do **Amazon Linux 2023** (x86_64, owner `amazon`) na região definida por `aws_region` (padrão `us-east-1`).
2. **Provisionamento**:
   - Atualiza o sistema e instala `httpd`, `php8.3`, `php8.3-fpm` e extensões (`mbstring`, `xml`, `mysqlnd`, `gd`, `zip`);
   - Habilita os serviços `httpd` e `php-fpm` para iniciar no boot;
   - Copia a pasta `app/` para `/var/www/html` e ajusta dono (`apache:apache`) e permissões.
3. **Resultado**: a instância é parada, vira uma AMI chamada `app-php83-apache-<timestamp>` e é encerrada.

## Pré-requisitos

- [Packer](https://developer.hashicorp.com/packer/install) instalado;
- Credenciais da AWS configuradas (`aws configure` ou variáveis `AWS_ACCESS_KEY_ID` / `AWS_SECRET_ACCESS_KEY`) com permissão para criar instâncias EC2 e AMIs.

## Como executar

```bash
packer init .
packer validate amazon.pkr.hcl
packer build amazon.pkr.hcl
```

Para usar outra região:

```bash
packer build -var "aws_region=sa-east-1" amazon.pkr.hcl
```

Ao final, o Packer exibe o ID da AMI criada.

## Testando a AMI

1. Lance uma instância EC2 a partir da AMI gerada;
2. Libere a porta **80** no Security Group;
3. Acesse `http://<ip-publico-da-instancia>/` — a página deve mostrar a versão do PHP e o hostname do servidor.

## Observação de custos

O build cria recursos temporários (instância, volume EBS, key pair e security group) que o Packer remove ao final. A AMI e o snapshot gerados permanecem na conta e geram custo de armazenamento — remova-os (deregister + delete snapshot) quando não forem mais necessários.

## Licença

Distribuído sob a licença MIT. Veja o arquivo [LICENSE](LICENSE).
