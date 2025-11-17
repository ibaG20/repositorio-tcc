# Justificativa de Projeto de TCC

## **PWA de Agendamento para Microempreendedores com Foco em Acessibilidade e Eficiência Operacional**

**Introdução: O Problema e a Oportunidade**

O Brasil conta com mais de 14,6 milhões de Microempreendedores Individuais (MEIs), que formam a espinha dorsal da economia e da geração de renda no país. A rotina desses profissionais... é marcada por um desafio constante: a gestão do tempo. Métodos de agendamento informais, como agendas de papel e trocas de mensagens no WhatsApp, são ineficientes, propensos a erros e consomem um tempo precioso.

Este projeto de TCC propõe o desenvolvimento de um Progressive Web App (PWA) de agendamento, projetado especificamente para sanar as dores deste público. A proposta se diferencia fundamentalmente das soluções existentes no mercado por três pilares: simplicidade radical, uma jornada de cliente automatizada e, o mais importante, uma arquitetura tecnológica *Cache-First* (otimizada para cache) que atende à realidade da conectividade no Brasil.

---

**Seção 1: A Lacuna no Mercado de Agendamento**

Uma análise aprofundada do mercado revela que, embora existam diversas ferramentas de agendamento, nenhuma delas foi verdadeiramente concebida para a realidade operacional do MEI generalista brasileiro.

**1.1. Análise dos Concorrentes**

As plataformas existentes podem ser agrupadas em três categorias principais:

**a) Ferramentas Complexas e de Alto Custo (Focadas em PMEs):** 

- **Concorrentes:** Booksy e Trinks.
- **Barreiras:** São sistemas de gestão completos (estoque, comissões) com custos de assinatura (R$ 99+) que os tornam financeiramente inviáveis para o MEI.

**b) Ferramentas com Modelos de Baixo Valor ou Limitados (Modelo "Isca"):** 

- **Concorrentes:** Minha Agenda Virtual e Reservio.
- **Barreiras:** Utilizam modelos *freemium* que impõem severas limitações (ex: número de agendamentos) para forçar um upgrade, criando barreiras artificiais.

**c) Ferramentas que Não Resolvem a Dor da Conectividade *Precária*:**

- **Concorrentes:** Prit, Gendo e, na prática, todas as outras mencionadas.
- **Barreiras:** Essas soluções não apenas dependem 100% de uma conexão ativa, mas também não são otimizadas para a *qualidade* dessa conexão. Elas presumem banda larga estável, resultando em uma experiência lenta e um alto consumo de dados em redes móveis instáveis, o que penaliza o MEI que depende de um plano de dados restrito.

---

**Seção 2: O Desafio Real da Conectividade no Brasil (Custo e Latência)**

A ideia de "Offline-First" como solução para a conectividade brasileira parte de uma premissa correta (a rede é ruim), mas chega a uma conclusão tecnicamente perigosa e complexa (sincronização de dados). O problema real do MEI não é apenas a ausência de sinal, mas a **precariedade** e o **custo** do sinal quando ele existe.

Dados da pesquisa TIC Domicílios 2023 validam este diagnóstico:

1. **A Qualidade é um Privilégio:** A grande maioria dos brasileiros (57%) vive em condições "precárias" de acesso à internet.
2. **Abismo Social:** Apenas 3% das classes D e E (onde se concentra o MEI) têm conexão satisfatória.
3. **O Custo Pesa no Bolso:** O preço do gigabyte (GB) força o "racionamento" do uso da internet.

Para o profissional em campo, que opera em "zonas de sombra" ou com um plano de dados limitado, uma aplicação que exige *sincronização de banco de dados* (o cerne do "offline-first") é tão ou mais custosa do que uma aplicação online tradicional. Cada tentativa de *sync* consome dados preciosos.

A verdadeira inovação não é evitar a rede, mas usá-la de forma **cirurgicamente eficiente**.

---

**Seção 3: A Proposta de Valor e o Diferencial do TCC**

O projeto resolve o problema da conectividade não com a complexidade de um banco local, mas com a eficiência de uma arquitetura *Cache-First* na borda.

1. PWA com Arquitetura 'Cache-First' na Borda:

O coração do projeto é uma arquitetura que entrega velocidade e baixo custo de dados.

- **Operação:** A disponibilidade da agenda é calculada no servidor (Supabase Edge Function) e mantida em um cache de altíssima velocidade (Redis/Upstash).
- **Resultado:** Quando o MEI ou o cliente consulta a agenda, ele recebe uma resposta de poucos kilobytes (KB) em milissegundos. Isso ataca diretamente os dois problemas reais:
    - Custo de Dados: A requisição de 5KB é marginal e não impacta o "racionamento" de dados do MEI.
    - Latência: A resposta é quase instantânea, mesmo em redes 3G/4G instáveis.

2. Jornada do Cliente Automatizada e Profissional:

A plataforma automatiza os pontos de contato que o MEI hoje faz manualmente. O cliente recebe confirmações e lembretes (via Resend) e, crucialmente, pode cancelar ou reagendar 24/7 através de um link único (cancellation_token) enviado por email. Isso profissionaliza o MEI, reduz o "não-comparecimento" e libera o horário automaticamente, invalidando o cache para o próximo cliente.

3. Acessibilidade Através da Eficiência de Custos:

A arquitetura baseada em Supabase (Postgres), Upstash (Redis) e Resend (Email) não é apenas performática, é radicalmente barata. A análise de viabilidade financeira (detalhada no documento de arquitetura) demonstra um custo operacional marginal por usuário (estimado em R$ 0,068 por prestador/mês para 100.000 usuários). Isso permite um modelo de assinatura de baixo custo (e alto valor), quebrando a barreira de R$ 99+ dos concorrentes de forma sustentável.

---

**Conclusão**

Este projeto de TCC não é uma replicação de ferramentas existentes. É uma intervenção direcionada que nasce de uma análise profunda das necessidades do MEI e da realidade da infraestrutura brasileira.

Ele se justifica por sua **inovação técnica** (a eficiência de uma arquitetura 'Cache-First' na borda, em oposição à complexidade do 'Offline-First'), sua **relevância de mercado** (atender a um público massivo e mal atendido) e sua **viabilidade econômica** (um modelo de negócio sustentável baseado em eficiência de custos). O resultado é uma ferramenta de trabalho radicalmente eficiente e acessível, verdadeiramente adaptada à realidade do microempreendedor.