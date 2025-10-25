# Sistema de Gerenciamento de Demandas (Service Desk)
## Corregedoria do Tribunal de Justiça do Estado de Goiás (TJGO)

Projeto desenvolvido como atividade principal da disciplina de **Projeto de Software**, do curso de Pós-Graduação da **Universidade Federal de Goiás (UFG)**, em colaboração com o **Tribunal de Justiça do Estado de Goiás**.

O objetivo central deste repositório é aplicar o aprendizado obtido na disciplina para transformar uma lista de requisitos de negócio (originados na disciplina de Engenharia de Requisitos) em um projeto de software estruturado, documentado e pronto para o desenvolvimento.

---

### Equipe do Projeto (Autores)

* [José Solenir Lima Figuerêdo](https://github.com/Solenir)
* [Owen Alves Lima](https://github.com/mr0wen)
* [Renato Aparecido dos Santos Júnior](https://github.com/renatojunior0)
* [Victor Alexandre de Carvalho Coelho](https://github.com/victor-alexandre)

### Orientação

* **Professor(a):** *Fábio Nogueira de Lucena*

---

## Sumário do Projeto

Esta seção serve como um índice de navegação para os artefatos e a documentação do projeto.

### 1. Visão Geral (Neste Documento)

Links para as seções principais deste `README.md`:

* [Contexto do Projeto](#1-contexto)
* [O Problema](#2-o-problema)
* [A Proposta de Solução](#3-a-proposta-de-solução)

### 2. Artefatos de Projeto (Links Externos)

Links para os documentos e diagramas detalhados do projeto:

* **Modelagem de Negócio:**
    * [Project Model Canvas](imagens/CanvasTJGO.jpg) 
* **Engenharia de Requisitos:**
    * [Lista Detalhada de Requisitos (RF, RA, RNF)](Requisitos/README.md)
    * [Casos de Uso](imagens/casos_de_uso.png)

---



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




