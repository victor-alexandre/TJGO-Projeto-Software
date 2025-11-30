@startuml
' Usando a biblioteca C4 embutida (mais rápido e seguro contra firewall)
!include <C4/C4_Component>

' Se a linha acima falhar, descomente a linha abaixo para usar a versão online:
' !include https://raw.githubusercontent.com/plantuml-stdlib/C4-PlantUML/master/C4_Component.puml

' CORREÇÃO: Removido o espaço extra aqui
LAYOUT_WITH_LEGEND()

title Diagrama de Componentes - API Gateway/REST (Core)

' --- External Containers (Contexto) ---
Container(spa, "Aplicação Web (SPA)", "React/TypeScript", "Faz requisições HTTPS/JSON")
ContainerDb(db, "Banco de Dados", "PostgreSQL", "Armazena dados transacionais")
Container(rabbitmq, "Fila de Mensagens", "RabbitMQ", "Recebe eventos de negócio")
ContainerDb(filesystem, "Storage NFS", "Volume Compartilhado", "Armazena binários de anexos")
System_Ext(auth_sys, "Sistema de Auth Central", "AD/LDAP", "Valida credenciais")

' --- Container Boundary: API Core ---
Container_Boundary(api, "API Gateway/REST (Core)") {

    Component(auth_middleware, "Auth Middleware", "Express Middleware", "Intercepta requisições para validar tokens e sessões via SSO/LDAP.")
    
    Component(ticket_controller, "Ticket Controller", "Express Router/Controller", "Recebe requisições HTTP, valida inputs (Zod/Joi) e despacha para serviços.")
    
    Component(ticket_service, "Ticket Service", "Class/Module", "Implementa lógica de negócio principal: fluxo de chamados e regras de atendimento.")
    
    Component(sla_service, "SLA Service", "Class/Module", "Calcula prazos de vencimento baseados em prioridade e calendário.")
    
    Component(file_service, "File Service", "Class/Module", "Gerencia upload/download de streams de arquivos para o volume físico.")
    
    Component(event_producer, "Event Producer", "Amqplib Wrapper", "Publica eventos de domínio (ex: 'ChamadoCriado') na fila.")
    
    Component(ticket_repo, "Ticket Repository", "Prisma ORM Client", "Abstrai queries SQL e interage com o banco de dados.")

    ' --- Relações Internas ---
    Rel(ticket_controller, auth_middleware, "Usa", "middleware")
    Rel(ticket_controller, ticket_service, "Chama métodos de", "async/await")
    Rel(ticket_controller, file_service, "Gerencia upload via", "multipart")
    
    Rel(ticket_service, sla_service, "Calcula prazos com", "function call")
    Rel(ticket_service, ticket_repo, "Persiste dados via", "Prisma Client")
    Rel(ticket_service, event_producer, "Notifica eventos via", "AMQP")
    
    Rel(file_service, ticket_repo, "Salva metadados do arquivo", "Prisma Client")
}

' --- Relações Externas ---
Rel(spa, auth_middleware, "Envia Requisição", "HTTPS/JSON")
Rel(auth_middleware, auth_sys, "Valida Token", "LDAP/SSO")

Rel(ticket_repo, db, "Leitura/Escrita", "TCP/SQL")
Rel(event_producer, rabbitmq, "Publica Mensagem", "AMQP 0-9-1")
Rel(file_service, filesystem, "Grava/Lê Binários", "NFS Mount")

@enduml
