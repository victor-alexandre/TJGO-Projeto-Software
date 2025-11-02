@startuml
!include https://raw.githubusercontent.com/plantuml-stdlib/C4-PlantUML/master/C4_Container.puml

LAYOUT_WITH_LEGEND()

title Diagrama de Container C4 (Nível 2) \n Sistema de Gerenciamento de Demandas - TJGO

' ===== ATORES (PESSOAS) =====
Person(servidor, "Servidor do Tribunal", "Funcionário que registra e acompanha chamados de TI")
Person(tecnico, "Técnico de TI", "Profissional que atende e resolve demandas técnicas")
Person(gestor_ti, "Gestor de TI", "Gerente que define prioridades, aloca recursos e monitora equipe")
Person(corregedor, "Corregedor", "Autoridade máxima que visualiza indicadores estratégicos (KPIs)")

' ===== BOUNDARY DO SISTEMA =====
System_Boundary(sgd, "Sistema de Gerenciamento de Demandas (SGD)") {
    
    ' Container 1: Frontend
    Container(spa, "Single-Page Application", "React 18 + TypeScript + Material-UI", "Interface responsiva que permite registro, acompanhamento e gestão de chamados. Suporta desktop e mobile.")
    
    ' Container 2: Backend API
    Container(api, "API REST", "Node.js 20 + Express + TypeScript", "Processa regras de negócio (RFs), gerencia chamados, atribuições automáticas, cálculo de SLAs e orquestra notificações.")
    
    ' Container 3: Banco de Dados
    ContainerDb(db, "Database", "PostgreSQL 15", "Armazena chamados, usuários, técnicos, históricos de auditoria (RNF05), SLAs e métricas de desempenho.")
    
    ' Container 4: Scheduler
    Container(scheduler, "Serviço de Agendamento", "Node.js + node-cron", "Executa tarefas periódicas: verificação de SLAs próximos do vencimento, geração de relatórios automáticos e alertas de chamados atrasados.")
    
    ' Container 5: Notification Service
    Container(notification, "Serviço de Notificações", "RabbitMQ 3.12 (Message Broker)", "Gerencia fila de notificações assíncronas. Garante entrega confiável de e-mails com retry automático e Dead Letter Queue.")
}

' ===== SISTEMAS EXTERNOS =====
System_Ext(auth_system, "Sistema de Autenticação", "Active Directory / LDAP corporativo TJGO. Gerencia identidades, credenciais e perfis de acesso.")
System_Ext(email_system, "Sistema de E-mail", "Servidor SMTP corporativo (Exchange). Envia notificações de chamados, alertas de SLA e relatórios.")

' ===== RELACIONAMENTOS: ATORES → SISTEMA =====
Rel(servidor, spa, "Registra chamados e acompanha status", "HTTPS")
Rel(tecnico, spa, "Atende chamados, atualiza progresso e resolve demandas", "HTTPS")
Rel(gestor_ti, spa, "Gerencia equipe, define prioridades e visualiza relatórios", "HTTPS")
Rel(corregedor, spa, "Visualiza dashboards estratégicos e indicadores de desempenho", "HTTPS")

' ===== RELACIONAMENTOS: CONTAINERS INTERNOS =====
Rel(spa, api, "Faz requisições REST (consulta, cria, atualiza chamados)", "HTTPS/JSON")
Rel(api, db, "Lê e escreve dados de chamados, usuários e históricos", "TCP/SQL (Prisma ORM)")
Rel(api, notification, "Publica eventos de notificação (chamado criado, atribuído, SLA)", "AMQP 0-9-1")

Rel(scheduler, db, "Consulta chamados com SLA próximo do vencimento", "TCP/SQL (Prisma ORM)")
Rel(scheduler, notification, "Agenda notificações periódicas (relatórios, alertas)", "AMQP 0-9-1")

Rel(notification, email_system, "Consome fila e envia e-mails de notificação", "SMTP/TLS")

' ===== RELACIONAMENTOS: SISTEMA → EXTERNOS =====
Rel(api, auth_system, "Valida credenciais e consulta perfis de usuário (SSO)", "LDAP v3 / LDAPS")

' ===== NOTAS ARQUITETURAIS =====
note right of notification
  **Padrão:** Publish-Subscribe
  **Retry:** 3 tentativas com backoff exponencial
  **DLQ:** Mensagens com falha permanente
  **TTL:** 24 horas
end note

note right of scheduler
  **Tarefas:**
  - Verificação de SLA (a cada 15 min)
  - Relatórios diários (00:00)
  - Alertas de atraso (a cada hora)
end note

@enduml