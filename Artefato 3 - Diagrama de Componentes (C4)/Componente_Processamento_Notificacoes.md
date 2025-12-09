@startuml
!include <C4/C4_Component>

' Fluxo da Esquerda (Fila) para Direita (SMTP)
LAYOUT_LEFT_RIGHT()

title Diagrama de Componentes - Notification Service (Worker Assíncrono)

' --- 1. BLOCOS EXTERNOS (EXTREMIDADES) ---
ContainerQueue(broker, "RabbitMQ", "AMQP 0-9-1", "Origem dos Eventos (Queues: 'email.general', 'email.urgent')")
System_Ext(email_gateway, "SMTP Server", "Exchange/Relay", "Entrega final do e-mail")

Container_Boundary(notification_service, "Notification Service") {

    ' --- COLUNA 1: ENTRADA ---
    Boundary(listener_layer, "Camada de Mensageria (Entry Point)") {
        Component(consumer, "Queue Consumer", "RabbitMQ Client", "Mantém conexão persistente.\nEscuta filas e faz o 'Prefetch' de mensagens.")
        Component(resiliency, "Retry & DLQ Manager", "Resiliency Pattern", "Gerencia falhas (NACK).\nEnvia para Dead Letter Queue após X tentativas falhas.")
    }

    ' --- COLUNA 2: CORE (APLICAÇÃO E DOMÍNIO AGRUPADOS VISUALMENTE) ---
    ' Definimos primeiro a aplicação e depois o domínio para ficarem alinhados
    Boundary(app_layer, "Camada de Aplicação (Orquestração)") {
        Component(notification_handler, "Notification Handler", "UseCase", "Recebe o payload (JSON).\nValida dados obrigatórios (Destinatário, Assunto, Contexto).")
        
        ' As interfaces ficam aqui, próximas de quem as usa
        Component(i_email_sender, "IEmailSender", "Interface", "Contrato para envio de mensagens.")
        Component(i_template_provider, "ITemplateProvider", "Interface", "Contrato para carregar templates.")
    }

    Boundary(domain_layer, "Camada de Domínio (Core)") {
        Component(message_entity, "Message Entity", "Domain Object", "Regras de validação do conteúdo da mensagem.")
        Component(template_engine, "Template Engine", "Handlebars/EJS", "Carrega o template HTML em disco e injeta as variáveis do payload (Merge).")
    }

    ' --- COLUNA 3: INFRAESTRUTURA ---
    Boundary(infra_layer, "Camada de Infraestrutura") {
        Component(smtp_adapter, "SMTP Adapter", "Nodemailer Transport", "Impl IEmailSender (Gerencia pool de conexões SMTP.\nTransforma HTML em MIME Message).")
        Component(fs_adapter, "File System Adapter", "fs module", "Impl ITemplateProvider (Lê arquivos de templates (.hbs/.html) do disco local do container).")
    }
}

' ============================================================
' RELACIONAMENTOS (REORGANIZADOS PARA EVITAR CRUZAMENTOS)
' ============================================================

' 1. Fluxo de Entrada
Rel(broker, consumer, "Push de Mensagens (Eventos)")
Rel(consumer, resiliency, "Usa estratégia de Ack/Nack")
Rel(resiliency, broker, "Republica (Retry) ou move para DLQ")
Rel(consumer, notification_handler, "1. Envia Payload")

' 2. Fluxo Interno (Aplicação e Domínio)
Rel(notification_handler, message_entity, "2. Valida")
Rel(notification_handler, template_engine, "3. Solicita Merge")

' 3. Conexão com Interfaces
' O Handler usa o enviador
Rel(notification_handler, i_email_sender, "5. Usa")
' O Template Engine usa o provedor (Seta ÚNICA de dependência)
Rel(template_engine, i_template_provider, "4. Usa")

' 4. Implementações (Infraestrutura)
Rel_Up(smtp_adapter, i_email_sender, "Implementa")
Rel_Up(fs_adapter, i_template_provider, "Implementa")

' 5. Saída Externa
Rel(smtp_adapter, email_gateway, "SMTP / TLS")

@enduml