# 📅 PWA de Agendamento para Microempreendedores (TCC)

> Uma solução de agendamento Cache-First, escalável e de baixo custo, projetada para profissionalizar o atendimento do MEI brasileiro.
> 

---

## 🎨 Design & Prototipagem

[Figma](https://img.shields.io/badge/Acesse_o_Layout-Figma-F24E1E?style=for-the-badge&logo=figma&logoColor=white)

---

## 📖 Sobre o Projeto

Este projeto é parte do Trabalho de Conclusão de Curso (TCC) de Engenharia de Software da UTFPR.

O objetivo é resolver o problema da gestão de tempo de microempreendedores (manicures, barbeiros, consultores) que dependem de agendamentos manuais via WhatsApp. Diferente de soluções caras ou limitadas do mercado, este PWA oferece uma experiência fluida, profissional e simplificada.

### 🚀 Diferenciais Arquiteturais

O projeto utiliza uma arquitetura **Cache-First na Borda**:

- **⚡ Performance Extrema:** Consultas de disponibilidade batem no Cache (Redis/Upstash) e retornam em <50ms.
- **💸 Custo Marginal:** Arquitetura Serverless (Supabase) mantém o custo operacional próximo de zero (R$ 0,07/usuário).
- **🛡️ Segurança:** Transações ACID para evitar *race conditions* e Row Level Security (RLS) para proteção de dados.

---

## 🛠️ Tech Stack (A Melhor do Mercado)

Nós não usamos ferramentas de brinquedo.

- **Frontend:** [Next.js 14](https://nextjs.org/) (App Router, React Server Components) + Tailwind CSS.
- **Backend (BaaS):** [Supabase](https://supabase.com/) (PostgreSQL, Auth, Edge Functions).
- **Cache:** [Upstash](https://upstash.com/) (Serverless Redis).
- **Emails:** [Resend](https://resend.com/) (Emails transacionais).
- **Diagramas:** Mermaid.

---

## 🧩 Arquitetura e Modelagem

A documentação técnica completa e os diagramas de modelagem estão disponíveis na rais do repositório:

---

## ✨ Funcionalidades Principais

### 👨‍ Prestador de Serviço

- [x]  Login seguro.
- [x]  **Gestão de Regras de Horário:** Criar intervalos de trabalho flexíveis (ex: "Segunda 09-12h").
- [x]  **Exceções:** Bloquear dias específicos (Férias/Médico) ou adicionar horários extras.
- [x]  Cancelar agendamentos (com notificação automática).
- [x]  Visualizar agenda diária com status de ocupação.

### 👤 Cliente Final

- [x]  Acesso via Link Público (Sem login/senha).
- [x]  Visualização de horários livres (Cacheada).
- [x]  Agendamento em 3 cliques.
- [x]  **Email de Confirmação:** Contendo link único de cancelamento.
- [x]  Cancelamento autônomo via Token.
