@startuml
!include https://raw.githubusercontent.com/plantuml-stdlib/C4-PlantUML/master/C4_Container.puml

LAYOUT_WITH_LEGEND()

title Diagrama de Container C4 (Nível 2) Otimizado \n Sistema de Gerenciamento de Demandas (SGD) - TJGO

' ===== ATORES (PESSOAS) =====
Person(servidor, "Servidor do Tribunal", "Funcionário que registra e acompanha chamados de TI (Usuário Final)")
Person(tecnico, "Técnico de TI", "Profissional que atende, diagnostica e resolve demandas técnicas (Tier 1/2)")
Person(gestor_ti, "Gestor de TI", "Gerente que define prioridades, aloca recursos e monitora performance da equipe")
Person(corregedor, "Corregedor", "Autoridade que visualiza KPIs estratégicos de tempo de atendimento (TMA/TME)")

' ===== BOUNDARY DO SISTEMA =====
System_Boundary(sgd, "Sistema de Gerenciamento de Demandas (SGD) - Plataforma K8s/Docker") {

    ' Container 1: Frontend
    Container(spa, "Aplicação Web Responsiva (SPA)", "React 18 + TypeScript + Material-UI", "Interface para registro, acompanhamento e gestão de chamados. Servida via Nginx/CDN.")

    ' Container 2: Backend API (Core do Negócio)
    Container(api, "API Gateway/REST (Core)", "Node.js 20 + Express + TypeScript", "Valida requisições, implementa regras de negócio (criação/atualização de chamados) e calcula SLAs.")

    ' Container 3: Banco de Dados
    ContainerDb(db, "Banco de Dados Transacional", "PostgreSQL 15 (Managed Service)", "Armazena dados estruturados: chamados, usuários, históricos de auditoria (RNF05) e metadados dos anexos.")

    ' Container 4: Scheduler (Serviço Cron)
    Container(scheduler, "Serviço de Agendamento (Job Runner)", "Node.js + node-cron/Agenda", "Executa tarefas críticas: **Escalonamento** de SLAs próximos do vencimento e geração programada de relatórios (BI).")

    ' Container 5: Notification Service (Microsserviço Assíncrono)
    Container(notification, "Serviço de Processamento de Notificações", "Node.js Consumer + RabbitMQ Client", "Consome a fila, formata o conteúdo dos e-mails (templates) e os envia. Garante Retry e DLQ.")

    ' Container 6: Message Broker
    ContainerQueue(broker, "Fila de Mensagens (Message Broker)", "RabbitMQ 3.12 (Cluster)", "Ponto central para desacoplamento. Armazena eventos (Payloads) antes que sejam processados de forma assíncrona pelo Notification Service.")

    ' Container 7: File Storage (Armazenamento de Binários)
    Container(file_storage, "Armazenamento Persistente de Arquivos", "Volume NFS Compartilhado / PVC K8s", "Armazena exclusivamente os binários dos anexos (imagens, logs, PDFs). Gerenciado por sistema de arquivos.")

}

' ===== SISTEMAS EXTERNOS =====
System_Ext(auth_system, "Sistema de Autenticação Central", "Active Directory / LDAP corporativo TJGO", "Fornece autenticação (SSO) e autorização (perfis de acesso).")
System_Ext(email_system, "Servidor de E-mail Corporativo", "Servidor SMTP (Exchange)", "Ponto de saída final (relé) para o envio de e-mails para os usuários.")

' ===== RELACIONAMENTOS: ATORES → SISTEMA =====
Rel(servidor, spa, "Registra/Acompanha chamados", "HTTPS")
Rel(tecnico, spa, "Atende/Resolve demandas", "HTTPS")
Rel(gestor_ti, spa, "Gerencia filas/Monitora equipe", "HTTPS")
Rel(corregedor, spa, "Visualiza Dashboards Estratégicos", "HTTPS")

' ===== RELACIONAMENTOS: CONTAINERS INTERNOS (Fluxo de Dados Otimizado) =====
Rel(spa, api, "Requisições CRUD de chamados", "HTTPS/JSON")
Rel(api, db, "Consultas e Transações de Dados", "TCP/SQL (Prisma ORM)")

' --- Fluxo de Notificações (API publica, Broker armazena, Notification consome) ---
Rel(api, broker, "Publica eventos assíncronos (chamado criado, SLA expirando)", "AMQP 0-9-1")
Rel(scheduler, broker, "Publica eventos de alerta/relatório (SLA vencido, relatórios diários)", "AMQP 0-9-1")
Rel(notification, broker, "Consome mensagens da fila (eventos)", "AMQP 0-9-1")
Rel(notification, email_system, "Envia e-mails formatados", "SMTP/TLS")

' --- Fluxo de Arquivos (API como Proxy Único e Ponto de Autorização) ---
Rel(api, file_storage, "Armazena/Recupera arquivos binários via path", "NFS / Volume Protocol")
Rel(spa, api, "Upload/Download de anexos", "HTTPS/Multipart-Form")

' --- Relacionamentos de Suporte/Configuração ---
Rel(scheduler, db, "Consulta dados para verificar SLAs", "TCP/SQL (Prisma ORM)")
Rel(api, auth_system, "Valida token e consulta perfis de usuário (SSO)", "LDAP v3 / LDAPS")

' ===== NOTAS ARQUITETURAIS PARA MAIOR CLAREZA =====
note right of broker
**Separação:**
O Broker (RabbitMQ) agora é um contêiner explícito, separando a **Mensageria** do **Processamento** (Notification Service).
end note

note right of file_storage
**Anexos:**
Max. 10MB/arquivo.
A API gerencia o **caminho** no DB (metadado) e o **acesso físico** ao volume.
end note

note top of api
**Design:**
Pode ser futuramente subdividido em microsserviços por domínio (Ex: Chamados, Usuários, SLAs).
end note
@enduml