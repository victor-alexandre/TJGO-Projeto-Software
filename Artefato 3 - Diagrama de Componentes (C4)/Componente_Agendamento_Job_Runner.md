@startuml
!include <C4/C4_Component>

' Layout da Esquerda para Direita: Timer -> Job -> Processamento -> Saída
LAYOUT_LEFT_RIGHT()

title Diagrama de Componentes - Scheduler Service (Job Runner)

' --- SISTEMAS E CONTAINERS EXTERNOS (Dependências) ---
ContainerDb(db, "PostgreSQL", "RDBMS", "Consulta de SLAs e Dados para Relatórios")
ContainerQueue(broker, "RabbitMQ", "Exchange/Queue", "Publicação de Eventos (Ex: SLA Vencido)")
Component(api_core, "API Core", "Container", "Opcional: Reutilização de lógica via lib compartilhada")

' ============================================================
' 1. CAMADA DE INFRAESTRUTURA DE AGENDAMENTO (Driver)
' ============================================================
Container_Boundary(infra_cron, "Agendador (Driver)") {
    Component(cron_engine, "Cron Engine", "node-cron / Agenda", "Gerencia o ciclo de vida dos jobs e dispara eventos baseados em tempo (Time-based events).")
}

' ============================================================
' 2. CAMADA DE JOBS (Entry Points)
' ============================================================
Container_Boundary(jobs, "Camada de Jobs (Tarefas)") {
    Component(sla_job, "SLA Monitor Job", "Task", "Executa a cada 5 min.\nBusca chamados prestes a vencer ou vencidos.")
    Component(report_job, "Daily Report Job", "Task", "Executa à 00:00.\nGera consolidação estatística do dia anterior.")
    Component(cleanup_job, "Cleanup Job", "Task", "Executa semanalmente.\nLimpeza de logs temporários ou dados obsoletos.")
}

' ============================================================
' 3. CAMADA DE DOMÍNIO (Service Layer)
' ============================================================
Container_Boundary(domain, "Camada de Domínio (Lógica)") {
    
    ' Services
    Component(sla_checker, "SLA Checker Service", "Domain Service", "Aplica regras para identificar violação de SLA (considera feriados/calendário).")
    Component(report_builder, "Report Builder Service", "Domain Service", "Agrega dados brutos e calcula KPIs (TMA/TME).")
    
    ' Interfaces (Portas)
    Component(i_repo_ticket, "ITicketRepository", "Interface", "Leitura de chamados abertos.")
    Component(i_event_pub, "IEventPublisher", "Interface", "Contrato para notificar o sistema.")
}

' ============================================================
' 4. CAMADA DE ADAPTADORES (Infraestrutura de Saída)
' ============================================================
Container_Boundary(adapters, "Adaptadores de Saída") {
    Component(repo_ticket_impl, "Ticket Repo Impl", "Prisma Client", "Queries otimizadas para varredura (Batch processing).")
    Component(rmq_producer, "RabbitMQ Producer", "AMQP Client", "Publica mensagens nas filas de notificação.")
}

' ============================================================
' RELACIONAMENTOS
' ============================================================

' 1. O Gatilho de Tempo
Rel(cron_engine, sla_job, "Dispara (*/5 * * * *)")
Rel(cron_engine, report_job, "Dispara (0 0 * * *)")
Rel(cron_engine, cleanup_job, "Dispara (Weekly)")

' 2. Job orquestra o Serviço
Rel(sla_job, sla_checker, "Invoca Verificação")
Rel(report_job, report_builder, "Invoca Geração")

' 3. Serviço usa Portas
Rel(sla_checker, i_repo_ticket, "Busca Chamados Pendentes")
Rel(sla_checker, i_event_pub, "Publica 'SLA_EXPIRED'")
Rel(report_builder, i_repo_ticket, "Lê dados históricos")
Rel(report_builder, i_event_pub, "Publica 'REPORT_READY'")

' 4. Implementação das Interfaces
Rel_Up(repo_ticket_impl, i_repo_ticket, "Impl")
Rel_Up(rmq_producer, i_event_pub, "Impl")

' 5. Saída para o Mundo Externo
Rel(repo_ticket_impl, db, "SELECT/UPDATE (SQL)")
Rel(rmq_producer, broker, "AMQP Publish")

@enduml