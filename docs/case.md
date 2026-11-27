# Case pessoal: app Ninho

Nov 20, 2026 · @Guto

O case do programa é o **Ninho**, um app para a família usar no dia a dia, no navegador e no celular (PWA, mais um app Android de captura), com cinco módulos que entram ao longo das três formações. A arquitetura usa de propósito fila, cache, SQL e NoSQL, mesmo sem volume que justifique, para exercitar system design. Depois vem uma versão simplificada só com Postgres, e a comparação vira um ADR.

## Os cinco módulos

| Módulo | Problema que resolve | Conceitos de system design |
| --- | --- | --- |
| 1. Memória de gastos | Na hora de classificar no Mobills, ninguém lembra o que foi cada lançamento | Webhook de entrada, fila, idempotência, um leitor por banco (hexagonal), NoSQL para as notificações brutas, notificação de volta |
| 2. Rotina | Divisão das tarefas da casa e listas de compras | Tempo real, edição offline com conflito, placar em Redis, mural em NoSQL |
| 3. Agenda e lembretes | Compromissos da família e lembretes que não podem falhar; os lembretes saem pelo serviço de Notificações | Fila com atraso, outbox, entrega idempotente, sincronização nos dois sentidos com o Google Calendar (webhooks, conflitos), preferências de canal |
| 4. Consumo | Acompanhar água e energia e detectar falhas (vazamento, defeito) | Série temporal em NoSQL, ingestão, agregações, detecção estatística, migração de dados |
| 5. Despensa | Saber o que acabou e não esquecer de comprar | Eventos entre módulos, cache; depois busca textual e importação de receitas por link (fila, idempotência por URL) |

## Requisitos funcionais

### 1. Memória de gastos

**MVP (F1 M2 e M3)**

- Anotação rápida no app: valor, o que foi, quem gastou (Guto ou a esposa), meio de pagamento (cartão Itaú ou Nubank, Visa ou Master, conta ou dinheiro), data e hora automáticas, foto opcional.
- Lista do mês para conciliar, com filtros por pessoa, cartão e período e busca por valor.
- Marcar a anotação como conciliada.
- Fechamento do mês: o que ficou sem conciliar.
- A classificação continua no Mobills, feita pelo Guto, no fim do mês.

**Fase 2 (F1 M4): captura automática.** Um app Android lê as notificações de compra dos apps do Itaú e do Nubank, envia à API (guardando e reenviando se estiver sem internet), um leitor por banco extrai valor, estabelecimento e cartão, e quem comprou recebe "o que foi isso?" para responder na hora. Só esses dois apps são lidos, e nenhum dado vai para o repositório.

**Fase 3:** importação do Mobills com casamento automático entre anotação e lançamento; memória por estabelecimento. **Formação 3:** foto do cupom e voz com IA.

### 2. Rotina

**MVP (F1 M4)**

- Tarefas para todos da família: única ou recorrente, responsável, prazo e pontos.
- Conclusão sem aprovação (vale o que a pessoa marcar), com foto opcional.
- Pontos e placar por pessoa e por semana, sem conversão em dinheiro.
- Várias listas de compras (mercado, farmácia, feira…) compartilhadas em tempo real e funcionando sem internet, com quem pediu cada item e o que já foi comprado.
- Mural da família com o que cada um fez na semana.

**Depois do MVP:** recompensas por pontos, conquistas, lembretes de tarefa pela Agenda.

### 3. Agenda e lembretes

**MVP (F1 M5.1)**

- Calendário compartilhado por todos, uma cor por pessoa, visões de dia, semana e mês.
- Sincronização nos dois sentidos com uma agenda "Família" no Google Calendar; em conflito, vale a última edição e o app avisa.
- Recorrência e categorias (escola, saúde, aniversários, casa, lazer).
- Lembretes por push e e-mail com antecedência configurável; cada pessoa liga e desliga cada canal quando quiser.
- Lembrete com confirmação: repete até alguém marcar "feito".
- Lembretes enviados pelo serviço de Notificações, um contexto próprio que os outros módulos também usam.

**Depois do MVP:** resumo do dia seguinte, aviso de conflito de horário.

### 4. Consumo

**MVP (F1 M5.3)**

- Água: leitura semanal do hidrômetro e leitura da conta, que fecha o ciclo. Energia: só pela conta, em kWh. O valor em reais de cada conta é registrado quando ela chega.
- Migração das 101 leituras do painel Consumo da Casa, por pipeline com validação.
- Consumo por ciclo, média diária, comparação com o ciclo anterior e com o mesmo mês do ano anterior.
- Detecção de falhas: escore z modificado por ciclo (atenção ≥ 2, alerta ≥ 3), com base na mediana e no MAD dos últimos 12 ciclos; banda de erro de ±1 m³ na leitura semanal; CUSUM para tendência. No histórico real, pegou os 4 incidentes anotados sem nenhum alarme falso.

**Depois do MVP:** gás, custo estimado pela tarifa, leitura do medidor por foto (Formação 3), metas de redução.

### 5. Despensa

**MVP (Formação 2)**

- Itens da casa com estado "tem" ou "acabou", por categoria.
- Item marcado como "acabou" entra sozinho na lista de compras certa da Rotina; comprado, volta para "tem".
- Perfil alimentar da família, cadastrado aos poucos: preferências, restrições e o que cada um não gosta.

**Depois do MVP:** quantidade em estoque; receitas, com cadastro manual e importação por link de sites; cardápio da semana para todas as refeições, respeitando o perfil alimentar; lista de compras gerada do cardápio; busca por nome e ingrediente; validade dos itens; assistente de IA que monta o cardápio (Formação 3).

## Arquitetura da versão completa

| Peça | Tecnologia | O que exercita |
| --- | --- | --- |
| Front web e celular | React como PWA instalável, na Vercel | Cache no cliente, uso offline |
| App de captura | Android (Kotlin ou React Native), instalado fora da loja | Leitura de notificações, fila local offline, envio idempotente |
| Backend | NestJS, monólito modular com um módulo por área, no AWS Lightsail | Fronteiras de domínio e, depois, extração de um serviço |
| Banco transacional | PostgreSQL no Supabase, usado só como banco | Transações e constraints |
| Banco de documentos | MongoDB (Atlas, plano gratuito) | Notificações brutas, mural, auditoria, séries temporais |
| Cache e placar | Redis (Upstash, plano gratuito) | Cache-aside, TTL, sorted sets, rate limit |
| Fila | RabbitMQ: Docker localmente, CloudAMQP (plano gratuito) em produção | At-least-once, DLQ, retry com backoff, consumidor idempotente, mensagens com atraso |
| Tempo real | WebSocket (Socket.IO) | Fan-out e reconexão |
| Notificações | Web Push e e-mail (provedor gratuito), num serviço em Go a partir do F1 M5.1 | Outbox, entrega idempotente, preferências por canal |
| Integrações | Google Calendar API com webhooks | Sincronização nos dois sentidos, retry, circuit breaker |
| Deploy e CI | Vercel (front), AWS Lightsail (backend e Notificações), Supabase; GitHub Actions | Ambientes e pipeline |

Localmente, tudo sobe com Docker Compose. A decisão de hospedagem está no [ADR 0002](adr/0002-hospedagem-e-infraestrutura.md).

## Quando cada módulo entra no programa

| Momento | O que se constrói no Ninho |
| --- | --- |
| F1 M1.1 e M1.2 | Curso da Alura como aquecimento. No fim do M1.2 nasce o repositório do Ninho: CLAUDE.md, 1 skill e o primeiro PR com o agente (login, família, membros e papéis) |
| F1 M2 | PRD e design doc da Memória de gastos |
| F1 M3 | Memória de gastos (MVP) com spec e testes antes do código e revisor automático no CI |
| F1 M4 | Rotina (MVP) e a captura automática no Android, construídas por 2 ou 3 agentes em paralelo |
| F1 M5.1 | Agenda e lembretes; Notificações extraído como serviço separado, com ADR de extração; versão simplificada só com Postgres, comparada em outro ADR |
| F1 M5.2 | Refatoração com DDD do código que os agentes geraram |
| F1 M5.3 | Consumo: migração do painel e detecção de falhas; SLOs e observabilidade a fundo |
| F1 M5.4 | ADR do front: PWA mais app Android de captura, monorepo modular |
| F1 M6.1 | Pasta de contexto, glossário, rubrica de review e modelo de ameaças |
| F1 M6.2 | Agente revisor de PR e agente de operação (fila parada, DLQ cheia, alerta de consumo) |
| F1 M7 | Sem construção: o one-pager de adoção usa o que o Ninho mostrou |
| Formação 2 | Despensa (MVP) com processo de produto: conversas com a família, RICE, feature flag e medição de uso; depois receitas e cardápio |
| Formação 3 | IA no Ninho: foto do cupom e voz na Memória de gastos, leitura do medidor por foto, assistente de cardápio com RAG e ferramentas, evals, segurança e tracing |

## Arquitetura e boas práticas

Cada prática entra no Ninho no módulo do programa em que é estudada, com fonte que você já tem.

| Prática | Como aparece no Ninho | Quando | Fontes |
| --- | --- | --- | --- |
| DDD estratégico | 7 bounded contexts (os 5 módulos mais Notificações e Identidade), com mapa de contextos e glossário da linguagem ubíqua | F1 M2 (desenho) e M5.2 (refatoração) | Livro Full Cycle (DDD); *Learning Domain-Driven Design* |
| DDD tático | Invariantes no domínio: anotação conciliada uma vez só; notificação de compra processada uma vez só; lembrete com confirmação não some sem "feito" | F1 M3 e M5.2 | Livro Full Cycle; *Learning DDD* |
| Arquitetura hexagonal | Domínio sem dependência de framework; portas para fila, cache, banco de documentos e leitores de banco (Itaú e Nubank como adaptadores). A versão simplificada é só trocar adaptadores | F1 M3 em diante; troca no M5.1 | Livro Full Cycle (Hexagonal e Clean Architecture); Golang Tech Week |
| Monólito modular | Um deploy, módulos isolados, comunicação por eventos; extração só de Notificações | F1 M5.1 | *Deconstructing the Monolith* (Shopify); *MonolithFirst* |
| Eventos e outbox | Compra capturada vira pergunta "o que foi isso?"; item que acabou entra na lista de compras; leitura de consumo dispara a detecção de falhas | F1 M4 e M5.1 | Livro Full Cycle (Resiliência; Coreografia e orquestração) |
| CQRS leve | Modelos de leitura para o placar e para o consumo agregado, alimentados por eventos | F1 M5.1 | Livro Full Cycle |
| Resiliência | Retry com backoff e jitter, DLQ, circuit breaker na integração com o Google Calendar, rate limit na API de captura | F1 M5.1 | Livro Full Cycle (Resiliência); Tech Week for Seniors (mensageria) |
| Fitness functions | Testes que quebram o build se um módulo importar o domínio de outro ou se o domínio depender de framework | F1 M5.1 e M6.1 | *Building Evolutionary Architectures*; *Balancing Coupling* |
| Testes | Pirâmide: unidade no domínio, integração com containers reais, contrato entre front e API, poucos E2E | F1 M3 | Spec Kit; Learn Harness Engineering (L09 e L10); Alura módulo 5 |
| Feature flags | Todo módulo novo entra desligado e é liberado para a família aos poucos | F1 M4; F2 M4 | *Feature Toggles* (Pete Hodgson) |
| Segurança | Login por família, papéis (responsável e dependente), dados financeiros e de saúde protegidos, segredos fora do código | F1 M6.1; F3 M7 | OWASP Top 10 for LLM; Alura módulo 5 |

## Observabilidade técnica e de negócio

Tudo instrumentado com OpenTelemetry desde o primeiro módulo, para que cada decisão de arquitetura tenha um número por trás.

| Camada | O que medir no Ninho | Ferramenta | Quando |
| --- | --- | --- | --- |
| Traces, métricas e logs | Latência por rota, tempo e profundidade de fila, taxa de erro, cache hit ratio, logs estruturados com correlação de trace | OpenTelemetry com Grafana (local ou Grafana Cloud gratuito) | F1 M3 (base); F1 M5.3 (a fundo) |
| Erros | Exceções no front, no back e no app Android | Sentry (plano gratuito) | F1 M3 |
| SLOs e alertas | Notificação de compra processada em até 10 s; lembrete entregue até 1 min depois do horário; sincronização com o Google em até 2 min; DLQ vazia | Alertas no Grafana | F1 M5.3 e M6.2 |
| Métricas de negócio | % dos lançamentos do mês com anotação; tempo entre a compra e a resposta "o que foi isso?"; tarefas concluídas por pessoa; leituras semanais feitas no prazo; alertas de consumo confirmados e falsos | Dashboard de negócio no Grafana | F2 M2 e M5 |
| Analítica de produto | Uso semanal por pessoa, funil e retenção por módulo, adoção de cada feature liberada por flag | PostHog (plano gratuito) | F2 M2 e M5 |
| Observabilidade dos agentes | Rastro de cada execução do agente revisor e do agente de operação | Tracing do Claude Code e logs no repositório | F1 M6.2 |
| Tokens e custo de IA | Dos agentes que constroem o Ninho: tokens e custo por sessão e por PR. Das features de IA: tokens, cache, latência e custo por feature, por pessoa e por unidade de valor, com orçamento mensal e alerta | Telemetria do Claude Code por OpenTelemetry; convenções de GenAI do OpenTelemetry; Grafana | F1 M3, M6.2 e M7; F3 M2 e M7 |
| Evals de IA | Acurácia da leitura de cupom e de medidor; qualidade do cardápio sugerido; limite de autonomia do assistente | Conjunto dourado versionado, rodando no CI | F3 M6 |
| Métricas de engenharia | DORA do próprio repositório: frequência de deploy, lead time, falhas, tempo de recuperação | GitHub Actions e Insights | F1 M7 |

Fontes que você já tem: Tech Week for Seniors (OpenTelemetry), livro Full Cycle (observabilidade), Learn Harness Engineering L11, tracing da OpenAI e do MLflow, *Demystifying evals* e Hamel Husain, DORA 2025 e DX AI Measurement Framework.

## Documentos e processos

Cada tipo de documento é escrito uma vez de verdade no Ninho, no módulo em que é estudado, e fica versionado no repositório (`docs/`).

| Documento | No Ninho | Quando | Referência |
| --- | --- | --- | --- |
| CLAUDE.md, AGENTS.md e skills | Contexto do projeto para os agentes | F1 M1.2; revisado no M6.1 | *Claude Code: Best practices*; AGENTS.md |
| PRD | Um por módulo da casa | F1 M2; F2 M3 | Casos de uso da Claude Academy (PRD) |
| Design doc ou RFC | Memória de gastos: anotação, conciliação e captura automática | F1 M2 | *Design Docs at Google* |
| Spec (spec-driven) | Antes de cada feature construída por agente | F1 M3 em diante | Spec Kit; *Understanding SDD* (Böckeler) |
| ADR | Hexagonal, fila, extração de Notificações, versão completa × simplificada, método de detecção de falhas, front | F1 M2 a M5.4 | Nygard |
| Diagramas C4 | Contexto, contêineres e componentes do Ninho, usados também nas entrevistas | F1 M5.1 | Livro Full Cycle (arquitetura) |
| Mapa de contextos e glossário | Bounded contexts e linguagem ubíqua | F1 M5.2 e M6.1 | *Learning DDD* |
| Modelo de ameaças | Notificações bancárias, dados de saúde e de uma criança: o que proteger e como | F1 M6.1 | OWASP |
| SLOs e runbook | O que monitorar e o que fazer quando a fila trava ou a DLQ enche | F1 M5.3 | Livro Full Cycle (Resiliência) |
| Postmortem | Para cada incidente real do Ninho, mesmo pequeno | Quando acontecer | Caso de uso *Draft the incident postmortem* (Claude Academy) |
| Checklist de lançamento e release notes | Antes de liberar cada módulo para a família | F1 M4; F2 M4 | Caso de uso *Launch readiness sweep* |
| Relatório de impacto | Uso e resultado de cada módulo depois de um mês | F2 M5 | *Trustworthy Online Controlled Experiments* |

## Gestão do trabalho: GitHub Issues e Projects

O progresso do Ninho fica no GitHub Issues com um quadro no GitHub Projects. A planilha Checklist do Expert autodidata continua para o estudo; o GitHub fica para a construção.

- **Por que o GitHub:** fica junto do código, é gratuito, cada PR fecha a issue que resolve, e o Claude Code lê e escreve issues direto (CLI `gh` ou MCP do GitHub). Dá para mandar o agente "pegar a próxima issue pronta", que é o fluxo de fábrica do Módulo 4.
- **Organização:** um milestone por módulo do programa; labels por bounded context (`conciliacao`, `consumo`, `rotina`, `despensa`, `agenda`, `notificacoes`, `identidade`) e por tipo (`feature`, `bug`, `adr`, `spike`, `doc`).
- **Quadro:** Backlog, Pronto para o agente, Em andamento, Em revisão, Feito; campos de módulo do programa e de estimativa.
- **Modelos de issue:** feature (com critério de aceite e link para a spec), bug, ADR e spike.
- **Alternativas:** Linear tem a melhor experiência e também tem MCP, mas separa o trabalho do código; Jira é o que muitas empresas usam, mas é pesado para um projeto pessoal.

## Critérios de sucesso

1. O Ninho no ar e usado pela família toda semana: Memória de gastos e Rotina no celular, e a classificação do mês no Mobills feita sem "não lembro o que foi isso".
2. Um harness no repositório (CLAUDE.md, skill, spec, testes antes do código e revisor no CI) que pegou pelo menos um erro real do agente.
3. Um desenho de system design do Ninho, completo e simplificado, que eu explico em 5 minutos numa entrevista, mais o repositório público e um post de 1 página.

## Restrições

- Ritmo de 3 a 4 horas por semana, uma parte de módulo por semana.
- Stack do curso da Alura (Node, React, NestJS), mais o app Android de captura e o serviço de Notificações em Go; as peças extras só onde exercitam um conceito.
- Tudo em planos gratuitos; a AWS só onde não houver opção gratuita (hoje, só o backend, até US$ 10 por mês). O deploy entra no escopo, porque a família vai usar.
- Repositório público sem dados reais da família; segredos fora do código.
- Fora de escopo: publicação em loja de apps e escala real.

## Projeto futuro: Finanças completo

Fora da prioridade agora, porque o controle financeiro já é feito pelo dashboard gerado pela skill /dashboard-financeiro. Fica registrado para uma versão futura do Ninho.

**Funcionalidades de base**

1. Contas da família (corrente, cartões, dinheiro, poupança) e a conta da mesada.
2. Lançamentos de despesa e receita com valor, data, categoria, conta e autor, rápidos pelo celular.
3. Categorias padrão editáveis.
4. Contas fixas com vencimento, aviso antes de vencer e lançamento ao marcar como paga.
5. Transferências entre contas (ex.: pagamento de fatura), fora da despesa.
6. Saldo por conta e extrato filtrável.
7. Resumo do mês: entradas, saídas, gastos por categoria, contas a vencer.
8. Mesada: crédito pelos responsáveis, extrato próprio do filho.
9. Importação de extrato sem duplicar lançamentos.
10. Categorização automática dos importados.
11. Conciliação com o saldo real do banco.
12. Metas com progresso.
13. Orçamento por categoria com alerta.
14. Crédito de mesada a partir de tarefas da Rotina.

**Réplica do que a skill /dashboard-financeiro faz hoje**

- **Fluxo de caixa** a partir do relatório do Mobills, com as regras de negócio já validadas: compras do cartão por competência sem contar a fatura, exclusão de autotransferências e de aplicações e resgates automáticos, rendimentos como receita, eventos não recorrentes sinalizados.
- **Processamento incremental**: cada relatório novo começa no dia seguinte à última data processada; meses fechados não são recalculados; duplicatas de extração removidas.
- **Investimentos** a partir do relatório do Gorila: avaliação de cada ativo (bem, estável, mal), melhor e pior por classe, alocação por instituição com o teto do FGC, valores líquidos de IR em destaque.
- **Patrimônio**: caixa mais investimentos, distribuição por classe, evolução no ano e nos últimos 12 meses, bens à parte.
- **Indicadores**: saldo médio, despesa média e lastro (patrimônio sobre despesa média), com divisor crescendo até 12 meses.
- **Análise 70/20/10** (essencial, desejável, poupança), com os maiores gastos e sugestões.
