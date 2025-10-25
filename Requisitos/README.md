# Sistema de Gerenciamento de Demandas - Corregedoria TJGO

*O contexto e a proposta de valor deste projeto estão visualmente resumidos no Project Model Canvas (CanvasTJGO.jpg) e detalhados abaixo.*

## 1. Contexto

A Corregedoria do Tribunal de Justiça do Estado de Goiás (TJGO) desempenha um papel essencial no acompanhamento, fiscalização e orientação das atividades administrativas e judiciais. Além de suas funções centrais, a Corregedoria gera e recebe diversas demandas internas relacionadas a sistemas e processos de Tecnologia da Informação (TI).

Embora já existam sistemas judiciais para requisições específicas, eles não oferecem funcionalidades de gestão centralizada de demandas, semelhantes a plataformas de chamados corporativos.

## 2. O Problema

Atualmente, a equipe de TI da Corregedoria enfrenta um desafio operacional: não há um mecanismo integrado para controlar, priorizar e acompanhar o ciclo de vida das demandas de TI recebidas.

Quando surgem solicitações de manutenção, melhorias ou incidentes, a atribuição das tarefas ocorre de forma manual e verbal, alocada diretamente pelo gerente de projeto aos membros da equipe.

Esse processo informal gera uma série de lacunas ("gaps") operacionais:

* Dificuldade em acompanhar o histórico das demandas.
* Ausência de indicadores claros de desempenho (KPIs).
* Falta de visibilidade sobre a carga de trabalho (workload) de cada colaborador.
* Risco de esquecimento ou perda de demandas.
* Dificuldade em priorizar os chamados conforme critérios objetivos de negócio.

## 3. A Proposta de Solução

Para solucionar este cenário, este projeto propõe a criação de um **Sistema de Gerenciamento de Demandas para a Corregedoria do TJGO**, funcionando como uma central de serviços (*Service Desk*) interna.

O objetivo é que este sistema permita:

* O registro estruturado de todas as solicitações de TI.
* A atribuição automática ou semiautomática de tarefas aos responsáveis.
* O acompanhamento em tempo real do status de cada chamado.
* A definição de prazos, prioridades e SLAs.
* A emissão de relatórios e dashboards de apoio à gestão.

Com a implementação desta solução, a Corregedoria obterá maior eficiência operacional, transparência na distribuição das demandas, e um controle efetivo sobre os prazos e indicadores de desempenho da equipe de TI.

---

## Requisitos do Projeto

Os requisitos listados abaixo detalham as funcionalidades (RFs), regras de arquitetura (RAs) e restrições não-funcionais (RNFs) necessárias para a construção desta solução.




### RA01 - Autenticação e Perfis de Usuário

**Objetivo:** Garantir login e papéis distintos (Servidor, Técnico, Gestor, Corregedor).
**Fluxo Principal:**

1. Usuário acessa tela de login.
2. Informa credenciais.
3. Sistema valida e aplica permissões.
   **Prioridade:** Essencial (pré-requisito de todos os outros RFs).

---

### RA02 - Cadastro de Técnicos

**Objetivo:** Permitir que gestores cadastrem técnicos de TI no sistema.
**Fluxo Principal:**

1. Gestor acessa Cadastro de Técnicos".
2. Informa dados básicos (nome  e-mail  área).
   **Pós-condição:** Técnico disponível para atribuição de chamados.
   **Prioridade:** Essencial." DEV (green)

---

### RF01 - Registro de Chamados

**Objetivo:** Permitir que o servidor registre uma demanda de TI.
**Atores:** Servidor do Tribunal
**Pré-condições:** Usuário autenticado no sistema.
**Fluxo Principal:**

1. Servidor acessa Registrar Chamado".
2. Informa título  descrição  categoria  prioridade inicial e anexos (opcional).
3. Sistema valida os dados.
4. Chamado é registrado com um identificador único.
   **Pós-condição:** Chamado disponível para atribuição e acompanhamento.
   **Prioridade:** Alta (primeira funcionalidade a ser implementada)." DEV (green)

---

### RF02 - Atribuição Automática de Chamados

**Objetivo:** Distribuir automaticamente os chamados entre os técnicos.
**Atores:** Sistema, Gerente de TI
**Pré-condições:** Técnicos cadastrados + chamados pendentes de atribuição.
**Fluxo Principal:**

1. Chamado entra na fila de atribuição.
2. Sistema avalia a carga de trabalho da equipe.
3. Sistema atribui o chamado ao técnico mais adequado.
   **Fluxo Alternativo:** Se não houver técnico disponível, chamado permanece na fila.
   **Pós-condição:** Chamado vinculado a um técnico de TI.
   **Prioridade:** Alta.

---

### RF03 - Acompanhamento de Status

**Objetivo:** Usuário acompanha o status de seus chamados.
**Atores:** Servidor que abriu o chamado
**Pré-condições:** Usuário autenticado e com chamados registrados.
**Fluxo Principal:**

1. Usuário acessa Meus Chamados".
2. Sistema lista chamados (abertos  em andamento  concluídos).
3. Usuário seleciona um chamado e vê detalhes (atualizações  responsáveis).
   **Pós-condição:** Usuário tem visibilidade do progresso da sua demanda.
   **Prioridade:** Alta." DEV (green)

---

### RF04 - Definição de Prioridade de Chamados

**Objetivo:** Permitir ao gestor definir ou alterar prioridade de chamados.
**Atores:** Gestor
**Pré-condições:** Gestor autenticado.
**Fluxo Principal:**

1. Gestor acessa lista de chamados.
2. Seleciona um chamado.
3. Define prioridade (baixa, média, alta, crítica).
4. Sistema atualiza prioridade e reordena fila.
   **Pós-condição:** Chamados passam a ter tratamento conforme prioridade.
   **Prioridade:** Média.

---

### RF05 - Notificações de Novos Chamados

**Objetivo:** Notificar técnicos sobre novos chamados atribuídos.
**Atores:** Técnico de TI
**Pré-condições:** Técnico cadastrado e ativo.
**Fluxo Principal:**

1. Chamado é atribuído ao técnico.
2. Sistema envia notificação (e-mail, app ou sistema interno).
3. Técnico acessa chamado.
   **Pós-condição:** Técnico informado em tempo hábil.
   **Prioridade:** Média.

---

### RF06 - Histórico de Demandas

**Objetivo:** Permitir que o gestor visualize o histórico de chamados.
**Atores:** Gestor
**Pré-condições:** Gestor autenticado com perfil de gestão.
**Fluxo Principal:**

1. Gestor acessa Histórico de DemandAS".
2. Sistema apresenta chamados filtrados por período  tipo  unidade etc.
3. Gestor pode exportar ou analisar os dados.
   **Pós-condição:** Informações consolidadas para análise e planejamento.
   **Prioridade:** Média." DEV (green)

---

### RF07 - Relatórios Periódicos

**Objetivo:** Permitir que o gestor emita relatórios da equipe.
**Atores:** Gestor
**Pré-condições:** Gestor autenticado.
**Fluxo Principal:**

1. Gestor acessa Relatórios".
2. Define parâmetros (período  tipo  equipe etc.).
3. Sistema gera relatório visual e exportável (PDF  Excel).
   **Pós-condição:** Dados organizados para apresentação e análise.
   **Prioridade:** Baixa." DEV (green)

---

### RF08 - Dashboards de Indicadores

**Objetivo:** Exibir indicadores de desempenho para o corregedor.
**Atores:** Corregedor
**Pré-condições:** Corregedor autenticado e autorizado.
**Fluxo Principal:**

1. Corregedor acessa Indicadores".
2. Sistema apresenta métricas (tempo médio de atendimento  nº de chamados etc.).
3. Corregedor aplica filtros (período  tipo  unidade).
   **Pós-condição:** Corregedor tem visão consolidada da performance da equipe.
   **Prioridade:** Baixa." DEV (green)

