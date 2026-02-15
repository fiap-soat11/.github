# FIAP Pós Tech - Arquitetura de Software - Hackathon 
Repositórios do projeto em grupo "Tech Challenge" (Grupo 11 da SOAT11 de 2025) do curso de [Arquitetura de Software da FIAP Pós Tech](https://postech.fiap.com.br/curso/software-architecture/).

## Projeto  

A empresa FIAP X precisa avançar no desenvolvimento de um projeto de processamento de imagens. Em uma rodada de investimentos, a empresa apresentou um projeto simples que processa um vídeo e retorna as imagens dele em um arquivo .zip. 
Os investidores gostaram tanto do projeto, que querem investir em uma versão onde eles possam enviar um vídeo e fazer download deste zip. 

O objetivo do projeto é aplicar os tópicos relacionados à arquitetura de software para atender os requistos propostos:

 - A nova versão do sistema deve processar mais de um vídeo ao mesmo tempo 
 - Em caso de picos, o sistema não deve perder uma requisição 
 - O Sistema deve ser protegido por usuário e senha 
 - O fluxo deve ter uma listagem de status dos vídeos de um usuário 
 - Em caso de erro, um usuário pode ser notificado (e-mail ou outro meio de comunicação)

## Arquitetura

### HLD High-Level Design

![HLD](/hack/hld.png)

O diagrama de **High-Level Design (HLD)** apresenta uma visão arquitetural simplificada do sistema FIAP X hospedado na AWS. A arquitetura utiliza uma **Virtual Private Cloud (VPC)** para isolar os recursos de rede, garantindo segurança e controle sobre o tráfego.

**Componentes principais:**
- **API Gateway**: Atua como ponto de entrada único para todas as requisições HTTP vindas de clientes externos, gerenciando o tráfego e fornecendo uma camada de abstração
- **Amazon EKS (Elastic Kubernetes Service)**: Orquestra os containers da WebAPI, permitindo escalabilidade horizontal e alta disponibilidade para processar múltiplos vídeos simultaneamente
- **Amazon RDS MySQL**: Banco de dados relacional gerenciado que armazena informações de usuários, status de processamento de vídeos e metadados do sistema
- **Private Subnet**: Sub-rede privada que contém a WebAPI e o banco de dados, impedindo acesso direto da internet e aumentando a segurança da aplicação

O fluxo de dados segue o padrão: Cliente → API Gateway → EKS (WebAPI) → RDS MySQL, garantindo um processamento seguro e escalável das requisições.

### LLD Low-Level Design

![LLD](/hack/lld.png)

O diagrama de **Low-Level Design (LLD)** detalha a implementação técnica da infraestrutura AWS, expandindo o HLD com especificações concretas de recursos e configurações.

**Componentes detalhados:**
- **API Gateway REST API**: Implementação específica usando o tipo REST API do AWS API Gateway, responsável pelo roteamento de requisições
- **Amazon EKS Cluster**: Ambiente Kubernetes gerenciado executando a aplicação WebAPI em containers, com suporte a auto-scaling para atender picos de demanda
- **RDS MySQL Instance**: 
  - Engine: MySQL 8.0
  - Classe de instância: db.t3.micro (otimizada para workloads de desenvolvimento)
  - Configuração adequada para armazenar dados de usuários e status de processamento
- **Amazon ECR (Elastic Container Registry)**: Repositório de imagens Docker contendo:
  - `fiapx-usuarios`: Imagem do microsserviço de gestão de usuários
  - `fiapx-videos`: Imagem do microsserviço de interação com o frontend para upload e download.
  - `fiapx-processamento`: Imagem do microsserviço de processamento de vídeos
- **VPC com CIDR 172.31.0.0/16**: Rede privada virtual isolada com endereçamento IP privado
- **Private Subnet**: Segmento de rede sem acesso direto à internet, protegendo recursos críticos

A arquitetura LLD demonstra uma implementação baseada em microsserviços, onde cada funcionalidade (usuários e vídeos) é containerizada separadamente, promovendo desacoplamento e facilitando manutenção e escalabilidade independente.

### Arquitetura do Software

![DAS](/hack/DAS.png)

O diagrama de **Arquitetura do Software (DAS)** ilustra a organização interna da aplicação seguindo os princípios de **Clean Architecture** e **Arquitetura Hexagonal (Ports and Adapters)**.

**Camadas da arquitetura:**

1. **WebAPI** (Camada de Apresentação - Azul):
   - Ponto de entrada da aplicação, expondo endpoints REST para comunicação com clientes
   - Recebe requisições HTTP e delega o processamento para camadas internas
   - Responsável pela serialização/desserialização de dados e validações de entrada

2. **Adapters** (Camada de Adaptadores - Verde):
   - Implementa os portos de entrada e saída da arquitetura hexagonal
   - Converte dados entre formatos externos e o domínio da aplicação
   - Isola a lógica de negócios de dependências externas (databases, APIs, etc.)

3. **Application** (Camada de Aplicação - Laranja):
   - Orquestra casos de uso e regras de negócio da aplicação
   - Implementa services e handlers que coordenam operações entre diferentes componentes
   - Gerencia transações e fluxos de trabalho complexos

4. **Domain** (Camada de Domínio - Amarelo):
   - Núcleo da aplicação contendo as entidades de negócio e regras de domínio puras
   - Independente de frameworks e tecnologias externas
   - Representa conceitos do negócio (Vídeo, Usuário, Status de Processamento, etc.)

5. **Datasource** (Camada de Infraestrutura de Dados - Azul):
   - Implementa a persistência de dados
   - Gerencia conexões com bancos de dados (MySQL, S3 para armazenamento de arquivos)
   - Implementa repositórios e data mappers

6. **MySQL Database**: Banco de dados relacional para armazenamento persistente

**Fluxo de dependências:**
O fluxo segue a regra de dependência da Clean Architecture: camadas externas dependem de camadas internas, mas nunca o contrário. O Domain é completamente independente, garantindo testabilidade e manutenibilidade.

### Arquitetura de Dados

O modelo de dados do sistema FIAP X é projetado para gerenciar usuários e o processamento de vídeos de forma eficiente e escalável. O banco de dados MySQL armazena informações essenciais para autenticação, rastreamento de processamento e armazenamento de metadados.

#### Modelo Entidade-Relacionamento

```mermaid
erDiagram
    users ||--o{ video_processings : "submits"
    
    users {
        int id PK "Auto increment"
        varchar(255) name "User full name"
        varchar(255) email UK "Unique email"
        varchar(255) password_hash "Hashed password"
        timestamp created_at "Record creation timestamp"
        timestamp updated_at "Last update timestamp"
    }
    
    video_processings {
        int id PK "Auto increment"
        int user_id FK "References users(id)"
        varchar(255) original_file_name "Original video filename"
        enum status "Pending, Processing, Completed, Failed"
        varchar(2048) s3_input_path "S3 path for input video"
        varchar(2048) s3_output_path "S3 path for output ZIP"
        text failure_reason "Error description if failed"
        timestamp created_at "Processing request timestamp"
        timestamp completed_at "Processing completion timestamp"
    }
```

#### Descrição das Entidades

**Tabela `users`** - Gerenciamento de Usuários:
- Armazena informações de autenticação e identificação dos usuários do sistema
- **id**: Chave primária com auto-incremento para identificação única
- **name**: Nome completo do usuário
- **email**: Email único para login e notificações (constraint UNIQUE)
- **password_hash**: Senha criptografada usando algoritmos de hash seguros (bcrypt, Argon2, etc.)
- **created_at/updated_at**: Timestamps automáticos para auditoria de criação e modificação de registros

**Tabela `video_processings`** - Processamento de Vídeos:
- Rastreia cada requisição de processamento de vídeo com seu estado e resultados
- **id**: Chave primária com auto-incremento
- **user_id**: Chave estrangeira referenciando `users(id)` com `ON DELETE CASCADE` (quando um usuário é deletado, todos seus processamentos são removidos)
- **original_file_name**: Nome original do arquivo enviado pelo usuário
- **status**: Estado atual do processamento com 4 possíveis valores:
  - `Pending`: Aguardando processamento na fila
  - `Processing`: Sendo processado no momento
  - `Completed`: Processamento concluído com sucesso
  - `Failed`: Falha durante o processamento
- **s3_input_path**: Caminho completo no Amazon S3 onde o vídeo original foi armazenado
- **s3_output_path**: Caminho no S3 onde o arquivo ZIP com frames extraídos está armazenado (NULL até conclusão)
- **failure_reason**: Descrição detalhada do erro caso o status seja `Failed`
- **created_at**: Timestamp de quando a requisição foi criada
- **completed_at**: Timestamp de conclusão do processamento (NULL enquanto não finalizado)

#### Relacionamentos

- **users → video_processings**: Relacionamento **1:N** (um usuário pode submeter múltiplos processamentos)
- A integridade referencial é garantida pela constraint `FOREIGN KEY` com `ON DELETE CASCADE`
- Permite listagem de todos os vídeos de um usuário e rastreamento completo do histórico de processamento

#### Estratégia de Armazenamento

O modelo adota uma abordagem híbrida:
- **Metadados no MySQL**: Informações estruturadas para consultas rápidas e relacionamentos
- **Arquivos no Amazon S3**: Vídeos originais e ZIPs de saída armazenados em object storage escalável
- Paths do S3 armazenados como strings longas (VARCHAR 2048) para suportar estruturas de diretórios complexas

Esta arquitetura suporta os requisitos do projeto:
- ✅ Processamento múltiplo: Múltiplos registros `video_processings` com status independentes
- ✅ Proteção por usuário/senha: Tabela `users` com autenticação
- ✅ Listagem de status: Query por `user_id` para visualizar todos os processamentos
- ✅ Notificação de erros: Campo `failure_reason` armazena detalhes para envio de alertas

### Fluxo da informação


```mermaid
flowchart TD
   Start([Cliente inicia requisição]) --> Auth{Autenticado?}
   Auth -->|Não| Login[POST /auth/login<br/>email + password]
   Login --> ValidaCred{Credenciais<br/>válidas?}
   ValidaCred -->|Não| ErrAuth[Retorna 401 Unauthorized]
   ValidaCred -->|Sim| GeraToken[Gera JWT Token]
   GeraToken --> AuthOK
   Auth -->|Sim| AuthOK[Token JWT válido]
   
   AuthOK --> Acao{Escolhe ação}
   
   Acao -->|Upload| Upload[POST /videos/upload<br/>Envia arquivo de vídeo]
   Upload --> SaveS3[Salva vídeo no S3<br/>s3_input_path]
   SaveS3 --> CreateRecord[Cria registro em<br/>video_processings<br/>status: Pending]
   CreateRecord --> Queue[Envia para fila<br/>de processamento]
   Queue --> ReturnUpload[Retorna 202 Accepted<br/>com video_id]
   
   Acao -->|Listar| List[GET /videos<br/>Lista vídeos do usuário]
   List --> QueryDB[Query: SELECT * FROM<br/>video_processings<br/>WHERE user_id = ?]
   QueryDB --> ReturnList[Retorna JSON com lista<br/>id, status, created_at, etc]
   
   Acao -->|Download| Download[GET /videos/:id/download]
   Download --> CheckStatus{Status =<br/>Completed?}
   CheckStatus -->|Não| ErrNotReady[Retorna 400<br/>Vídeo não processado]
   CheckStatus -->|Sim| GetS3[Obtém arquivo do S3<br/>usando s3_output_path]
   GetS3 --> ReturnZip[Retorna arquivo ZIP<br/>com frames extraídos]
   
   Queue --> Worker[Worker processa vídeo]
   Worker --> UpdateProcessing[UPDATE status = Processing]
   UpdateProcessing --> ExtractFrames[Extrai frames do vídeo]
   ExtractFrames --> CreateZip[Cria arquivo ZIP<br/>com imagens]
   CreateZip --> UploadZip[Upload ZIP para S3<br/>salva s3_output_path]
   UploadZip --> Success{Sucesso?}
   
   Success -->|Sim| UpdateCompleted[UPDATE status = Completed<br/>completed_at = NOW]
   Success -->|Não| UpdateFailed[UPDATE status = Failed<br/>failure_reason = erro]
   UpdateFailed --> Notify[Envia notificação<br/>de erro ao usuário]
   
   UpdateCompleted --> End([Fim])
   ReturnUpload --> End
   ReturnList --> End
   ReturnZip --> End
   ErrAuth --> End
   ErrNotReady --> End
   Notify --> End
   
   style Start fill:#90EE90
   style End fill:#FFB6C1
   style UpdateCompleted fill:#90EE90
   style UpdateFailed fill:#FF6B6B
   style ErrAuth fill:#FF6B6B
   style ErrNotReady fill:#FF6B6B
```

#### Descrição do Fluxo

**1. Autenticação:**
- Cliente tenta acessar o sistema
- Se não autenticado, realiza login com email/senha
- Sistema valida credenciais na tabela `users`
- Gera token JWT para autenticação subsequente

**2. Upload de Vídeo:**
- Cliente envia vídeo via POST /videos/upload
- Sistema salva arquivo no S3 (s3_input_path)
- Cria registro em `video_processings` com status `Pending`
- Adiciona job na fila de processamento
- Retorna imediatamente com HTTP 202 (requisição aceita mas não processada)

**3. Processamento Assíncrono:**
- Worker retira job da fila
- Atualiza status para `Processing`
- Extrai frames do vídeo
- Cria arquivo ZIP com as imagens
- Faz upload do ZIP para S3 (s3_output_path)
- Atualiza status para `Completed` ou `Failed`
- Em caso de falha, envia notificação ao usuário

**4. Listagem de Status:**
- Cliente consulta GET /videos
- Sistema busca todos os registros do usuário em `video_processings`
- Retorna lista com status atual de cada processamento

**5. Download do Resultado:**
- Cliente solicita GET /videos/:id/download
- Sistema verifica se status = `Completed`
- Se sim, busca arquivo ZIP do S3 e retorna
- Se não, retorna erro 400 informando que processamento não finalizou

Este fluxo garante processamento assíncrono de múltiplos vídeos, tolerância a picos de demanda via fila, 
e rastreamento completo do status de cada processamento.

LOGICA DE NEGÓCIO COMPLETA – PROJETO FIAP X (PROCESSAMENTO DE VÍDEO)

1. ENTIDADES DO DOMÍNIO + REGRAS

📌 Entidade: User
Atributos
- id
- name
- email
- passwordHash
- createdAt
- updatedAt

Regras de Domínio
- email deve ser único
- senha nunca pode ser salva sem hash
- usuário só acessa seus próprios vídeos
- ao deletar usuário, todos os vídeos dele devem ser removidos (ON DELETE CASCADE)

📌 Entidade: VideoProcessing
Atributos
- id
- userId
- originalFileName
- status (Pending | Processing | Completed | Failed)
- s3InputPath
- s3OutputPath
- failureReason
- createdAt
- completedAt

Regras de Domínio
- status inicial sempre = Pending
- se status = Completed → s3OutputPath não pode ser null
- se status = Failed → failureReason deve ter texto
- completedAt só pode ser preenchida se finalizado (completed ou failed)
- nunca reprocessar um vídeo Completed sem criar um novo registro

🟧 2. CASOS DE USO (USE CASES)
AuthenticateUser
UploadVideo
StartVideoProcessingJob (worker)
ProcessVideoFrames (worker)
GenerateZipFile (worker)
MarkProcessingCompleted
MarkProcessingFailed
ListUserVideos
DownloadProcessedZip

🟧 3. REGRAS DE NEGÓCIO POR CASO DE USO
1️⃣ AuthenticateUser (Login)
* Entrada: email, password
* Saída: JWT token

Regras
- email deve existir
- password → comparar hash
- se falhar, retornar erro de credenciais
- gerar JWT
- token expira (ex: 1h)

2️⃣ UploadVideo (POST /videos/upload)
* Entrada: arquivo de vídeo, userId
* Saída: id do processamento, status = Pending

Regras
- aceitar apenas formatos permitidos (mp4, mov, avi)
- enviar vídeo para S3 → gerar s3InputPath
- criar registro VideoProcessing com:
  status = Pending
  s3OutputPath = null
  failureReason = null
- adicionar job na fila (ex.: SQS/Kafka)
- retornar HTTP 202 (Accepted)

3️⃣ StartVideoProcessingJob (Worker – início do job)
* Entrada: videoProcessingId

Regras
- validar se status atual é Pending
- atualizar status para Processing
- atualizar timestamp de início (opcional)
- chamar processamento real

4️⃣ ProcessVideoFrames (Worker – extrair frames)**

Regras
- baixar arquivo de vídeo do S3 (s3InputPath)
- extrair frames
- salvar frames em diretório temporário
- se falhar → emitir evento “ProcessingFailed”

5️⃣ GenerateZipFile (Worker – compactar frames)**

Regras
- zipar todas as imagens
- enviar ZIP final ao S3
- gerar s3OutputPath
- se falhar → emitir evento “ProcessingFailed”

6️⃣ MarkProcessingCompleted
* Entrada: videoProcessingId, s3OutputPath

Regras
- status = Completed
- completedAt = agora
- failureReason = null

7️⃣ MarkProcessingFailed
* Entrada: videoProcessingId, reason

Regras
- status = Failed
- failureReason obrigatório
- completedAt = agora
- enviar notificação ao usuário (email)

8️⃣ ListUserVideos (GET /videos)
* Entrada: userId
* Saída: lista de VideoProcessing

Regras
- usuário só pode listar os vídeos dele
- retorno deve incluir status atual
- vídeos ordenados por data

9️⃣ DownloadProcessedZip (GET /videos/:id/download)
* Entrada: videoProcessingId, userId

Regras
- registro deve existir
- registro deve ser do usuário
- status deve ser Completed
- se não for Completed → erro “Processing not finished”
- baixar arquivo do S3 e retornar

🟦 4. FLUXO COMPLETO DE NEGÓCIO
📌 Fluxo 1 – Login
- usuário envia email/senha
- sistema valida credenciais
- se OK → gera JWT
- usuário usa JWT nas próximas requisições

📌 Fluxo 2 – Upload de vídeo
- cliente envia vídeo
- WebAPI valida extensão
- salva arquivo no S3 → gera s3InputPath
- cria registro em video_processings com status Pending
- envia job para fila
- retorna 202 para o cliente

📌 Fluxo 3 – Processamento assíncrono
- worker lê job
- muda status para Processing
- baixa vídeo do S3
- extrai frames
- gera ZIP
- envia ZIP ao S3 → s3OutputPath
- salva status Completed
- se erro:
  - status Failed
  - failureReason preenchido
  - notificar usuário

📌 Fluxo 4 – Listar vídeos
- usuário chama GET /videos
- sistema busca todos registros onde userId = usuário
- retorna lista + status

📌 Fluxo 5 – Baixar ZIP
- usuário chama /download
- sistema verifica:
  - usuário dono?
  - status Completed?
- se OK → baixa ZIP do S3
- retorna arquivo

🟥 5. ERROS DE NEGÓCIO (DOMAIN ERRORS)
- UserNotFound
- InvalidCredentials
- UnsupportedFileFormat
- VideoNotFound
- VideoNotBelongsToUser
- ProcessingNotFinished
- ProcessingAlreadyFinished
- StorageError
- NotificationError

🟩 6. EVENTOS DE DOMÍNIO
- VideoUploaded
- ProcessingStarted
- FramesExtracted
- ZipGenerated
- ProcessingCompleted
- ProcessingFailed

🟪 7. RESUMO – LÓGICA DE NEGÓCIO
. Lógica de Negócio – FIAP X Video Processing
. Usuários devem autenticar com email/senha para acessar qualquer recurso.
. O upload de vídeo gera um registro de processamento com status inicial Pending.
. Cada vídeo é processado de forma assíncrona, permitindo múltiplos processamentos simultâneos.
. Um worker atualiza o status conforme o vídeo progride:
. Pending → Processing → Completed ou Failed
. O resultado do processamento (frames ZIP) é armazenado no S3 e vinculado ao registro.
. O usuário pode listar todos os processamentos e seus respectivos status.
. O download só é permitido se o status for Completed.
. Em caso de falha, o usuário deve ser notificado e o motivo registrado.

Core Business Logic & Architectural Constraints
Esta seção descreve as diretrizes fundamentais que regem o comportamento do ecossistema FIAP X, 
garantindo a integridade dos dados, a segurança multi-inquilino (multi-tenancy) e a resiliência operacional.

1. Segurança e Governança de Acesso
- Identidade e Proteção (IAM): O acesso a qualquer recurso da plataforma é estritamente condicionado à autenticação via JWT (JSON Web Token).
  Credenciais sensíveis (senhas) devem ser persistidas utilizando algoritmos de hashing com salt (ex: BCrypt ou Argon2) para mitigar ataques de dicionário.
- Isolamento de Dados (Tenant Isolation): O sistema opera sob uma política rigorosa de Ownership. Um usuário autenticado possui visibilidade e autoridade 
  exclusivamente sobre seus próprios registros e arquivos. Tentativas de acesso a video_id de terceiros devem ser interceptadas e resultar em 403 Forbidden.

2. Ciclo de Vida e Máquina de Estados (State Machine)
O processamento de vídeo é tratado como uma transação de longa duração, governada por uma máquina de estados finitos para garantir a consistência eventual:
- Pending: Estado inicial. O artefato foi recebido no Object Storage (S3) e o evento de processamento foi enfileirado.
- Processing: O Worker assumiu a custódia do job. Neste estágio, o recurso está bloqueado para novas tentativas de processamento simultâneo.
- Completed: Estado terminal de sucesso. O artefato de saída (ZIP) está disponível e o completed_at é registrado.
- Failed: Estado terminal de erro. O motivo da falha (Reason) deve ser persistido para fins de auditoria e feedback ao usuário.

3. Resiliência e Ingestão de Carga
- Desacoplamento por Mensageria: Para suportar picos de demanda sem perda de requisições, a WebAPI não processa vídeos diretamente.
  Ela atua apenas como produtor de eventos. A carga de trabalho é delegada a Workers assíncronos, garantindo que a API permaneça responsiva sob pressão.
- Backpressure & Scalability: O sistema utiliza os princípios de Horizontal Pod Autoscaling (HPA) no Kubernetes. A capacidade de processamento deve escalar
  linearmente conforme o volume da fila de mensagens aumenta.

4. Estratégia de Persistência Híbrida
- Metadados Relacionais: Informações estruturadas, relações de propriedade e logs de status são armazenados no RDS MySQL, garantindo conformidade ACID para transações de estado.
- Unstructured Blob Storage: Vídeos originais e arquivos ZIP finais são armazenados no Amazon S3. A base de dados armazena apenas os S3 Paths (URIs), evitando o overhead de
- armazenamento de binários no banco de dados relacional.

5. Notificação e Observabilidade
- Fluxo de Notificação Proativo: Em cenários de falha técnica ou de violação de formato de arquivo, o sistema deve disparar um evento de notificação (via SMTP ou Webhook) para
  informar o usuário final, reduzindo a fricção e o suporte manual.
- Integridade de Download: O acesso ao arquivo processado é condicionado ao estado Completed. Solicitações de download para estados intermediários devem ser rejeitadas para
  evitar inconsistências de arquivo corrompido ou incompleto.

📂 Definição de Endpoints (Technical Specs)
Seguindo o padrão de Arquitetura Hexagonal, a camada de Adaptores de Entrada expõe os seguintes contratos:

Auth Interface
- POST /auth/register: Criação de identidade.
- POST /auth/login: Troca de credenciais por Token de acesso.

Video Management (Async Context)
- POST /videos/upload: Ingestão de binário. Retorna 202 Accepted.
  - Constraint: Apenas formatos suportados (ex: .mp4, .mkv) são aceitos.
- GET /videos: Query de telemetria. Retorna a lista de processamentos e seus respectivos status atuais.
- GET /videos/:id/download: Recuperação de artefato.
  - Logic: Se status != Completed, retorna 400 Bad Request. Se user_id != owner, retorna 403 Forbidden.

🛠️ Procedimento de Execução (Architect's View)
A infraestrutura é tratada como código (IaC). Para replicar o ambiente:
- Provisionamento da Infra: Certifique-se de que o Cluster EKS e a instância RDS estão acessíveis via VPC.
- Container Registry: As imagens devem ser buildadas e enviadas ao Amazon ECR:
  . docker build -t fiapx/webapi .
  . docker push [aws_account_id].dkr.ecr.[region].amazonaws.com/fiapx-webapi
- Deployment via Helm/Kubectl:
  . kubectl apply -f ./k8s/secrets.yaml
  . kubectl apply -f ./k8s/deployment.yaml

