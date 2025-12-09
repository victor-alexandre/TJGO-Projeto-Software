# 🧭 Sistema de Gerenciamento de Demandas — Corregedoria TJGO  
### Corregedoria do Tribunal de Justiça do Estado de Goiás (TJGO)

Projeto desenvolvido como atividade principal da disciplina **Projeto e Design de Software**, do curso de Pós-Graduação da **Universidade Federal de Goiás (UFG)**, em parceria com o **Tribunal de Justiça do Estado de Goiás (TJGO)**.

O objetivo deste repositório é aplicar os conhecimentos adquiridos na disciplina para **transformar uma lista de requisitos de negócio** — elaborada na disciplina de **Engenharia de Requisitos** — em um **projeto de software completo, documentado e pronto para desenvolvimento**.

---

## 👥 Equipe do Projeto

| Integrante | GitHub |
|-------------|---------|
| José Solenir Lima Figuerêdo | [@Solenir](https://github.com/Solenir) |
| Owen Alves Lima | [@mr0wen](https://github.com/mr0wen) |
| Renato Aparecido dos Santos Júnior | [@renatojunior0](https://github.com/renatojunior0) |
| Victor Alexandre de Carvalho Coelho | [@victor-alexandre](https://github.com/victor-alexandre) |

### Orientação
**Professor:** [Fábio Nogueira de Lucena](https://ww2.inf.ufg.br/~fabio/)

---

## 📚 Sumário do Projeto

Esta seção serve como índice de navegação para os principais artefatos e documentos do projeto.

### 1. Visão Geral (neste documento)
- [Contexto do Projeto](#1-contexto)  
- [O Problema](#2-o-problema)  
- [A Proposta de Solução](#3-a-proposta-de-solução)

### 2. Artefatos de Projeto (links externos)
#### 🧩 Modelagem de Negócio
- Canvas:
[![Project Model Canvas](https://raw.githubusercontent.com/victor-alexandre/TJGO-Projeto-Software/main/imagens/CanvasTJGO.png)](imagens/CanvasTJGO.png)

#### 📝 Engenharia de Requisitos
- [Lista Detalhada de Requisitos (RF, RA, RNF)](Requisitos/README.md)  
- Casos de Uso:    
  [![Casos de Uso](https://raw.githubusercontent.com/victor-alexandre/TJGO-Projeto-Software/main/imagens/CasosDeUso.png)](imagens/CasosDeUso.png)


#### 🧠 Projeto de Software
- Diagrama de Contexto (Modelo C4):  
  [![Diagrama de Contexto - C4](https://raw.githubusercontent.com/victor-alexandre/TJGO-Projeto-Software/main/imagens/DiagramaDeContexto_C4.png)](imagens/DiagramaDeContexto_C4.png)

---
- Diagrama de Container (Modelo C4):  
  [![Diagrama de Container - C4](https://raw.githubusercontent.com/victor-alexandre/TJGO-Projeto-Software/main/imagens/DiagramaDeContainer_melhorado_C4.png)](imagens/DiagramaDeContainer_melhorado_C4.png)

---

- Diagrama de Componentes - API Gateway (Modelo C4):  
  [![Diagrama de Componente - API Gateway - C4](https://raw.githubusercontent.com/victor-alexandre/TJGO-Projeto-Software/main/imagens/DiagramaComponente_Api_Gateway_v1.2.png)](imagens/DiagramaComponente_Api_Gateway_v1.2.png)

  - Diagrama de Componentes - Serviço de Processamento de notificações (Modelo C4):
  [![Diagrama de Componente - Processamento Notificacoes - C4](https://raw.githubusercontent.com/victor-alexandre/TJGO-Projeto-Software/main/imagens/DiagramaProcessamentoNotificacoes_v1.1.png)](imagens/DiagramaProcessamentoNotificacoes_v1.1.png)

- Diagrama de Componentes - Serviço de Agendamento (Job runner) (Modelo C4):
  [![Diagrama de Componente - Agendamento - C4](https://raw.githubusercontent.com/victor-alexandre/TJGO-Projeto-Software/main/imagens/DiagramaAgendamentJobRunner_v1.1.png)](imagens/DiagramaAgendamentJobRunner_v1.1.png)


---

## 1️⃣ Contexto

A **Corregedoria do Tribunal de Justiça do Estado de Goiás (TJGO)** é responsável por supervisionar, fiscalizar e orientar as atividades administrativas e judiciais do tribunal.  

No âmbito da Tecnologia da Informação (TI), a Corregedoria gera e recebe **diversas demandas internas** relacionadas a sistemas, infraestrutura e suporte técnico.  

Apesar da existência de sistemas judiciais específicos, **não há uma plataforma unificada** para o gerenciamento centralizado dessas solicitações — semelhante a ferramentas corporativas de *Service Desk*.

---

## 2️⃣ O Problema

Atualmente, a equipe de TI da Corregedoria enfrenta um **problema de gestão operacional**:  
não existe um mecanismo integrado para **registrar, priorizar e acompanhar o ciclo de vida** das demandas de TI.

As solicitações de manutenção, melhorias ou incidentes são repassadas de forma **manual e verbal**, diretamente do gerente aos técnicos.  

Essa abordagem informal gera diversos entraves:

- Falta de rastreabilidade e histórico das demandas.  
- Ausência de indicadores e métricas de desempenho (KPIs).  
- Baixa visibilidade sobre a carga de trabalho de cada colaborador.  
- Risco de perda ou esquecimento de solicitações.  
- Dificuldade em definir prioridades baseadas em critérios objetivos.  

---

## 3️⃣ A Proposta de Solução

Para resolver esses desafios, o projeto propõe o desenvolvimento de um **Sistema de Gerenciamento de Demandas** para a Corregedoria do TJGO — uma solução interna inspirada em modelos de *Service Desk*.

O sistema visa proporcionar:

- **Registro estruturado** de solicitações de TI;  
- **Atribuição automática ou semiautomática** de tarefas;  
- **Monitoramento em tempo real** do status das demandas;  
- **Definição de prazos, prioridades e SLAs**;  
- **Geração de relatórios e dashboards** de apoio à gestão.  

### 🎯 Benefícios Esperados
A implementação da solução permitirá:

- Maior **eficiência operacional** na gestão das demandas;  
- **Transparência** na distribuição e acompanhamento das tarefas;  
- **Controle efetivo** de prazos, indicadores e carga de trabalho;  
- **Suporte estratégico** à tomada de decisão na área de TI.  

---

### 🧾 Licença
Este projeto é de uso **estritamente acadêmico** e foi desenvolvido no contexto da disciplina de Projeto de Software da **UFG**.  
Qualquer reutilização fora deste escopo deve ser previamente autorizada pelos autores e pelo orientador.
