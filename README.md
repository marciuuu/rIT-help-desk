# rIT-help-desk

> Sistema de gestão de chamados e suporte de TI.

## 📌 Visão Geral
O **rIT-help-desk** é uma solução para triagem, atendimento e governança de incidentes e requisições técnicas de TI. A aplicação adota o estilo arquitetural de **Monólito Modular (Package by Component)**, isolando regras de negócio em módulos coesos e desacoplados, sustentada por princípios de Clean Architecture e Clean Code

## 🛠️ Tecnologias & Engenharia
- **Linguagem & Framework:** Java 21 | Spring Boot 3 (Spring Web, Spring Security, Spring Data JPA)
- **Persistência & Migrações:** Spring Data JPA / Hibernate | PostgreSQL | Flyway
- **Segurança & Auditoria:** Autenticação stateless (JWT), controle de acesso baseado em papéis (RBAC), senhas com BCrypt e trilha de auditoria em logs JSONB
- **Modelagem & Processos:** UML (PlantUML), DER/Relacional (brModelo, DBeaver)
