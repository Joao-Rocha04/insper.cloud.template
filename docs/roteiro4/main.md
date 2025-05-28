# IaC
Infraestrutura como Código (IaC) é uma prática que traz os princípios de desenvolvimento de software para o gerenciamento e provisionamento de infraestrutura de TI. Em vez de configurar manualmente servidores, redes e demais componentes, tudo é descrito em arquivos de configuração versionados, permitindo que ambientes sejam criados, modificados e replicados de forma automática e previsível.

## Terraform como Iac
Terraform é uma ferramenta de Infraestrutura como Código (IaC) desenvolvida pela HashiCorp que permite definir, versionar e gerenciar recursos de infraestrutura em múltiplas nuvens e provedores por meio de código declarativo. A seguir, os principais aspectos de como o Terraform se encaixa no paradigma de IaC:

- **Arquivos de configuração em HCL**
Linguagem HCL (HashiCorp Configuration Language): sintaxe legível por humanos, com blocos como provider, resource, variable e output.

- **Estado e Idempotência**
Estado remoto: o Terraform mantém um arquivo de estado (terraform.tfstate) que registra o que foi provisionado. Com isso, ele sabe exatamente o que criar, alterar ou destruir.

Idempotência: ao rodar terraform apply múltiplas vezes sobre o mesmo conjunto de configurações, o Terraform só aplicará mudanças que alterem o estado atual para o desejado — sem recriar recursos desnecessariamente.

## Instalar o terraform
Para instalar o terraform, foi necessário os seguintes comandos: 
``` bash
wget -O- https://apt.releases.hashicorp.com/gpg | gpg --dearmor | sudo tee /usr/share/keyrings/hashicorp-archive-keyring.gpg
```
``` bash
gpg --no-default-keyring --keyring /usr/share/keyrings/hashicorp-archive-keyring.gpg --fingerprint
```
``` bash
echo "deb [signed-by=/usr/share/keyrings/hashicorp-archive-keyring.gpg] https://apt.releases.hashicorp.com $(lsb_release -cs) main" | sudo tee /etc/apt/sources.list.d/hashicorp.list
```
``` bash
sudo apt update && sudo apt install terraform
```

# Infra

Neste trabalho, foi configurada a hierarquia de projetos no OpenStack via Horizon para isolar os recursos de cada aluno. Primeiramente, criou-se um domínio denominado AlunosDomain. Em seguida, gerou-se para cada estudante um projeto seguindo o padrão *Kit<letra>_<nome_do_aluno>* e definiram-se cotas específicas conforme a necessidade. Após isso, criaram-se os usuários ( “João” e “Marinna”), associaram-se cada um ao seu respectivo projeto em AlunosDomain e atribuíram-se a eles o papel de administrador. Por fim, validaram-se as permissões navegando em Identity > Projects, acessando a aba Members de cada projeto e confirmando que os usuários dispunham dos papéis corretos para gerenciar seus recursos.

# App

## Criando a Infraestrutura utilizando IaC

O Terraform conta com arquivos .tf, que definem a infraestrutura, o Terraform sabe o que é implantado por meio do arquivo de estado. Este estado é armazenado por padrão em um arquivo local chamado “terraform.tfstate”.


O Terraform permite vários provedores, normalmente inicializados em um arquivo chamado provider.tf.

A estrutura de pastas ficará conforme abaixo:    --Terraform
                                                   |--instance.tf
                                                   |--network.tf
                                                   |--provider.tf
                                                   |--router.tf

Isso foi feito para cada aluno, em uma pasta para cada.

Foi criado os arquivos abaixo:

provider.tf
``` bash 

# Terraform Openstack deployment
# Author: Tiago Demay - tiagoaodc@insper.edu.br
    # Define required providers
terraform {
    required_version = ">= 0.14.0"
    required_providers {
        openstack = {
        source  = "terraform-provider-openstack/openstack"
        version = "~> 1.35.0"
        }
    }
}
# Configure the OpenStack Provider

provider "openstack" {
region              = "RegionOne"
user_name           = "SEU_USUARIO"
}


```

instance1.tf
``` bash 
resource "openstack_compute_instance_v2" "instancia" {
  name            = "basic"
  image_name      = "bionic-amd64"
  flavor_name     = "m1.small"
  key_pair        = "mykey"
  security_groups = ["default"]

  network {
    name = "network_1"
  }

  depends_on = [openstack_networking_network_v2.network_1]

}


```

instance2.tf
``` bash 
resource "openstack_compute_instance_v2" "instancia" {
  name            = "basic2"
  image_name      = "jammy-amd64"
  flavor_name     = "m1.tiny"
  key_pair        = "mykey"
  security_groups = ["default"]

  network {
    name = "network_1"
  }

  depends_on = [openstack_networking_network_v2.network_1]

}

```

network.tf
``` bash 

resource "openstack_networking_network_v2" "network_1" {
  name           = "network_1"
  admin_state_up = "true"
}

resource "openstack_networking_subnet_v2" "subnet_1" {
  network_id = "${openstack_networking_network_v2.network_1.id}"
  cidr       = "192.167.199.0/24"
}


```

router.tf
``` bash 

resource "openstack_networking_router_v2" "router_1" {
  name                = "my_router"
  admin_state_up      = true
  external_network_id = <"ID_EXT_NETWORK">
}

resource "openstack_networking_router_interface_v2" "int_1" {
  router_id = "${openstack_networking_router_v2.router_1.id}"
  subnet_id = "${openstack_networking_subnet_v2.subnet_1.id}"
}

```

Nos arquivos acima, foi feito as alterações necessárias para cada aluno, foi criado uma key_par para cada aluno e diferentes redes.


Para fazer a implementação da infraestrutura, execute os comandos abaixo:

```  bash
terraform plan
```

Este comando é utilizado para criar um plano de execução. Ele mostra quais ações o Terraform executará quando você aplicar suas configurações.

``` bash
terraform apply
```

O comando terraform apply aplica as mudanças necessárias para alcançar o estado desejado da sua configuração. Ele cria, atualiza ou destrói os recursos conforme necessário.


### Tarefa (Ambos)

[AlunosDomain](./imagens_joao/alunos_domain.jpeg)
[Users](./imagens_joao/users.jpeg)

### Tarefa (João)

[Overview](./imagens_joao/overview.jpeg)
[Instancias](./imagens_joao/instancias.jpeg)
[Network](./imagens_joao/network.jpeg)

### Tarefa (Marinna)

[Overview](./imagens_marinna/overview.jpeg)
[Instancias](./imagens_marinna/instancias.jpeg)
[Network](./imagens_marinna/network.jpeg)


## Criando um plano de Disaster Recovery e SLA

1. Public Cloud ou Private Cloud?
- Private Cloud. Garante isolamento físico e lógico para dados sensíveis, controle de compliance (LGPD), criptografia dedicada e latência previsível entre filiais, com custo otimizado via consolidação e automação.

2. Por que precisamos de um time de DevOps?

- CI/CD: automatiza build, testes e deploy, reduzindo lead time e risco de falhas em produção.

- IaC: infraestrutura versionada e reproduzível (Terraform/Ansible), elimina drift entre ambientes.

- Observabilidade: métricas e alertas proativos para detectar e corrigir incidentes antes que impactem o usuário.

- Cultura colaborativa: elimina silos Dev × Ops, acelera entregas e resposta a mudanças.

- Segurança integrada: scanners de vulnerabilidade e policies no pipeline, garantindo compliance contínuo.

3. Plano de DR & HA

**Ameaças principais:**
1. Falha de hardware
2. Queda de energia
3. Desastre natural
4. Ataque cibernético (DDoS, ransomware)
5. Erro humano
6. Falha de link de rede


**Recuperação (priorizada)**
1. Failover automático para site secundário
2. Restore de backup crítico em até 1 h
3. Roll-back de versão pelo CI/CD
4. Execução de runbooks testados
5. Mitigação DDoS e bloqueio de IP malicioso


**Política de backup**
- Incremental diário (retêm 7 dias)
- Full semanal (4 semanas) e mensal (12 meses)
- Armazenamento off-site cifrado
- Testes de restore trimestrais


**Alta Disponibilidade**
- Clusters ativos em dois datacenters
- Load balancer distribuído
- Replicação síncrona de banco (PostgreSQL/MySQL)
- Rede redundante e BGP
- Orquestração de containers (Kubernetes) com anti-affinity



Assim, cobrimos isolamento, agilidade, segurança e resiliência para manter o sistema crítico sempre disponível e protegido.