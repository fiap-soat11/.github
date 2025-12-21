# FIAP Pós Tech - Arquitetura de Software - Tech Challenge  
Repositórios do projeto em grupo "Tech Challenge" (Grupo 163 da SOAT11 de 2025) do curso de [Arquitetura de Software da FIAP Pós Tech](https://postech.fiap.com.br/curso/software-architecture/).

## Projeto  
O objetivo do projeto foi estudar tópicos relacionados à arquitetura de software por meio da criação de um sistema de autoatendimento de fast-food e gerenciamento da cozinha para uma lanchonete fictícia.

O cliente pode se identificar; realizar o checkout de um pedido; confirmá-lo; pagá-lo através de um serviço de pagamento externo, e o pedido pode então ser processado pelos funcionários da cozinha.
A lanchonete pode manter cadastro de clientes; gerenciar produtos e categorias; acompanhar os pedidos.

## Fases  

O projeto, assim como o curso, é dividido em várias 'fases', cada uma abordando tópicos relacionados à arquitetura de software.  
Alguns desses tópicos exigem reescrita de código, refatorações e divisão dos repositórios.


  <summary>Documentação de cada fase</summary>

### Fase 1




  - Aplicação monolítica .Net Core em Arquitetura Hexagonal com Banco de Dados Mysql  
  
  - [Event Storming (DDD)](https://miro.com/app/board/uXjVIG95Hyw=/)
    
  - [Linguagem Ubíqua](https://github.com/fiap-soat11/.github/wiki/Linguagem-Ub%C3%ADqua)
  
  - [Arquitetura Hexagonal (Ports and Adapters)](https://github.com/fiap-soat11/.github/wiki)
    
    ![DAS](/fase1/docs/DAS.png)
    
  - [Modelo de entidade relacional](https://github.com/fiap-soat11/.github/wiki/Modelo-de-dados)
  
    ![MER](/fase1/docs/MER.png)

  - [Plano de Testes](/fase1/docs/plano-de-testes.md)

 

### Fase 2



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


 

### Fase 3



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


### Banco Relacional e MySql

**Justificativa para Escolha de Banco de Dados Relacional**

Optamos pelo uso de um banco de dados relacional, mais especificamente o MySQL, no projeto da Lanchonete FIAP por entender que este modelo é o mais alinhado às necessidades operacionais e aos objetivos do negócio.

**Estruturação e Integridade dos Dados**

O sistema da lanchonete precisa lidar com informações altamente estruturadas, como clientes, pedidos, pagamentos, produtos e ingredientes. Existe uma forte dependência entre essas entidades, que se relacionam de formas diversas (ex: um pedido pode conter múltiplos produtos; um produto, por sua vez, pode ter vários ingredientes). O banco de dados relacional permite estabelecer essas conexões de maneira precisa, por meio de chaves primárias e estrangeiras, assegurando consistência e integridade em todas as transações.

**Facilidade nas Operações e Consultas Complexas**

Para o dia a dia do negócio, é essencial obter rapidamente informações como histórico de pedidos de um cliente, status atual de cada pedido, composição dos produtos e controle dos pagamentos realizados. O modelo relacional suporta operações de consulta complexas (como consultas com múltiplos filtros e junções entre tabelas), facilitando a extração de dados relevantes para tomada de decisão e relatórios operacionais.

**Suporte à Evolução do Negócio**

À medida que a lanchonete cresce, novas funcionalidades podem ser implementadas facilmente. O modelo relacional é flexível e robusto para aceitar expansão de tabelas, inclusão de novos atributos ou mesmo ajustes em relações, minimizando riscos de inconsistência e sustentando o crescimento do sistema sem prejudicar o legado.

**Controle e Auditoria**

O uso de um banco relacional favorece o controle rigoroso sobre as informações cadastradas no sistema e permite rastrear alterações relevantes, impactando diretamente na rastreabilidade e na segurança da operação. Isso é fundamental para manter a confiabilidade das informações e atender eventuais exigências regulatórias ou de auditoria.

**Padrão de Mercado e Suporte Tecnológico**

A escolha pelo MySQL também se justifica pela sua ampla utilização em sistemas transacionais, além de disponibilizar vasto suporte da comunidade, documentação, ferramentas e profissionais experientes. Trata-se de uma solução open source, com alta confiabilidade, desempenho eficiente e excelente custo-benefício, facilitando ainda futuras integrações com outros sistemas já consolidados no mercado.

**Conclusão:**

A escolha pelo banco de dados relacional MySQL está fundamentada na necessidade de garantir organização, eficiência, segurança e escalabilidade para o sistema da lanchonete, alinhando o ambiente tecnológico com os desafios e oportunidades do negócio.



### Fase 4 - Microsservicos


  **Arquitetura**

  ![Arquitetura](/fase4/arch.png)

  **Microsserviços Implementados:**

  1. **Cliente Service**
    - Gerencia o cadastro e autenticação de clientes
    - Expõe endpoints para operações CRUD de clientes
    - Integra-se com o DynamoDB para persistência de dados

  2. **Pedido Service**
    - Responsável pela criação e gerenciamento de pedidos
    - Controla o fluxo desde a seleção de produtos até a confirmação
    - Comunica-se com outros serviços via mensageria

  3. **Preparo Service**
    - Gerencia a fila de pedidos na cozinha
    - Controla os status: Recebido, Em Preparação, Pronto
    - Notifica outros serviços sobre mudanças de estado

  4. **Pagamento Service**
    - Processa pagamentos através de gateway externo
    - Valida e confirma transações financeiras
    - Atualiza status do pedido após confirmação

  **Comunicação entre Serviços:**

  - **Síncrona:** Via API Gateway e chamadas REST entre microsserviços
  - **Assíncrona:** Utilização de RabbitMQ (message broker) executado no Kubernetes para desacoplamento e resiliência
  - **Event-Driven:** Arquitetura orientada a eventos para propagação de mudanças de estado através de filas e exchanges do RabbitMQ

  **Benefícios da Arquitetura:**

  ✅ Escalabilidade independente por serviço  
  ✅ Deploy isolado sem impacto em outros componentes  
  ✅ Resiliência através de circuit breakers e fallbacks  
  ✅ Manutenibilidade facilitada por bounded contexts claros  
  ✅ Flexibilidade tecnológica (cada serviço pode usar stack diferente)

---

  **Infra**

  ![AWS](/fase4/infra.png)

  A infraestrutura foi projetada para garantir alta disponibilidade, escalabilidade e segurança, utilizando serviços gerenciados da AWS e seguindo as melhores práticas de arquitetura de microsserviços.

  **Componentes Principais:**

  **1. Rede e Conectividade**
  - VPC isolada (172.31.0.0/16) com sub-redes públicas e privadas distribuídas em múltiplas zonas de disponibilidade

  **2. Camada de Entrada**
  - **API Gateway** como único ponto de entrada para requisições externas
  - Integração com Lambda Functions para autenticação e operações de cliente
  - Proxy HTTP configurado para rotear requisições ao cluster EKS
  - Autenticação baseada em tokens JWT

  **3. Camada de Computação**
  - **Lambda Functions** para operações serverless (autenticação e cliente)
  - **EKS (Elastic Kubernetes Service)** gerenciando cluster de microsserviços
  - **EC2 Node Group** com instâncias t3.medium para workers do Kubernetes
  - Auto Scaling configurado para ajustar capacidade conforme demanda

  **4. Camada de Persistência**
  - **DynamoDB** para armazenamento NoSQL de clientes e pedidos
  - **RDS MySQL 8.0** (db.t3.micro) para dados relacionais em Multi-AZ
  - **Persistent Volumes** no Kubernetes para dados de aplicação

  **5. Registro e Imagens**
  - **ECR (Elastic Container Registry)** armazenando imagens Docker dos microsserviços
  - Repositórios separados para cada microsserviço (cliente, pedido, preparo, pagamento)

  **6. Mensageria e Integração**
  - **RabbitMQ** executado como StatefulSet no cluster Kubernetes para comunicação assíncrona entre microsserviços
  - Filas e exchanges dedicados para cada fluxo de eventos
  - Persistent Volume para garantir durabilidade das mensagens
  - Management UI para monitoramento e administração das filas

  **7. Segurança**
  - **IAM Roles** com permissões específicas para cada recurso
  - **Security Groups** controlando tráfego de entrada e saída
  - Recursos críticos em sub-redes privadas sem acesso direto à internet
  - Criptografia em trânsito e em repouso

  **Fluxo de Requisição:**

  1. Cliente externo acessa via API Gateway (HTTPS)
  2. API Gateway valida JWT e roteia para Lambda ou EKS
  3. Lambdas processam autenticação/cliente consultando DynamoDB
  4. Requisições de negócio são direcionadas aos microsserviços no EKS
  5. Microsserviços comunicam-se via RabbitMQ (executado no cluster) e acessam bancos de dados conforme necessário
  6. Respostas retornam pelo mesmo caminho até o cliente
  
#### VPC (Virtual Private Cloud)

- **CIDR:** 172.31.0.0/16  
Toda a infraestrutura está isolada dentro de uma VPC, garantindo segurança na comunicação entre os serviços internos.

---

#### API Gateway

- Serve como principal ponto de entrada HTTP para clientes externos.
- Rotas configuradas:
  - `/auth` → método **POST** → chama **Authentication Lambda**
  - `/cliente` → método **GET** → chama **Client Lambda**
  - `/eks/{proxy+}` → método **ANY** do tipo **HTTP_PROXY**
- Autenticação via header **JWT**
- Permissões configuradas com política **lambda:InvokeFunction**

---

#### Lambdas

Existem duas funções principais:

1. **Authentication Function (Lambda)**  
   - Responsável por autenticação de usuários.
   - Interage com o **DynamoDB** para validação de credenciais e dados.

2. **Client Function (Lambda)**  
   - Manipula lógica de cliente.
   - Acessa base de dados conforme necessário.

---

#### Banco de Dados

A solução utiliza dois bancos distintos:

| Tecnologia | Função | Observações |
|-----------|---------|-------------|
| DynamoDB  | Clientes & Pedidos | Armazenamento NoSQL, escalabilidade automática |
| RDS MySQL | Dados relacionais | engine: `mysql 8.0`, classe `db.t3.micro` |

  **Modelo Relacional**

  ![BD relacional](/fase4/BD_relacional.png)

  **Modelo NoSql**

  ![BD NoSql](/fase4/BD_nosql.png)

---

#### ECR (Elastic Container Registry)

Repositório privado com imagens Docker utilizadas pelos microsserviços executados no cluster Kubernetes.

Repositórios disponíveis:
- `fiap-lambda`
- `fiap-cliente`
- `fiap-pedido`
- `fiap-preparo`
- `fiap-pagamento`

---

#### Cluster Kubernetes – EKS

- Worker nodes configurados em **NodeGroup**
  - Tipo de instância: `t3.medium`
  - Disco: 50GB
  - Mínimo de nós: 1
  - Máximo de nós: 2

**Recursos Implementados**

- **Deployment e Pods** para cada microsserviço
- **Local Balancer (Microserviço)**
- **Autoscaling com base em CPU e memória**
- **StatefulSet** para serviços com estado
- **Volumes persistentes (PVC / PV)** para armazenamento

---

#### RabbitMQ (Message Broker)

**Deployment no Kubernetes**

- Executado como **StatefulSet** para garantir identidade estável dos pods
- Imagem oficial: `rabbitmq:3-management`
- **Portas expostas:**
  - `5672` - Protocolo AMQP para comunicação dos microsserviços
  - `15672` - Management UI para administração

---

#### Microsserviços

- Aplicação executada por meio dos pods do cluster.
- Balanceamento interno para distribuição equitativa de carga.
- Quando necessário, o cluster escala automaticamente.
- Integração com RabbitMQ via bibliotecas AMQP client (.NET: RabbitMQ.Client)

---



### Hackathon



## Versionamento  
Os repositórios desta organização seguem majoritariamente o versionamento semântico (semver), mas a versão principal corresponde à fase do projeto no momento do commit.  

## Hackaton  


## Integrantes do grupo
- [AdinaildoRibeiro](https://github.com/AdinaildoRibeiro)
- [EderGRocha](https://github.com/EderGRocha)
- [mandstoni](https://github.com/mandstoni)
- [MarioSergio07](https://github.com/MarioSergio07)
- [renaildos](https://github.com/renaildos)  
