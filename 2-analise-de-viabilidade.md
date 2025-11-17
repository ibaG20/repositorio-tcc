# **Análise de Viabilidade Técnica e de Custos**

# PWA de Agendamento

### Introdução: O Modelo de Negócio e os Diferenciais

Este projeto foca-se no desenvolvimento de um PWA (Progressive Web App) de agendamento, desenhado especificamente para microempreendedores e prestadores de serviço.

Os diferenciais estratégicos deste modelo são:

- **Ser um PWA (Progressive Web App):** Oferece uma experiência de utilização semelhante a um aplicativo nativo, acessível através de um link direto, eliminando a necessidade de download e instalação via lojas de aplicativos (App Store, Google Play).
- **Profissionalizar com Simplicidade:** O fluxo de utilização é otimizado para ser simples e direto:
    - **Prestador de Serviço:** Efetua login (via Supabase Auth) numa área administrativa para gerir os seus horários e regras de disponibilidade.
    - **Cliente Final:** Não necessita de login. Acede ao link público (`public_slug`) do prestador, visualiza os horários disponíveis, escolhe o que deseja e preenche os dados solicitados para finalizar o agendamento.

### Jornada do Cliente: Automação e Profissionalismo

Um pilar central do projeto é a automação da jornada do cliente, projetada para transmitir profissionalismo e confiança:

1. **Confirmação Imediata:** Assim que o cliente preenche os dados do agendamento, recebe automaticamente um email profissional (enviado via Resend) com todos os detalhes da marcação.
2. **Autonomia de Cancelamento:** O email de confirmação contém um link de cancelamento único (baseado num `cancellation_token`). Caso o cliente precise de cancelar, pode fazê-lo a qualquer momento. Esta ação (via Edge Function) remove o agendamento no Postgres e dispara a invalidação do cache no Redis, libertando o horário para outros clientes de forma imediata, sem intervenção manual do prestador.

---

### Seção 1: Arquitetura Otimizada para Leitura

O maior desafio técnico de um sistema de agendamento é o alto volume de consultas (leituras) para verificar a disponibilidade. A arquitetura proposta foi desenhada para ser altamente eficiente e escalável, utilizando uma estratégia de cache inteligente.

### Stack Tecnológica

- **Hospedagem PWA:** Vercel (com Next.js) ou Netlify.
- **Backend/Banco/Auth:** Supabase (Postgres para dados, Auth para login, Edge Functions para lógica serverless).
- **Cache:** Upstash (Redis) para cachear os horários disponíveis.
- **Emails Transacionais:** Resend (para confirmações e cancelamentos).

### O Fluxo Otimizado

A lógica é separada em operações de escrita (raras) e leitura (massivas).

**1. Operação de Escrita (Lenta e Rara)**

- **Ação:** Um prestador cria/altera uma regra de disponibilidade, ou um cliente faz um novo agendamento.
- **Processo:** O PWA escreve a informação diretamente no banco Postgres do Supabase.
- **Gatilho (Trigger):** Um gatilho no Postgres (ou uma Edge Function chamada pela aplicação) é acionado. A sua **única responsabilidade** é invalidar o cache relevante no Redis (Upstash).
- **Exemplo de Comando:** `DEL "cache:<provider_id>:<date>"`
- **Nota:** A função de escrita apenas invalida. Ela não recalcula o cache, garantindo uma operação rápida.

**2. Operação de Leitura (Rápida e Massiva)**

- **Ação:** O cliente abre o link do prestador para ver os horários de um dia específico.
- **Processo:** O PWA chama uma Supabase Edge Function (ex: `getAvailability?date=...`).

**3. Lógica de Leitura na Edge Function (Otimização de Cache)**

Esta função contém a lógica principal de eficiência:

- **a. Verificação de Cache:** A função primeiro consulta o Redis: "Existe um cache para `<provider_id>:<date>`?"
- **b. Se SIM (Cache Hit):** O Redis retorna o JSON com os horários disponíveis guardados. A função devolve esta resposta em milissegundos. O custo de banco é zero.
- **c. Se NÃO (Cache Miss):** Apenas se o cache não existir, a função executa a lógica mais "lenta":
    1. Faz as consultas necessárias no Postgres (ex: regras de disponibilidade, agendamentos existentes, exceções/overrides).
    2. Monta o JSON com os horários livres.
    3. Salva esse JSON no Redis (com um TTL curto, ex: 10 minutos) para que as próximas leituras sejam rápidas.
    4. Retorna o JSON para o cliente.

**Resultado:** A primeira leitura de um dia específico é "lenta" (ex: 500ms), pois acede ao banco. Todas as milhares de leituras seguintes (nos próximos 10 minutos) serão instantâneas, servidas diretamente pelo Redis.

---

### Seção 2: Análise de Custos por Componente

*(Estimativa para uma escala de 100.000 Prestadores)*

### 2.1. Custo de Autenticação (Supabase Auth)

- **Limite Gratuito:** 50.000 MAUs (Usuários Ativos Mensais).
- **Excedente Estimado:** 50.000 MAUs.
- **Custo:** 50.000 MAUs * $0.00325/usuário = **$162,50/mês** (~R$ 812,50)

### 2.2. Custo do Banco e Processamento (Supabase Pro + Edge Functions)

- **Banco (Postgres):** O plano "Pro" é necessário para escalar além do limite de 500MB do plano gratuito.
    - **Custo:** $25/mês (~R$ 125,00)
- **Processamento (Edge Functions):**
    - **Uso Estimado:** 15 milhões de execuções/mês.
    - **Limite Gratuito:** 500.000 execuções.
    - **Excedente:** 14,5 milhões de execuções.
    - **Custo:** (14.500.000 / 1.000.000) * $2 = **$29/mês** (~R$ 145,00)

### 2.3. Custo do Cache (Upstash Redis)

- **Custo:** Utilizando o plano "Pay-as-you-go" para milhões de comandos, estima-se um custo de **$20/mês** (~R$ 100,00).

### 2.4. Custo dos Emails (Resend)

- **Uso Estimado:** Assumindo que 10% das operações (15M) resultam em agendamentos/cancelamentos, temos 1,5 milhões de emails/mês.
- **Limite Gratuito:** 3.000/mês (desconsiderado na escala).
- **Custo:** (1.500.000 / 1.000) * $0.75 = **$1.125,00/mês** (~R$ 5.625,00)

---

### Seção 3: Tabela de Tecnologias e Valores

*(Estimativa para 100.000 prestadores, câmbio R$ 5,00/USD)*

| **Tecnologia** | **Componente** | **Limite Gratuito (Mensal)** | **Custo Estimado (Mensal)** |
| --- | --- | --- | --- |
| Vercel / Netlify | Hospedagem do PWA | 100 GB de banda | R$ 0,00 |
| Supabase Auth | Autenticação (Login) | 50.000 MAUs | ~ R$ 812,50 ($162,50) |
| Supabase Pro | Banco Postgres + Egresso | - (Plano Pro) | ~ R$ 125,00 ($25) |
| Supabase Edge Func | Processamento (Serverless) | 500.000 execuções | ~ R$ 145,00 ($29) |
| Upstash (Redis) | Cache de Leitura | (Plano Free disponível) | ~ R$ 100,00 ($20) |
| Resend | Envio de Emails | 3.000 emails | ~ R$ 5.625,00 ($1.125) |
| **CUSTO TOTAL MENSAL** |  |  | **~ R$ 6.807,50** |

---

### Seção 4: Conclusão — Custo Real por Prestador

Com base nesta arquitetura e nos volumes estimados, o custo real por prestador é calculado da seguinte forma:

> Custo por Prestador = Custo Total Mensal / Número de Prestadores
> 
> 
> R$ 6.807,50 / 100.000 = **R$ 0,068**
> 

O custo operacional real é de aproximadamente **R$ 0,07 (sete centavos) por mês, por prestador**.

Esta análise detalhada, que inclui os custos essenciais de cache e envio de emails transacionais, fornece uma base sólida e realista para o planeamento financeiro do projeto.