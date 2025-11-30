@startuml
!include <C4/C4_Context>

' Definição de Título
title Diagrama de Contexto C4 (Nível 1) \n Sistema de Gerenciamento de Demandas - TJGO

' Definição dos Atores (Pessoas)
Person(servidor, "Servidor do Tribunal", "Funcionário que abre e acompanha seus chamados.")
Person(tecnico, "Técnico de TI", "Membro da equipe que atende e atualiza os chamados.")
Person(gestor_ti, "Gestor de TI", "Gerente da equipe. Define prioridades, cadastra técnicos e vê relatórios.")
Person(corregedor, "Corregedor", "Gestor da Corregedoria que visualiza os dashboards de desempenho (KPIs).")

' Definição dos Sistemas Externos
System_Ext(auth_system, "Sistema de Autenticação", "Sistema corporativo (ex: AD/LDAP) que gerencia identidades e perfis.")
System_Ext(email_system, "Sistema de E-mail", "Servidor de e-mail corporativo (SMTP) para envio de alertas.")

' Definição do Nosso Sistema (O "Limite")
System(sgd, "Sistema de Gerenciamento de Demandas (SGD)", "Permite o registro, atribuição, acompanhamento e gestão de demandas de TI da Corregedoria.")

' Definição das Relações (Interfaces)

' Atores para o Sistema
Rel(servidor, sgd, "Registra e acompanha seus chamados", "HTTPS")
Rel(tecnico, sgd, "Atende e atualiza o progresso dos chamados", "HTTPS")
Rel(gestor_ti, sgd, "Gerencia equipe, prioridades e relatórios", "HTTPS")
Rel(corregedor, sgd, "Visualiza dashboards de indicadores (KPIs)", "HTTPS")

' Sistema para Externos
Rel(sgd, auth_system, "Valida credenciais e consulta perfis de usuário", "LDAP/HTTPS")
Rel(sgd, email_system, "Envia notificações de chamados (novos, atualizados, concluídos)", "SMTP")

@enduml