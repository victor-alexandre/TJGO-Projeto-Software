@startuml
!include <C4/C4_Component>

' Fluxo da Esquerda (Fila) para Direita (SMTP)
LAYOUT_LEFT_RIGHT()

title Diagrama de Componentes - Notification Service (Worker Assíncrono)

' --- SISTEMAS E CONTAINERS EXTERNOS ---
ContainerQueue(broker, "RabbitMQ", "AMQP 0-9-1", "Origem dos Eventos (Queues: 'email.general', 'email.urgent')")
System_Ext(email_gateway, "SMTP Server", "Exchange/Relay", "Entrega final do e-mail")

' ============================================================
' 1. CAMADA DE ENTRADA (Listener / Messaging Adapter)
' ============================================================
Container_Boundary(listener_layer, "Camada de Mensageria (Entry Point)") {
    
    Component(consumer, "Queue Consumer", "RabbitMQ Client", "Mantém conexão persistente.\nEscuta filas e faz o 'Prefetch' de mensagens.")
    
    Component(resiliency, "Retry & DLQ Manager", "Resiliency Pattern", "Gerencia falhas (NACK).\nEnvia para Dead Letter Queue após X tentativas falhas.")
}

' ============================================================
' 2. CAMADA DE APLICAÇÃO (Use Cases)
' ============================================================
Container_Boundary(app_layer, "Camada de Aplicação (Orquestração)") {
    
    Component(notification_handler, "Notification Handler", "UseCase", "Recebe o payload (JSON).\nValida dados obrigatórios (Destinatário, Assunto, Contexto).")
    
    Component(template_engine, "Template Engine", "Handlebars/EJS", "Carrega o template HTML em disco e injeta as variáveis do payload (Merge).")
}

' ============================================================
' 3. CAMADA DE INFRAESTRUTURA (Saída)
' ============================================================
Container_Boundary(infra_layer, "Camada de Infraestrutura") {
    
    Component(smtp_adapter, "SMTP Adapter", "Nodemailer Transport", "Gerencia pool de conexões SMTP.\nTransforma HTML em MIME Message.")
    
    Component(fs_adapter, "File System Adapter", "fs module", "Lê arquivos de templates (.hbs/.html) do disco local do container.")
}

' ============================================================
' RELACIONAMENTOS
' ============================================================

' 1. Consumo da Fila
Rel(broker, consumer, "Push de Mensagens (Eventos)")
Rel(consumer, resiliency, "Usa estratégia de Ack/Nack")
Rel(resiliency, broker, "Republica (Retry) ou move para DLQ")

' 2. Orquestração
Rel(consumer, notification_handler, "Envia Payload Validado")

' 3. Processamento do Template
Rel(notification_handler, template_engine, "Solicita Renderização")
Rel(template_engine, fs_adapter, "Lê arquivo do template")

' 4. Envio do E-mail
Rel(notification_handler, smtp_adapter, "Envia E-mail Renderizado")
Rel(smtp_adapter, email_gateway, "SMTP / TLS (Port 587)")

@enduml