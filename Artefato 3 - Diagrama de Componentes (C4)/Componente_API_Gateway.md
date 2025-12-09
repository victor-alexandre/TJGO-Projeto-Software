@startuml
!include <C4/C4_Component>

' Layout da Esquerda para Direita: Entrada -> Processamento -> Saída
LAYOUT_LEFT_RIGHT()

title Diagrama de Componentes - API Gateway (Clean Architecture Refactor)

' --- SISTEMAS E CONTAINERS EXTERNOS (Dependências) ---
ContainerDb(db, "PostgreSQL", "RDBMS", "Dados Transacionais")
ContainerDb(filesystem, "Object Storage", "S3/NFS", "Arquivos Físicos")
ContainerQueue(broker, "RabbitMQ", "Exchange/Queue", "Mensageria")
System_Ext(email_sys, "SMTP Server", "Exchange", "Envio de E-mails")
System_Ext(auth_sys, "Identity Provider", "LDAP/AD", "Autenticação Corporativa")

Container_Boundary(api, "API Gateway / Core Application") {

    ' ============================================================
    ' 1. CAMADA DE INTERFACE (Primary Adapters / Drivers)
    ' ============================================================
    ' Responsável apenas por receber a requisição (HTTP), validar o formato (DTO) e chamar a Aplicação.
    Boundary(presentation, "Camada de Interface (Controllers)") {
        Component(security, "Security Middleware", "Filter", "Intercepta requisições, valida Token e RBAC.")

        Component(auth_ctrl, "Auth Controller", "REST", "Endpoint de Login e Refresh.")
        Component(ticket_ctrl, "Ticket Controller", "REST", "Endpoints de Chamados (POST/GET/PUT).")
        Component(admin_ctrl, "Admin Controller", "REST", "Endpoints de Gestão de Técnicos.")
        Component(report_ctrl, "Report Controller", "REST", "Endpoints de Dashboards.")
        Component(file_ctrl, "File Controller", "REST", "Endpoints de Upload/Download.")
    }

    ' ============================================================
    ' 2. CAMADA DE APLICAÇÃO (Application Business Rules)
    ' ============================================================
    ' Orquestra os fluxos. Não contém regras de negócio complexas, apenas manda fazer.
    ' Define as PORTAS (Interfaces) que o sistema precisa.
    Boundary(app_layer, "Camada de Aplicação (Use Cases & Ports)") {

        ' --- CASOS DE USO (Orquestradores) ---
        Component(auth_uc, "Auth UseCase", "App Service", "Orquestra login e geração de sessão.")
        Component(ticket_uc, "Ticket UseCase", "App Service", "Fluxo de abertura, atendimento e fechamento.")
        Component(staff_uc, "Staff UseCase", "App Service", "Fluxo de cadastro de técnicos.")
        Component(report_uc, "Analytics UseCase", "App Service", "Consolida dados para relatórios.")

        ' --- PORTAS DE SAÍDA (Interfaces / Output Ports) ---
        ' A aplicação define o contrato, a infraestrutura implementa (DIP).
        Component(i_repo_ticket, "ITicketRepository", "Interface", "Contrato para persistir chamados.")
        Component(i_repo_staff, "IStaffRepository", "Interface", "Contrato para persistir técnicos.")
        Component(i_notifier, "INotifierPort", "Interface", "Contrato para enviar notificações.")
        Component(i_storage, "IStoragePort", "Interface", "Contrato para salvar arquivos.")
        Component(i_audit, "IAuditRepository", "Interface", "Contrato para logs de auditoria.")
        
        ' --- Interface para Autenticação ---
        Component(i_auth_provider, "IAuthProvider", "Interface", "Contrato para validar credenciais externas.")
    }

    ' ============================================================
    ' 3. CAMADA DE DOMÍNIO (Enterprise Business Rules)
    ' ============================================================
    ' O coração do sistema. Regras puras que independem de tecnologia.
    ' Entidades e Serviços de Domínio.
    Boundary(domain_layer, "Camada de Domínio (Core)") {

        ' --- ENTIDADES (Regras de Estado) ---
        Component(ticket_entity, "Ticket Entity", "Domain Object", "Validações de status e transições.")
        Component(staff_entity, "Staff Entity", "Domain Object", "Regras de perfil e disponibilidade.")

        ' --- SERVIÇOS DE DOMÍNIO (Regras Complexas) ---
        Component(sla_service, "SLA Calculator", "Domain Service", "Cálculo de vencimento (Feriados/Horas).")
        Component(assign_strategy, "Assignment Strategy", "Domain Service", "Algoritmo de distribuição de carga.")
    }

    ' ============================================================
    ' 4. CAMADA DE INFRAESTRUTURA (Secondary Adapters / Driven)
    ' ============================================================
    ' Implementações concretas das interfaces definidas na Aplicação.
    Boundary(infra_layer, "Camada de Infraestrutura (Adapters)") {

        ' Implementações de Repositórios (DB)
            Component(repo_ticket_impl, "Ticket Repo SQL", "Prisma", "Impl ITicketRepository (Persiste e busca chamados no banco).")
            Component(repo_staff_impl, "Staff Repo SQL", "Prisma", "Impl IStaffRepository (Gerencia dados cadastrais dos técnicos).")
            Component(repo_audit_impl, "Audit Repo SQL", "Prisma", "Impl IAuditRepository (Registra logs de ações críticas no banco).")

            ' Implementações de Adaptadores Externos
            Component(rmq_adapter, "RabbitMQ Adapter", "AMQP Client", "Impl INotifierPort (Publica eventos assíncronos na fila).")
            Component(s3_adapter, "Disk/S3 Adapter", "File Driver", "Impl IStoragePort (Gerencia upload/download de anexos em disco).")
            Component(ldap_adapter, "LDAP Adapter", "LDAP Client", "Impl IAuthProvider (Conecta no AD para validar credenciais).")
        }

}

' ============================================================
' RELACIONAMENTOS (O FLUXO DE CONTROLE)
' ============================================================

' 1. Interface -> Aplicação
Rel(security, ticket_ctrl, "Fwd")
Rel(security, auth_ctrl, "Fwd")

Rel(ticket_ctrl, ticket_uc, "Chama (DTO)")
Rel(auth_ctrl, auth_uc, "Chama (DTO)")
Rel(admin_ctrl, staff_uc, "Chama (DTO)")
Rel(report_ctrl, report_uc, "Chama (DTO)")
Rel(file_ctrl, ticket_uc, "Anexa em Chamado")

' 2. Aplicação -> Domínio (Usa a regra)
Rel(ticket_uc, ticket_entity, "Manipula")
Rel(ticket_uc, sla_service, "Solicita Cálculo")
Rel(ticket_uc, assign_strategy, "Solicita Distribuição")
Rel(staff_uc, staff_entity, "Cria/Edita")

' Domínio conversa entre si
Rel(sla_service, ticket_entity, "Lê dados")

' 3. Aplicação -> Portas (Interfaces)
Rel(ticket_uc, i_repo_ticket, "Usa")
Rel(ticket_uc, i_notifier, "Usa")
Rel(ticket_uc, i_audit, "Usa")
Rel(ticket_uc, i_storage, "Usa")
Rel(staff_uc, i_repo_staff, "Usa")
Rel(auth_uc, i_audit, "Usa")
Rel(auth_uc, i_auth_provider, "Usa")

' 4. Infraestrutura -> Portas (Implementação / Herança)
Rel_Up(repo_ticket_impl, i_repo_ticket, "Implementa")
Rel_Up(repo_staff_impl, i_repo_staff, "Implementa")
Rel_Up(rmq_adapter, i_notifier, "Implementa")
Rel_Up(s3_adapter, i_storage, "Implementa")
Rel_Up(repo_audit_impl, i_audit, "Implementa")
Rel_Up(ldap_adapter, i_auth_provider, "Implementa")

' 5. Infraestrutura -> Mundo Externo
Rel(repo_ticket_impl, db, "SQL")
Rel(repo_staff_impl, db, "SQL")
Rel(repo_audit_impl, db, "Insert")

Rel(rmq_adapter, broker, "Publish")
Rel(s3_adapter, filesystem, "I/O")
Rel(ldap_adapter, auth_sys, "LDAP Search")

@enduml