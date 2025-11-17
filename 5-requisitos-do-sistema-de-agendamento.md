# Requisitos do Sistema de Agendamento

### 1. Requisitos Funcionais (RF)

### 1.1. Ator: Prestador de Serviço

| **ID** | **Requisito** | **Descrição (O QUE, seu merda)** |
| --- | --- | --- |
| **RF-P01** | Autenticar-se | O sistema deve permitir que o Prestador se autentique usando email/senha ou provedor social. (**Supabase Auth**). |
| **RF-P02** | Gerenciar Perfil | O Prestador deve poder criar e atualizar seu perfil público, incluindo `nome`, `telefone` e um `public_slug` (URL) único. |
| **RF-P03** | Gerenciar Grupos de Disponibilidade | O Prestador deve poder realizar CRUD (Criar, Ler, Atualizar, Deletar) em `AvailabilityRuleGroup` (Ex: "Horário Comercial"). |
| **RF-P04** | Ativar/Desativar Grupos | O Prestador deve poder Ativar ou Desativar um `AvailabilityRuleGroup`. O sistema deve impedir a ativação de grupos com dias da semana conflitantes (Regra de Negócio). |
| **RF-P05** | Gerenciar Regras de Disponibilidade | O Prestador deve poder, *dentro* de um grupo, definir intervalos (`start_time`, `end_time`) para dias da semana específicos (CRUD na `AvailabilityRule`). |
| **RF-P06** | Gerenciar Exceções (Overrides) | O Prestador deve poder bloquear (`is_available = false`) ou adicionar (`is_available = true`) horários específicos em datas específicas (CRUD na `AvailabilityOverride`). |
| **RF-P07** | Visualizar Agenda Diária | O sistema deve exibir uma visão diária que calcula a disponibilidade real, mostrando horários como: Disponível, Ocupado (`Appointment`), ou Bloqueado (`Override`). |
| **RF-P08** | Cancelar Agendamento (do Cliente) | O Prestador deve poder cancelar um `Appointment` existente, mudando seu `status` para `CANCELLED`. |
| **RF-P09** | Notificar Cliente (Cancelamento) | Ao executar **RF-P08**, o sistema deve *obrigatoriamente* disparar um email (via **Resend**) para o `Cliente`, informando do cancelamento. |
| **RF-P10** | Visualizar Lista de Clientes | O Prestador deve poder ver uma lista de clientes (`Client`) que já realizaram `Appointments` com ele. |

### 1.2. Ator: Cliente Final

| **ID** | **Requisito** | **Descrição (O QUE, seu merda)** |
| --- | --- | --- |
| **RF-C01** | Consultar Disponibilidade | Um Cliente *anônimo* deve poder acessar a agenda pública do prestador (via `public_slug`) e visualizar os horários disponíveis. |
| **RF-C02** | Realizar Agendamento | O Cliente deve poder selecionar um horário disponível e confirmar o agendamento fornecendo `nome`, `email` e `telefone`. |
| **RF-C03** | Criar/Vincular Cliente | Ao executar **RF-C02**, o sistema deve verificar se o `email` fornecido já existe na tabela `Client`. Se sim, vincular o `Appointment` ao `client_id` existente. Se não, criar um novo registro `Client`. |
| **RF-C04** | Notificar Cliente (Confirmação) | Ao executar **RF-C02**, o sistema deve *obrigatoriamente* disparar um email (via **Resend**) para o Cliente com o resumo do agendamento e o `cancellation_token` único. |
| **RF-C05** | Cancelar Agendamento (Próprio) | O Cliente deve poder cancelar seu próprio agendamento acessando um link único (contendo o `cancellation_token`) enviado no email de confirmação. |

---

### 2. Requisitos Não-Funcionais (RNF)

### 2.1. Desempenho (Performance)

- **RNF-D01 (Cache-First):** A consulta de disponibilidade do cliente (**RF-C01**) DEVE ser servida por um cache de borda (Redis/Upstash). 95% das requisições de disponibilidade devem ser respondidas em **menos de 100ms**.
- **RNF-D02 (PWA Load):** A aplicação (PWA) deve atingir uma pontuação de Performance no Google Lighthouse **acima de 90**.
- **RNF-D03 (TBT):** O TBT (Total Blocking Time) da página pública de agendamento (**RF-C01**) não deve exceder **200ms**, para garantir que o cliente possa clicar num horário sem atraso
- **RNF-D04 (Invalidação de Cache):** Qualquer operação de escrita (RF-P05, RF-P06, RF-P08, RF-C02, RF-C05) deve invalidar o cache (Redis) do dia/prestador afetado *imediatamente*.

### 2.2. Segurança (Security)

- **RNF-S01 (RLS Obrigatório):** A arquitetura **Supabase** DEVE implementar RLS (Row Level Security) em **TODAS** as tabelas. Um `provider_id` autenticado SÓ PODE ler/escrever os seus próprios dados.
- **RNF-S02 (Proteção de Acesso):** Todas as rotas de API do Prestador (ex: `/api/provider/*`) devem ser protegidas e exigir um JWT válido do Supabase Auth.
- **RNF-S03 (Token de Cancelamento):** O cancelamento do cliente (**RF-C05**) deve ser *impossível* sem o `cancellation_token` (UUID v4) único e imprevisível.
- **RNF-S04 (Não-Exposição):** O `cancellation_token` NUNCA deve ser exposto na URL pública do prestador, apenas no email de confirmação.

### 2.3. Confiabilidade e Integridade (Reliability)

- **RNF-C01 (Transação ACID):** O "Realizar Agendamento" (**RF-C02**) DEVE ser executado como uma **Transação de Banco de Dados** (ACID) dentro de uma Supabase Edge Function. Isso garante que o sistema verifique a disponibilidade e crie o `Appointment` atomicamente, impedindo *race conditions* (dois clientes marcando o mesmo horário).
- **RNF-C02 (Integridade de Dados):** O banco de dados (Postgres) DEVE usar `ENUMS` para a coluna `status` e `FOREIGN KEYS` para todos os relacionamentos (`provider_id`, `client_id`, `group_id`).
- **RNF-C03 (Entrega de Email):** O envio de emails (RF-C04, RF-P09) deve usar um serviço transacional robusto (Resend) e ser tratado de forma assíncrona (se possível) para não bloquear a resposta da API ao usuário.

### 2.4. Usabilidade (Usability)

- **RNF-U01 (PWA):** A aplicação DEVE ser um PWA (Progressive Web App) e ser instalável.
- **RNF-U02 (Mobile-First):** TODO o design deve ser Mobile-First.
- **RNF-U03 (Fluxo sem Fricção):** O fluxo de agendamento do Cliente (**RF-C01** a **RF-C04**) deve ser concluído em, no máximo, **3 etapas** e **NUNCA** exigir que o cliente crie uma senha ou faça login.

### 2.5. Manutenibilidade (A Minha Especialidade)

- **RNF-M01 (Código Limpo):** O codebase (Next.js/React) DEVE seguir os princípios SOLID, Clean Architecture (separação clara de `domain`, `data` e `presentation`) e usar TypeScript.
- **RNF-M02 (Stack Definida):** A stack tecnológica é Vercel (Frontend), Supabase (Auth, DB, Functions), Upstash (Cache) e Resend (Email).