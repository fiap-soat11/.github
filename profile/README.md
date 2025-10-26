# FIAP Pós Tech - Arquitetura de Software - Tech Challenge  
Repositórios do projeto em grupo "Tech Challenge" (Grupo 164 da SOAT11 de 2025) do curso de [Arquitetura de Software da FIAP Pós Tech](https://postech.fiap.com.br/curso/software-architecture/).

## Projeto  
O objetivo do projeto foi estudar tópicos relacionados à arquitetura de software por meio da criação de um sistema de autoatendimento de fast-food e gerenciamento da cozinha para uma lanchonete fictícia.

O cliente pode se identificar; realizar o checkout de um pedido; confirmá-lo; pagá-lo através de um serviço de pagamento externo, e o pedido pode então ser processado pelos funcionários da cozinha.
A lanchonete pode manter cadastro de clientes; gerenciar produtos e categorias; acompanhar os pedidos.

## Fases  

O projeto, assim como o curso, é dividido em várias 'fases', cada uma abordando tópicos relacionados à arquitetura de software.  
Alguns desses tópicos exigem reescrita de código, refatorações e divisão dos repositórios.

<details>
  <summary>Documentação de cada fase</summary>

### Fase 1

<details>


  - Aplicação monolítica .Net Core em Arquitetura Hexagonal com Banco de Dados Mysql  
  
  - [Event Storming (DDD)](https://miro.com/app/board/uXjVIG95Hyw=/)
    
  - [Linguagem Ubíqua](https://github.com/fiap-soat11/.github/wiki/Linguagem-Ub%C3%ADqua)
  
  - [Arquitetura Hexagonal (Ports and Adapters)](https://github.com/fiap-soat11/.github/wiki)
    
    ![DAS](/fase1/docs/DAS.png)
    
  - [Modelo de entidade relacional](https://github.com/fiap-soat11/.github/wiki/Modelo-de-dados)
  
    ![MER](/fase1/docs/MER.png)

  - [Plano de Testes](/fase1/docs/plano-de-testes.md)

</details> 

### Fase 2

<details>

#### Aplicação monolítica .Net Core em Arquitetura Limpa com Banco de Dados Mysql  

  [Repositório privado](https://github.com/fiap-soat11/backend) (branch feature/fase2 ou tag v2)

  ![Diagrama de Arquitetura de Software](/fase2/docs/DAS.png)

  ---

  **Descrição das Camadas**

  **1️⃣ Domain (Entidades de Negócio)**
  > Contém as entidades e as regras de negócio mais fundamentais do sistema.  
  > Essa camada é totalmente isolada e não depende de nenhuma tecnologia ou framework externo.  
  > Exemplo: Entidades como `Pedido`, `Cliente`, `Produto`.

  ---

  **2️⃣ Application (Casos de Uso)**
  > Define os fluxos de interação do sistema de forma independente de qualquer tecnologia.  
  > Contém os casos de uso da aplicação, orquestrando as regras de negócio para atender as necessidades dos usuários.  
  > Exemplo: CriarPedido, CalcularValorPedido.

  ---

  **3️⃣ Adapters (Controladores, Gateways, Presenters)**
  > Fazem a ponte entre o mundo externo (interfaces, bancos, APIs) e a aplicação.  
  > Realizam a orquestração dos casos de uso e a conversão de dados (DTOs).  
  > Inclui os **Controllers**, **Gateways** e **Presenters**.

  - **Controllers:** expõem a API e recebem as requisições.
  - **Gateways:** realizam integrações com bancos de dados ou serviços externos.
  - **Presenters:** formatam as respostas para o exterior (DTOs).

  ---

  **4️⃣ Datasource (Banco de Dados)**
  > Implementações concretas de acesso a dados, como repositórios e entidades de persistência.  
  > No caso deste projeto, utiliza-se o **MySQL** para armazenamento dos dados.

  ---

  **5️⃣ WebAPI / Swagger (Interface Externa)**
  > Interface Web responsável por expor as funcionalidades principais da aplicação.  
  > Permite a integração de sistemas ou o uso via ferramentas como Swagger para testes e documentação da API.

  ---

  **Fluxo Geral**

  1. O usuário faz uma requisição pela **WebAPI**.
  2. O **Controller** encaminha a requisição para o caso de uso correspondente na camada de **Application**.
  3. O caso de uso manipula as **Entidades de Negócio (Domain)** conforme a regra definida.
  4. Quando necessário, o caso de uso acessa os dados através dos **Gateways** que se comunicam com a camada de **Datasource**.
  5. A resposta é formatada pelos **Presenters** e devolvida pela API.
   
    
#### DEVOPS com Github Actions integrado ao Dockerhub
    
  ![Pipeline](/fase2/docs/DEVOPS.png)

#### Diagrama de Implantação com Kubernetes
  
  [Repositório privado](https://github.com/fiap-soat11/kubernetes) (branch feature/fase2 ou tag v2.)

  **Ambiente de Desenvolvimento - Minikube**
  ![Kubernetes](/fase2/docs/K8S-Minikube.png)

  **Ambiente de Produção**
  ![Kubernetes](/fase2/docs/K8S.png)

**1️⃣ Acesso Externo via Ingress Controller**
- O acesso à aplicação é realizado externamente através de uma **URL pública**, roteada por um **Ingress Controller (ing)**, que direciona as requisições HTTP/HTTPS para os serviços internos do cluster.

---

**2️⃣ Service e Load Balancing**
- O **Service (svc)** do tipo **ClusterIP** é utilizado para expor as aplicações dentro do cluster Kubernetes e para realizar o balanceamento de carga entre os **Pods** que compõem a aplicação.

---

**3️⃣ Deployment e ReplicaSet**
- A criação e gerenciamento dos **Pods** são realizados por meio de um **Deployment (ds)**, responsável por manter o estado desejado da aplicação.
- O **ReplicaSet** gerencia o número de réplicas dos Pods, garantindo alta disponibilidade e escalabilidade horizontal.

---

**4️⃣ Horizontal Pod Autoscaler (HPA)**
- O **HPA (Horizontal Pod Autoscaler)** é utilizado para escalar automaticamente a quantidade de réplicas com base no consumo de recursos como CPU e memória.

---

**5️⃣ ConfigMaps e Secrets**
- **ConfigMap (cm):** Armazena variáveis de ambiente e configurações não sensíveis da aplicação.
- **Secret:** Armazena informações sensíveis, como senhas e tokens, garantindo a segurança no acesso aos dados.

---

**6️⃣ Comunicação com Banco de Dados**
- A aplicação se comunica com o banco de dados através de um **Service (svc)** interno do cluster.
- O banco de dados roda em um **Pod** próprio, garantindo isolamento e controle de acesso.

---

**7️⃣ Persistent Volumes (PV / PVC)**
- O **PersistentVolumeClaim (pvc)** é responsável por requisitar volumes persistentes para armazenamento de dados.
- O **PersistentVolume (pv)** é a ligação com o armazenamento físico subjacente, garantindo a persistência dos dados, mesmo com reinícios dos Pods.

---

**Resultados HPA**

![K6-HPA](/fase2/docs/K6-HPA.png)

![K6-Results](/fase2/docs/K6-Results.png)

---

**Benefícios da Arquitetura de Implantação Kubernetes**

✅ Escalabilidade automática conforme demanda.  
✅ Balanceamento de carga eficiente entre réplicas.  
✅ Isolamento seguro por namespace.  
✅ Persistência garantida para dados críticos.  
✅ Configuração de ambiente centralizada e segura.


</details> 

### Fase 3

<details>

#### Infraestrutura AWS

  **HLD**
    ![AWS](/fase3/hld.png)

  **LLD**
    ![AWS](/fase3/lld.png)

  | **Categoria**                             | **Informação Importante**        | **Configuração de Demonstração (Free Tier)**|
  | ----------------------------------------- | -------------------------------- |----------------------------------------|  
  | **Rede e Segurança**                      | **Sub-redes**                    | 1 Sub-rede pública (para API Gateway e acesso inicial) e 2 Sub-redes privadas (para Lambda, Kubernetes e RDS).                                   |
  |                                           | **Security Groups / NACLs**      | API Gateway: HTTPS (443). Lambda e Kubernetes: acesso interno. RDS: porta 3306 liberada apenas para o cluster Kubernetes e Lambda.               |
  |                                           | **Região e AZs**                 | `us-east-1` (N. Virginia) — mais recursos gratuitos disponíveis. Sub-redes privadas replicadas em `us-east-1a` e `us-east-1b`.                   |
  |                                           | **Conectividade externa**        | Acesso público controlado via API Gateway + NAT Gateway para recursos privados quando necessário.                                                |
  | **Escalabilidade e Alta Disponibilidade** | **Auto Scaling**                 | Lambda escala automaticamente (Free Tier: 1 milhão de execuções/mês). Cluster Kubernetes em um único nó gratuito do EKS Fargate ou EC2 t2.micro. |
  |                                           | **Balanceamento de carga**       | Para demonstração: API Gateway já faz roteamento. Sem ALB/NLB adicional para evitar custos.                                                      |
  |                                           | **Resiliência**                  | RDS Single-AZ para reduzir custo, backups automáticos habilitados.                                                                               |
  | **Segurança e Compliance**                | **IAM Roles e Policies**         | Funções mínimas: Lambda com acesso apenas ao RDS e logs. WebAPI com acesso restrito ao banco.                                                    |
  |                                           | **Criptografia**                 | API Gateway com HTTPS, RDS com encriptação padrão (AES-256).                                                                                     |
  | **Custos e Dimensionamento**              | **Tipos de instância**           | **RDS MySQL:** db.t3.micro <br> **Kubernetes:** EC2 t3.medium <br> **Lambda:** 1M execuções/mês grátis.       |
  |                                           | **Previsão de custos**           | Mantendo o uso dentro da camada gratuita, custo estimado: **US\$ 0/mês** (exceto tráfego excessivo de saída ou armazenamento além dos limites).  |
  | **Fluxos e Dependências**                 | **Fluxo de autenticação**        | Lambda gera tokens JWT assinados e os retorna ao cliente, validade de 60min. Refresh opcional via endpoint seguro.                               |
  

#### Authentication Function com Lambda

  [Repositório serverless](https://github.com/fiap-soat11/serverless)

#### WebAPI no EKS

  [Repositório webapi](https://github.com/fiap-soat11/webapi)

#### Mysql Database no Amazon RDS
  
  [Repositório database](https://github.com/fiap-soat11/database)

#### IaC com Terraform
  
  [Repositório infra](https://github.com/fiap-soat11/infra)

</details>

### Fase 4

<details>

</details>

### Hackathon

</details>

## Versionamento  
Os repositórios desta organização seguem majoritariamente o versionamento semântico (semver), mas a versão principal corresponde à fase do projeto no momento do commit.  

## Hackaton  


## Integrantes do grupo
- [AdinaildoRibeiro](https://github.com/AdinaildoRibeiro)
- [EderGRocha](https://github.com/EderGRocha)
- [mandstoni](https://github.com/mandstoni)
- [MarioSergio07](https://github.com/MarioSergio07)
- [renaildos](https://github.com/renaildos)  
