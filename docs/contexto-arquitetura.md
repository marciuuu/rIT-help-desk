# Documentação de Contexto e Requisitos: rIT Help Desk

## 1. Visão Geral do Projeto
O **rIT Help Desk** é um sistema de governança e gestão de chamados de TI (Service Desk). Seu objetivo é erradicar o uso de canais informais de suporte, profissionalizar o atendimento e garantir rastreabilidade, métricas e controle de SLA.

### 1.1. Principais Características
* **Padronização:** Abertura de chamados com dados estruturados (categoria, prioridade, setor, anexos).
* **Interface e UX:** Acesso via PWA (Progressive Web App) para mobilidade técnica, com modos Light/Dark e painéis inspirados no Notion, Trello e Jira (Kanban e Backlog).
* **Automação e Rastreabilidade:** Trilha de auditoria imutável para cada transição de status e automação de alertas (SLA).
* **Notificações:** Ecossistema de comunicação em tempo real (E-mail e planejamento futuro para Bot no Telegram).

### 1.2. Diretrizes Arquiteturais (Contexto para IA)
* **Estilo Arquitetural:** Monólito Modular (Pacote por Componente).
* **Divisão Interna:** Camadas estritas dentro de cada módulo (`Controller` -> `Service` -> `Repository`).
* **Módulos de Domínio:** `auth`, `tickets`, `governance`, `notifications`.
* **Banco de Dados (Relacional):** Estratégia *Single Table* para usuários (uma única tabela `usuarios` com a coluna discriminadora `perfil`).
* **Processamento Assíncrono:** Disparo de notificações e auditorias utilizando Eventos de Domínio (ex: *Spring Events*) para não bloquear as transações do banco de dados.

---

## 2. Análise 5W2H

* **What? (O que será feito):** Um Sistema de Gestão de Chamados de TI (PWA), com quadros Kanban, controle de backlog, trilha de auditoria e RBAC.
* **Why? (Por que será feito):** Eliminar canais informais, centralizar demandas, garantir rastreabilidade, controlar SLAs e extrair métricas de produtividade/gargalos.
* **Who? (Quem estará envolvido):** Administrador de TI, Técnico de Suporte e Usuário Comum.
* **Where? (Onde será utilizado):** Ambiente institucional (Web Desktop para administração/abertura e PWA Mobile para suporte operacional em campo).
* **When? (Quando):** Fases de levantamento de requisitos/arquitetura concluídas. Próximos passos incluem desenvolvimento, implantação contínua e evolução (Roadmap: Telegram, relatórios avançados).
* **How? (Como será feito):** Interface responsiva, autenticação JWT, perfis de acesso restritos, formulários padronizados e interceptação transacional para logs imutáveis.
* **How Much? (Dimensão/Retorno):** Envolve esforço de design, arquitetura back-end/banco de dados e validação. O retorno esperado é a otimização do tempo, dados confiáveis para decisões gerenciais e justificativas estruturadas para investimentos.

---

## 3. Atores do Sistema

* **Solicitante (Usuário Comum):** Servidor/colaborador que utiliza o sistema para reportar incidentes, solicitar serviços, anexar evidências e acompanhar tickets.
* **Técnico (Suporte):** Profissional de TI que acessa o sistema para atender demandas, registrar notas internas, alterar status da fila e resolver chamados.
* **Administrador (Gestor de TI):** Controle total. Gerencia usuários, categorias, distribui chamados, supervisiona o Kanban global e extrai métricas.
* **Sistema (Automático):** Ator não-humano responsável por gerenciar trilhas de auditoria, calcular SLAs, validar JWT e disparar notificações.

---

## 4. Especificação de Casos de Uso

### UC0 - Autenticação e Autorização (Login)
* **Atores:** Solicitante, Técnico, Administrador.
* **Fluxo Principal:**
  1. O usuário acessa a página inicial (Web/PWA).
  2. Insere suas credenciais institucionais.
  3. O Sistema valida as credenciais, gera um token JWT identificando o perfil (RBAC).
  4. O Sistema redireciona o usuário para o painel correspondente.

### UC1 - Abertura de Chamado (Criação de Ticket)
* **Atores:** Solicitante, Sistema.
* **Fluxo Principal:**
  1. O Solicitante logado clica em "Novo Chamado".
  2. Preenche os campos obrigatórios (Categoria, Título, Setor, Descrição).
  3. Opcionalmente, anexa evidências (arquivos/imagens).
  4. Clica em "Enviar".
  5. O Sistema gera um ID, define o status como "Aberto", grava log de auditoria e salva o ticket.
  6. O Sistema dispara notificação para a fila global e confirmação para o solicitante.

### UC2 - Triagem e Atribuição (Gestão de Fila)
* **Atores:** Técnico, Administrador.
* **Fluxo Principal:**
  1. Visualiza o chamado na Fila de Entrada (Backlog).
  2. Analisa os detalhes e define a Prioridade (Baixa, Média, Alta, Crítica).
  3. O Administrador atribui o chamado a um Técnico, OU o Técnico clica em "Assumir Chamado".
  4. O Sistema altera o status para "Em Andamento" e grava log de auditoria.

### UC3 - Atendimento e Comunicação (Interações)
* **Atores:** Técnico, Solicitante.
* **Fluxo Principal:**
  1. O Técnico insere um comentário público solicitando mais informações.
  2. O Sistema altera o status para "Aguardando Usuário", notifica o Solicitante e grava log.
  3. O Solicitante responde via painel.
  4. O Sistema notifica o Técnico e retorna o status para "Em Andamento".
* **Fluxo Alternativo (Nota Interna):** O Técnico insere uma nota oculta (visível apenas para TI) descrevendo procedimentos executados.

### UC4 - Resolução e Fechamento
* **Atores:** Técnico, Solicitante.
* **Fluxo Principal:**
  1. O Técnico conclui o trabalho, registra a "Solução Aplicada" e altera o status para "Resolvido".
  2. O Sistema notifica o Solicitante para homologação.
  3. O Solicitante testa e clica em "Aceitar Resolução".
  4. O Sistema define status como "Fechado" e paralisa o cômputo do SLA.
* **Fluxo Alternativo (Recusa):** O Solicitante clica em "Recusar Resolução" com justificativa. O Sistema retorna o status para "Em Andamento".

### UC5 - Gestão de Usuários e Categorias
* **Atores:** Administrador.
* **Fluxo Principal:**
  1. Acessa o painel de "Configurações".
  2. Cadastra, edita ou inativa contas e perfis de acesso.
  3. Cria/organiza a árvore de categorias de serviço (ex: Hardware, Acessos).

### UC6 - Acompanhamento Visual (Kanban e Backlog)
* **Atores:** Técnico, Administrador.
* **Fluxo Principal:**
  1. Acessa a aba "Quadro de Chamados".
  2. O Sistema exibe a interface Kanban separada pelas colunas de status.
  3. O usuário arrasta um card entre colunas.
  4. O Sistema atualiza o banco de dados, dispara gatilhos de notificação e grava auditoria.

### UC7 - Rastreabilidade e Trilha de Auditoria
* **Atores:** Sistema (Automático), Administrador, Técnico.
* **Fluxo Principal:**
  1. Para qualquer alteração em um ticket (status, prioridade, responsável), o Sistema intercepta a transação.
  2. Salva um log imutável com: Usuário, Data/Hora, Valor Anterior e Valor Novo.
  3. Usuários da TI visualizam a linha do tempo completa na aba "Histórico" do ticket.

### UC8 - Monitoramento de SLA e Notificações
* **Atores:** Sistema (Automático).
* **Fluxo Principal:**
  1. Calcula continuamente o tempo decorrido do ticket com base na Prioridade.
  2. Se o SLA estiver prestes a vencer, dispara alertas visuais e notificações.
  3. Se o SLA for rompido, aplica marcação de "SLA Violado" (badge vermelho) e grava o evento.

### UC9 - Extração de Métricas e Relatórios
* **Atores:** Administrador.
* **Fluxo Principal:**
  1. Acessa "Dashboards e Relatórios".
  2. Aplica filtros (data, categoria, setor, técnico).
  3. O Sistema compila e exibe o tempo médio de atendimento, volume resolvido e gargalos operacionais.