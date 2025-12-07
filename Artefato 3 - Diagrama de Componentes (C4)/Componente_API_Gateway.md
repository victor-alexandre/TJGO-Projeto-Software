@startuml
!include <C4/C4_Component>

' Layout da Esquerda para Direita favorece a leitura "Entrada -> Processamento -> Saída"
LAYOUT_LEFT_RIGHT()

title Diagrama de Componentes - API Gateway/REST (CORE)

' --- SISTEMAS E CONTAINERS EXTERNOS ---
ContainerDb(db, "PostgreSQL", "RDBMS", "Dados Transacionais e Logs")
ContainerDb(filesystem, "Object Storage", "S3/MinIO/NFS", "Binários dos Anexos")
System_Ext(email_sys, "SMTP Server", "Exchange", "Envio de E-mails")
System_Ext(auth_sys, "Identity Provider", "LDAP/AD", "Autenticação Corporativa")

' ============================================================
' 1. CAMADA DE APRESENTAÇÃO (Drivers / Primary Adapters)
' ============================================================
Container_Boundary(presentation, "Camada de Apresentação (Web/API)") {
    
    ' O Porteiro (Segurança)
    Component(security, "Security Middleware", "Filter Chain", "Intercepta TUDO.\nValida Token, Sessão e RBAC.")

    ' Controllers (Pontos de Entrada)
    Component(auth_ctrl, "Auth Controller", "REST", "Login e Sessão.")
    Component(admin_ctrl, "Admin Controller", "REST", "Gestão de Técnicos (RA02).")
    Component(ticket_ctrl, "Ticket Controller", "REST", "Operação de Chamados (RF01).")
    Component(report_ctrl, "Report Controller", "REST", "Dashboards e KPIs (RF08).")
    Component(file_ctrl, "File Controller", "REST", "Upload/Download (Multipart).")
}

' ============================================================
' 2. CAMADA DE DOMÍNIO (Use Cases & Domain Services)
' ============================================================
Container_Boundary(domain, "Camada de Domínio (Core)") {
    
    ' --- USE CASES (Orquestradores de Fluxo) ---
    Component(auth_uc, "Auth UseCase", "App Service", "Lógica de Autenticação.")
    Component(staff_uc, "Staff UseCase", "App Service", "Cadastro e gestão de técnicos.")
    Component(ticket_uc, "Ticket UseCase", "App Service", "Orquestra abertura e atendimento.")
    Component(report_uc, "Analytics Service", "App Service", "Agregação de dados para relatórios.")
    Component(file_uc, "File UseCase", "App Service", "Regras de Upload (Tamanho, Tipo).")

    ' --- DOMAIN SERVICES (Lógica Especializada) ---
    Component(sla_service, "SLA Calculator", "Domain Service", "Calcula vencimento (Feriados/Horas).")
    Component(assign_service, "Assignment Strategy", "Domain Service", "Distribui chamados automaticamente.")
    Component(audit_service, "Audit Service", "Domain Service", "Rastreabilidade (Quem, Quando, Onde).")

    ' --- HELPER OBJECTS (Regras Estáticas) ---
    Component(calendar, "Calendar Rules", "Value Object", "Feriados e Expediente.")
    
    ' --- PORTAS (Interfaces - DIP) ---
    Component(i_repo_staff, "IStaffRepository", "Interface", "Contrato CRUD Técnicos.")
    Component(i_repo_ticket, "ITicketRepository", "Interface", "Contrato CRUD Chamados.")
    Component(i_repo_report, "IReportRepository", "Interface", "Contrato Leitura KPIs (CQRS).")
    Component(i_repo_audit, "IAuditRepository", "Interface", "Contrato Logs.")
   
    Component(i_storage, "IStoragePort", "Interface", "Contrato Persistência Física.")
    Component(i_notifier, "INotifierPort", "Interface", "Contrato Notificação.")
}

' ============================================================
' 3. CAMADA DE INFRAESTRUTURA (Secondary Adapters)
' ============================================================
Container_Boundary(infra, "Camada de Infraestrutura (Implementações)") {
    
    ' Implementações de Repositórios
    Component(repo_staff_impl, "Staff Repo Impl", "Prisma", "Impl IStaffRepository.")
    Component(repo_ticket_impl, "Ticket Repo Impl", "Prisma", "Impl ITicketRepository.")
    Component(repo_audit_impl, "Audit Repo Impl", "Prisma", "Impl IAuditRepository.")
    
    ' Implementação Otimizada (CQRS)
    Component(repo_report_impl, "Report Repo SQL", "Raw SQL", "Queries Otimizadas para KPIs.")

    ' Adaptadores Externos
    Component(ldap_adapter, "LDAP Adapter", "Client", "Conecta no AD.")
    Component(smtp_adapter, "SMTP Adapter", "Nodemailer", "Envia E-mail.")
    Component(storage_adapter, "Disk/S3 Adapter", "Driver", "Grava Binários.")
}

' ============================================================
' RELACIONAMENTOS (FLUXOS)
' ============================================================

' 1. Entrada Segura (Todos passam pelo Security)
Rel(security, auth_ctrl, "Fwd")
Rel(security, admin_ctrl, "Fwd")
Rel(security, ticket_ctrl, "Fwd")
Rel(security, report_ctrl, "Fwd")
Rel(security, file_ctrl, "Fwd")

' 2. Conexão Controller -> UseCase
Rel(auth_ctrl, auth_uc, "Login")
Rel(admin_ctrl, staff_uc, "Gerir Equipe")
Rel(ticket_ctrl, ticket_uc, "Operar Chamado")
Rel(report_ctrl, report_uc, "Ler KPIs")
Rel(file_ctrl, file_uc, "Upload")

' 3. Lógica de Negócio Detalhada
' --- Ticket usa serviços auxiliares ---
Rel(ticket_uc, sla_service, "Pede Prazo")
Rel(ticket_uc, assign_service, "Pede Técnico")
Rel(ticket_uc, audit_service, "Loga Ação")
Rel(sla_service, calendar, "Usa")

' --- Admin/Staff usa auditoria ---
Rel(staff_uc, audit_service, "Loga Alteração")

' --- Atribuição consulta carga ---
Rel(assign_service, i_repo_staff, "Lê Carga")

' --- Arquivo é independente ---
Rel(file_uc, i_storage, "Grava Bytes")


' 4. Persistência e Saída (Aplicação -> Interfaces)
Rel(auth_uc, ldap_adapter, "Valida")
Rel(staff_uc, i_repo_staff, "Persiste")
Rel(ticket_uc, i_repo_ticket, "Persiste")
Rel(ticket_uc, i_notifier, "Notifica")
Rel(report_uc, i_repo_report, "Lê (Read-Only)")
Rel(audit_service, i_repo_audit, "Grava Log")

' 5. Infraestrutura Implementando Interfaces (Setas UP)
Rel_Up(repo_staff_impl, i_repo_staff, "Impl")
Rel_Up(repo_ticket_impl, i_repo_ticket, "Impl")
Rel_Up(repo_report_impl, i_repo_report, "Impl")
Rel_Up(repo_audit_impl, i_repo_audit, "Impl")
Rel_Up(smtp_adapter, i_notifier, "Impl")
Rel_Up(storage_adapter, i_storage, "Impl")

' 6. Conexões Finais com Externo
Rel(repo_staff_impl, db, "SQL")
Rel(repo_ticket_impl, db, "SQL")
Rel(repo_audit_impl, db, "Insert")
Rel(repo_report_impl, db, "Select Aggregations")

Rel(smtp_adapter, email_sys, "SMTP")
Rel(storage_adapter, filesystem, "I/O")
Rel(ldap_adapter, auth_sys, "LDAP")

@enduml