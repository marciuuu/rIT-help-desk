# Dicionário de Dados — rIT Help Desk

> **Banco de Dados:** PostgreSQL  
> **Fonte:** [`diagrama-classes-uml.puml`](./diagrama-classes-uml/diagrama-classes-uml.puml)  
> **Gerado em:** 2026-09-07  
> **Arquitetura:** Monólito Modular — relacionamentos entre módulos distintos usam referências por ID (soft references), sem FK declarada no banco para garantir baixo acoplamento entre módulos.

---

## Módulo: Identidade e Acesso (`auth`)

---

### Tabela: `departamentos`

**Descrição de Negócio:** Representa as unidades organizacionais (departamentos) da empresa. É utilizada para agrupar usuários e identificar a origem dos chamados abertos.

| Coluna | Tipo | Tamanho | Precisão | Escala | Null | PK | UK | FK | Referência | Check / Default |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| `id` | `BIGSERIAL` | 8 bytes | 19 dígitos | N/A | NOT NULL | ✅ | — | — | — | — |
| `nome` | `VARCHAR` | Variável, máx. 150 chars | N/A | N/A | NOT NULL | — | ✅ | — | — | — |
| `sigla` | `VARCHAR` | Variável, máx. 20 chars | N/A | N/A | NOT NULL | — | ✅ | — | — | — |

---

### Tabela: `usuarios`

**Descrição de Negócio:** Representa todos os atores do sistema: solicitantes (usuários comuns), técnicos de suporte e administradores. O campo `perfil` define o nível de acesso e as permissões dentro da aplicação. `matricula_tecnica` é preenchido somente para usuários com perfil `TECNICO`.

| Coluna | Tipo | Tamanho | Precisão | Escala | Null | PK | UK | FK | Referência | Check / Default |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| `id` | `BIGSERIAL` | 8 bytes | 19 dígitos | N/A | NOT NULL | ✅ | — | — | — | — |
| `departamento_id` | `BIGINT` | 8 bytes | 19 dígitos | N/A | NOT NULL | — | — | ✅ | `departamentos(id)` | — |
| `nome` | `VARCHAR` | Variável, máx. 200 chars | N/A | N/A | NOT NULL | — | — | — | — | — |
| `email` | `VARCHAR` | Variável, máx. 255 chars | N/A | N/A | NOT NULL | — | ✅ | — | — | — |
| `hash_senha` | `VARCHAR` | Variável, máx. 255 chars | N/A | N/A | NOT NULL | — | — | — | — | — |
| `perfil` | `VARCHAR` | Variável, máx. 30 chars | N/A | N/A | NOT NULL | — | — | — | — | `CHECK (perfil IN ('ADMINISTRADOR', 'TECNICO', 'USUARIO_COMUM'))` |
| `status` | `VARCHAR` | Variável, máx. 30 chars | N/A | N/A | NOT NULL | — | — | — | — | Ver nota ¹ |
| `matricula_tecnica` | `VARCHAR` | Variável, máx. 50 chars | N/A | N/A | NULL | — | ✅ | — | — | — |

> ¹ **`status` (enum `StatusUsuario`):** Os valores possíveis de `StatusUsuario` não estão explicitados no diagrama. Recomenda-se adicionar `CHECK (status IN ('ATIVO', 'INATIVO', 'BLOQUEADO'))` após alinhamento com o domínio de negócio.

---

## Módulo: Gestão de Chamados (`tickets`)

---

### Tabela: `categorias`

**Descrição de Negócio:** Classifica os chamados em categorias hierárquicas (árvore de subcategorias). Um registro pode ser subcategoria de outro através do autorrelacionamento `categoria_pai_id`. Categorias inativas não devem ser selecionadas em novos chamados.

| Coluna | Tipo | Tamanho | Precisão | Escala | Null | PK | UK | FK | Referência | Check / Default |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| `id` | `BIGSERIAL` | 8 bytes | 19 dígitos | N/A | NOT NULL | ✅ | — | — | — | — |
| `categoria_pai_id` | `BIGINT` | 8 bytes | 19 dígitos | N/A | NULL | — | — | ✅ | `categorias(id)` | — |
| `nome` | `VARCHAR` | Variável, máx. 150 chars | N/A | N/A | NOT NULL | — | ✅ | — | — | — |
| `descricao` | `TEXT` | Variável, até 1 GB | N/A | N/A | NULL | — | — | — | — | — |
| `ativo` | `BOOLEAN` | 1 byte | N/A | N/A | NOT NULL | — | — | — | — | `DEFAULT TRUE` |

---

### Tabela: `chamados`

**Descrição de Negócio:** Entidade central do sistema. Registra todas as solicitações de suporte abertas pelos usuários. Armazena o ciclo de vida completo do chamado, desde a abertura até o fechamento, incluindo controle de SLA, prioridade, técnico responsável e solução aplicada. Relacionamentos com `usuarios` e `departamentos` são **soft references** (sem FK no banco) por cruzarem fronteiras de módulo.

| Coluna | Tipo | Tamanho | Precisão | Escala | Null | PK | UK | FK | Referência | Check / Default |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| `id` | `BIGSERIAL` | 8 bytes | 19 dígitos | N/A | NOT NULL | ✅ | — | — | — | — |
| `categoria_id` | `BIGINT` | 8 bytes | 19 dígitos | N/A | NOT NULL | — | — | ✅ | `categorias(id)` | — |
| `solicitante_id` | `BIGINT` | 8 bytes | 19 dígitos | N/A | NOT NULL | — | — | — | *soft ref →* `usuarios(id)` | — |
| `tecnico_atribuido_id` | `BIGINT` | 8 bytes | 19 dígitos | N/A | NULL | — | — | — | *soft ref →* `usuarios(id)` | — |
| `departamento_origem_id` | `BIGINT` | 8 bytes | 19 dígitos | N/A | NOT NULL | — | — | — | *soft ref →* `departamentos(id)` | — |
| `titulo` | `VARCHAR` | Variável, máx. 300 chars | N/A | N/A | NOT NULL | — | — | — | — | — |
| `descricao` | `TEXT` | Variável, até 1 GB | N/A | N/A | NOT NULL | — | — | — | — | — |
| `data_abertura` | `TIMESTAMPTZ` | 8 bytes | 6 dígitos (µs) | N/A | NOT NULL | — | — | — | — | `DEFAULT NOW()` |
| `data_fechamento` | `TIMESTAMPTZ` | 8 bytes | 6 dígitos (µs) | N/A | NULL | — | — | — | — | — |
| `status` | `VARCHAR` | Variável, máx. 30 chars | N/A | N/A | NOT NULL | — | — | — | — | `CHECK (status IN ('ABERTO', 'EM_ANDAMENTO', 'AGUARDANDO_USUARIO', 'RESOLVIDO', 'FECHADO'))` DEFAULT `'ABERTO'` |
| `prioridade` | `VARCHAR` | Variável, máx. 20 chars | N/A | N/A | NOT NULL | — | — | — | — | `CHECK (prioridade IN ('BAIXA', 'MEDIA', 'ALTA', 'CRITICA'))` |
| `tempo_sla_horas` | `INTEGER` | 4 bytes | 10 dígitos | N/A | NULL | — | — | — | — | `CHECK (tempo_sla_horas > 0)` |
| `sla_violado` | `BOOLEAN` | 1 byte | N/A | N/A | NOT NULL | — | — | — | — | `DEFAULT FALSE` |
| `solucao_aplicada` | `TEXT` | Variável, até 1 GB | N/A | N/A | NULL | — | — | — | — | — |

---

### Tabela: `comentarios`

**Descrição de Negócio:** Registra as interações textuais em um chamado, podendo ser mensagens do solicitante, atualizações do técnico ou notas internas. O campo `visibilidade` controla se o comentário é visível apenas para a equipe de TI (`INTERNO_TI`) ou para todos os envolvidos (`PUBLICO`). O relacionamento com `usuarios` é **soft reference** por cruzar fronteira de módulo.

| Coluna | Tipo | Tamanho | Precisão | Escala | Null | PK | UK | FK | Referência | Check / Default |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| `id` | `BIGSERIAL` | 8 bytes | 19 dígitos | N/A | NOT NULL | ✅ | — | — | — | — |
| `chamado_id` | `BIGINT` | 8 bytes | 19 dígitos | N/A | NOT NULL | — | — | ✅ | `chamados(id)` | — |
| `autor_id` | `BIGINT` | 8 bytes | 19 dígitos | N/A | NOT NULL | — | — | — | *soft ref →* `usuarios(id)` | — |
| `texto` | `TEXT` | Variável, até 1 GB | N/A | N/A | NOT NULL | — | — | — | — | — |
| `data_hora` | `TIMESTAMPTZ` | 8 bytes | 6 dígitos (µs) | N/A | NOT NULL | — | — | — | — | `DEFAULT NOW()` |
| `visibilidade` | `VARCHAR` | Variável, máx. 20 chars | N/A | N/A | NOT NULL | — | — | — | — | `CHECK (visibilidade IN ('PUBLICO', 'INTERNO_TI'))` DEFAULT `'PUBLICO'` |

---

### Tabela: `anexos`

**Descrição de Negócio:** Armazena os metadados de arquivos anexados a um chamado. O arquivo em si é armazenado em serviço externo (object storage); esta tabela guarda apenas a referência via URL e informações descritivas para exibição e controle de tamanho.

| Coluna | Tipo | Tamanho | Precisão | Escala | Null | PK | UK | FK | Referência | Check / Default |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| `id` | `BIGSERIAL` | 8 bytes | 19 dígitos | N/A | NOT NULL | ✅ | — | — | — | — |
| `chamado_id` | `BIGINT` | 8 bytes | 19 dígitos | N/A | NOT NULL | — | — | ✅ | `chamados(id)` | — |
| `nome_arquivo` | `VARCHAR` | Variável, máx. 255 chars | N/A | N/A | NOT NULL | — | — | — | — | — |
| `caminho_url` | `TEXT` | Variável, até 1 GB | N/A | N/A | NOT NULL | — | — | — | — | — |
| `tamanho_kb` | `INTEGER` | 4 bytes | 10 dígitos | N/A | NULL | — | — | — | — | `CHECK (tamanho_kb > 0)` |
| `data_upload` | `TIMESTAMPTZ` | 8 bytes | 6 dígitos (µs) | N/A | NOT NULL | — | — | — | — | `DEFAULT NOW()` |

---

## Módulo: Governança e Auditoria (`governance`)

---

### Tabela: `logs_auditoria`

**Descrição de Negócio:** Registra todas as alterações significativas ocorridas nas entidades do sistema (chamados, usuários, categorias etc.), formando uma trilha de auditoria imutável. O campo `payload_alteracoes` armazena em JSONB o estado anterior e/ou posterior do dado alterado. Os relacionamentos com `usuarios` e com outras entidades são **soft references** por cruzarem fronteiras de módulo — esta tabela intencionalmente não possui FKs, garantindo que registros de auditoria sobrevivam a exclusões lógicas.

| Coluna | Tipo | Tamanho | Precisão | Escala | Null | PK | UK | FK | Referência | Check / Default |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| `id` | `BIGSERIAL` | 8 bytes | 19 dígitos | N/A | NOT NULL | ✅ | — | — | — | — |
| `entidade_alvo_id` | `BIGINT` | 8 bytes | 19 dígitos | N/A | NOT NULL | — | — | — | *soft ref →* entidade auditada | — |
| `nome_entidade` | `VARCHAR` | Variável, máx. 100 chars | N/A | N/A | NOT NULL | — | — | — | — | Ex: `'chamados'`, `'usuarios'` |
| `payload_alteracoes` | `JSONB` | Variável, até 1 GB | N/A | N/A | NOT NULL | — | — | — | — | — |
| `usuario_autor_id` | `BIGINT` | 8 bytes | 19 dígitos | N/A | NOT NULL | — | — | — | *soft ref →* `usuarios(id)` | — |
| `data_hora` | `TIMESTAMPTZ` | 8 bytes | 6 dígitos (µs) | N/A | NOT NULL | — | — | — | — | `DEFAULT NOW()` |

---

## Resumo de Enumerações (Domínios)

| Enum (UML) | Coluna(s) que utiliza | Valores possíveis |
| :--- | :--- | :--- |
| `PerfilAcesso` | `usuarios.perfil` | `ADMINISTRADOR`, `TECNICO`, `USUARIO_COMUM` |
| `StatusUsuario` | `usuarios.status` | *(não definido no diagrama — ver nota ¹)* |
| `StatusChamado` | `chamados.status` | `ABERTO`, `EM_ANDAMENTO`, `AGUARDANDO_USUARIO`, `RESOLVIDO`, `FECHADO` |
| `NivelPrioridade` | `chamados.prioridade` | `BAIXA`, `MEDIA`, `ALTA`, `CRITICA` |
| `TipoVisibilidade` | `comentarios.visibilidade` | `PUBLICO`, `INTERNO_TI` |