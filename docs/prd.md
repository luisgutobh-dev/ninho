# PRD: Ninho

27/11/2026 · Proposta, versão 0.2

Documento de requisitos do Ninho, o app da família usado como case de system design. O contexto, a arquitetura e a ordem de construção estão no [case](case.md); este PRD fecha o que o produto faz e o que a engenharia se obriga a usar.

**Status:** proposta em revisão (issue #1).

| Versão | Data | O que mudou |
| --- | --- | --- |
| 0.1 | 24/11/2026 | Proposta inicial |
| 0.2 | 27/11/2026 | Identidade e família, Notificações, dados e privacidade, integrações, plataformas, métricas com definição e linha de base, evidências, risco de valor, orçamento de IA; conta própria do dependente decidida |

## Resumo

O Ninho é um app para uma família de três pessoas (dois responsáveis e um dependente), no navegador e no celular, com cinco módulos: Memória de gastos, Rotina, Agenda e lembretes, Consumo e Despensa. Cada módulo resolve um problema real da casa.

O PRD tem dois tipos de requisito não funcional. Os de produto são o que a família precisa (o lembrete chega na hora, a lista funciona sem internet). Os técnicos de estudo são tecnologias e padrões que o volume do Ninho não exige, mas que entram de propósito para praticar system design: fila, cache, banco SQL e NoSQL, outbox, tempo real, extração de serviço, observabilidade com SLOs. Cada um tem uma versão simplificada, e a comparação entre as duas vira ADR.

## Problema e contexto

- **Gastos sem memória:** o software financeiro já traz os lançamentos da conta e dos cartões por Open Finance, mas na hora de classificar no Mobills, no fim do mês, ninguém lembra o que foi cada compra, e o extrato não ajuda.
- **Rotina espalhada:** tarefas da casa e listas de compras ficam na cabeça ou em mensagens, sem saber quem fez o quê.
- **Compromissos e lembretes:** a agenda da família não é uma só, e lembretes importantes falham.
- **Consumo sem alerta:** as leituras de água e energia já são anotadas (101 leituras no painel Consumo da Casa), mas um vazamento só aparece quando a conta chega.
- **Despensa:** o que acabou não chega à lista de compras.

Todos na família usam Android. A família usa dois bancos, com cartões de crédito de cada um, e os apps dos bancos mandam push a cada compra.

**Evidências.** As conversas com a família estão em [discovery/conversas-familia.md](discovery/conversas-familia.md). As primeiras, sobre o Consumo:

- "Houve um vazamento que não havíamos identificado e só fomos descobrir quando a conta veio."
- "Hoje eu olho o consumo no relógio periodicamente e anoto em uma planilha que indica se teve uma divergência grande."
- "Somente eu olho o valor." (o alerta precisa chegar a quem acompanha as contas)

As evidências da Memória de gastos, da Rotina, da Agenda e da Despensa entram depois das conversas com os outros dois membros da família.

## Objetivos e métricas

**Objetivos de produto**

1. A classificação do mês no Mobills sem "não lembro o que foi isso".
2. Tarefas, listas, agenda e lembretes da família num lugar só, usados toda semana.
3. Falha de água ou energia detectada no ciclo em que acontece, não na conta seguinte.

**Objetivos de aprendizado**

1. Um desenho de system design do Ninho, completo e simplificado, explicado em 5 minutos numa entrevista.
2. Cada conceito de system design dos requisitos técnicos de estudo implementado, medido e registrado em ADR.
3. Um harness de desenvolvimento com agentes (CLAUDE.md, skills, spec, testes antes do código, revisor no CI) que pegou pelo menos um erro real.

| Métrica | Meta | Como medir | Linha de base |
| --- | --- | --- | --- |
| Lançamentos do mês com anotação (Memória de gastos) | 80% ou mais | Lançamentos do relatório do Mobills com anotação conciliada ÷ total de lançamentos do relatório, no fechamento do mês (RF-G-04) | Medir no fechamento de dezembro, antes do Ninho: quantos lançamentos ficaram sem lembrança |
| Tempo entre a compra e a resposta "o que foi isso?" (fase 2) | Mediana abaixo de 10 minutos | Horário da resposta menos horário do push do banco (eventos `compra_capturada` e `anotacao_respondida`), por pessoa e por semana | Não existe hoje; primeira medição na fase 2 |
| Pessoas com uso semanal | As 3, por 4 semanas seguidas depois de cada lançamento | Pessoa com pelo menos uma ação principal do módulo na semana (evento `acao_principal` no PostHog, por módulo) | Zero; a comparação é com o jeito atual descrito nas conversas |
| Leituras semanais de água feitas no prazo | 90% ou mais | Semanas com pelo menos uma leitura de água ÷ semanas do período | Calcular no histórico do painel Consumo da Casa durante a migração (RF-C-04) |
| Alertas de consumo | Nenhum incidente real perdido; no máximo 1 alarme falso por trimestre | Respostas a "era mesmo um problema?" (sim = incidente, não = alarme falso), por trimestre; incidente perdido = falha descoberta na conta sem alerta antes | Hoje, vazamento descoberto só na conta ou na planilha (4 incidentes no histórico) |
| Lembretes com confirmação marcados como "feito" | 95% ou mais | Lembretes com confirmação marcados "feito" ÷ lembretes com confirmação disparados, no mês | Contar nas conversas os compromissos esquecidos no último mês |
| Conceitos de estudo com ADR e medição | Todos os da tabela de requisitos técnicos (exceto os de IA) | RNT com ADR em `docs/adr/` e medição registrada ÷ RNT fora os de IA | Zero |

## Usuários e papéis

| Papel | O que pode |
| --- | --- |
| Responsável administrador | Tudo, incluindo membros, módulos ligados por feature flag e integrações |
| Responsável | Tudo dos módulos, sem administração da família |
| Dependente | Rotina, Agenda e Despensa; não vê a Memória de gastos nem o Consumo (leituras, consumo e contas) |

O login é por família: cada pessoa tem sua conta, e todos os dados são compartilhados dentro da família, exceto o que o papel restringe. O dependente tem conta própria desde o início.

## Requisitos funcionais

Os identificadores (RF-G-01 etc.) são os usados nas issues e nas specs.

### Identidade e família (base de todos os módulos)

**MVP, o walking skeleton**

- **RF-I-01** Criar a família: quem cria vira Responsável administrador.
- **RF-I-02** Convidar um membro por link, já com o papel definido; o convite vale 7 dias e serve uma vez só.
- **RF-I-03** Trocar o papel de um membro (só o administrador); a família nunca fica sem administrador.
- **RF-I-04** Remover um membro: o acesso acaba na hora, e o que ele criou continua na família.
- **RF-I-05** Login e recuperação de acesso, com o método decidido em ADR (ver Perguntas em aberto).
- **RF-I-06** Perfil de cada pessoa: nome, cor (usada na Agenda) e preferências de notificação (RF-N-02).

### 1. Memória de gastos

**MVP**

- **RF-G-01** Anotação rápida: valor, o que foi, quem gastou (um dos responsáveis), meio de pagamento (cartão de crédito, conta ou dinheiro), data e hora automáticas, foto opcional. Em até 3 toques a partir da tela inicial.
- **RF-G-02** Lista do mês para conciliar, com filtros por pessoa, meio de pagamento e período, e busca por valor.
- **RF-G-03** Marcar a anotação como conciliada; uma anotação conciliada não volta a pendente sem ação explícita.
- **RF-G-04** Fechamento do mês: o que ficou sem conciliar.
- A classificação continua no Mobills, feita pelo administrador no fim do mês.

**Fase 2: captura automática**

- **RF-G-05** O app Android lê só as notificações de compra dos apps dos bancos e as envia ao Ninho; sem internet, guarda e reenvia depois.
- **RF-G-06** O Ninho extrai valor, estabelecimento e cartão de cada notificação e cria uma anotação pendente, sem duplicar se a mesma notificação chegar duas vezes.
- **RF-G-07** Quem comprou recebe "o que foi isso?" e responde na hora, pela notificação ou no app.
- **RF-G-08** Notificação que não deu para ler fica visível para o administrador, com o texto original.

**Fase 3 e fase de IA**

- **RF-G-09** Importação do relatório do Mobills com casamento automático entre anotação e lançamento.
- **RF-G-10** Memória por estabelecimento ("da última vez, isso foi farmácia").
- **RF-G-11** Anotação por foto do cupom e por voz, com IA.

### 2. Rotina

**MVP**

- **RF-R-01** Tarefas para todos da família: única ou recorrente, responsável, prazo e pontos.
- **RF-R-02** Conclusão sem aprovação (vale o que a pessoa marcar), com foto opcional.
- **RF-R-03** Pontos e placar por pessoa e por semana, sem conversão em dinheiro.
- **RF-R-04** Várias listas de compras (mercado, farmácia, feira…), compartilhadas em tempo real, com quem pediu cada item e o que já foi comprado.
- **RF-R-05** As listas funcionam sem internet e sincronizam quando a conexão volta; se duas pessoas mexem no mesmo item, nada se perde sem aviso.
- **RF-R-06** Mural da família com o que cada um fez na semana.

**Depois do MVP:** recompensas por pontos, conquistas, lembretes de tarefa pela Agenda.

### 3. Agenda e lembretes

**MVP**

- **RF-A-01** Calendário compartilhado por todos, uma cor por pessoa, visões de dia, semana e mês.
- **RF-A-02** Sincronização nos dois sentidos com uma agenda "Família" no Google Calendar; em conflito, vale a última edição e o app avisa.
- **RF-A-03** Recorrência e categorias (escola, saúde, aniversários, casa, lazer).
- **RF-A-04** Lembretes por push e e-mail com antecedência configurável; o canal segue as preferências de cada pessoa (RF-N-02).
- **RF-A-05** Lembrete com confirmação: repete até alguém marcar "feito".
- **RF-A-06** Os lembretes saem pelo serviço de Notificações, um contexto próprio que os outros módulos também usam ("o que foi isso?", alerta de consumo, item que acabou).

**Depois do MVP:** resumo do dia seguinte, aviso de conflito de horário.

### 4. Consumo

**MVP**

- **RF-C-01** Água: leitura semanal do hidrômetro (m³, contador de 3 dígitos que volta a 000) e leitura da conta, que fecha o ciclo.
- **RF-C-02** Energia: só pela conta, em kWh, com os dias do ciclo.
- **RF-C-03** Valor em reais de cada conta, registrado quando ela chega.
- **RF-C-04** Migração das 101 leituras do painel Consumo da Casa, com validação e relatório do que foi importado.
- **RF-C-05** Consumo por ciclo, média diária, comparação com o ciclo anterior e com o mesmo mês do ano anterior.
- **RF-C-06** Detecção de falhas: escore z modificado por ciclo, M = 0,6745 × (x − mediana) / MAD, sobre os últimos 12 ciclos sem alerta confirmado (um alerta descartado devolve o ciclo à base); atenção com M ≥ 2 e alerta com M ≥ 3. Na leitura semanal, banda de ±1 m³ pelo arredondamento do hidrômetro. CUSUM (k = 0,5σ, h = 4σ, com σ = 1,4826 × MAD) para tendência de alta. No histórico real, o método pegou os 4 incidentes anotados sem alarme falso em 41 ciclos.
- **RF-C-07** Alerta pelo serviço de notificações, com a confirmação "era mesmo um problema?", que alimenta a métrica de alarme falso.

**Depois do MVP:** gás, custo estimado pela tarifa, leitura do medidor por foto (fase de IA), metas de redução.

### 5. Despensa

**MVP**

- **RF-D-01** Itens da casa com estado "tem" ou "acabou", por categoria.
- **RF-D-02** Item marcado como "acabou" entra sozinho na lista de compras certa da Rotina; comprado, volta para "tem".
- **RF-D-03** Perfil alimentar da família, cadastrado aos poucos: preferências, restrições e o que cada um não gosta.

**Depois do MVP:** quantidade em estoque; receitas com cadastro manual e importação por link de sites; cardápio da semana para todas as refeições, respeitando o perfil alimentar; lista de compras gerada do cardápio; busca por nome e ingrediente; validade dos itens; assistente de IA que monta o cardápio.

### Notificações (serviço usado por todos os módulos)

**MVP (como serviço separado; antes disso, módulo do monólito)**

- **RF-N-01** Canais: push (Web Push no PWA e notificação do app Android) e e-mail.
- **RF-N-02** Preferências por pessoa e por tipo de aviso (lembrete, "o que foi isso?", alerta de consumo, item que acabou): cada pessoa liga e desliga cada canal.
- **RF-N-03** Destinatários por tipo: o módulo diz quem recebe (por exemplo, o alerta de consumo vai só para quem acompanha as contas).
- **RF-N-04** Horário de silêncio por pessoa: o aviso espera o fim do horário, exceto os marcados como urgentes pelo módulo (vazamento).
- **RF-N-05** Reenvio quando a entrega falha, com intervalo crescente, e e-mail como reserva do push; o mesmo aviso nunca chega duas vezes (RNT-08).
- **RF-N-06** Histórico por pessoa: enviado, entregue, aberto e respondido.

**Depois do MVP:** resumo diário em vez de avisos soltos, para os tipos que a pessoa escolher.

## Requisitos não funcionais de produto

| ID | Requisito | Meta |
| --- | --- | --- |
| RNF-P-01 | Captura de compra processada | 95% em até 10 s entre a chegada ao Ninho e a pergunta "o que foi isso?" |
| RNF-P-02 | Lembrete entregue no horário | 99% em até 1 min depois do horário marcado |
| RNF-P-03 | Sincronização com o Google Calendar | 95% das mudanças refletidas em até 2 min, nos dois sentidos |
| RNF-P-04 | Uso sem internet | Listas de compras e anotação de gasto funcionam offline e sincronizam sem perda |
| RNF-P-05 | Nada se perde | Nenhuma notificação de compra, anotação ou leitura perdida; falha vai para a DLQ e é reprocessável |
| RNF-P-06 | Resposta da API com o serviço acordado | p95 abaixo de 300 ms nas leituras e 500 ms nas escritas |
| RNF-P-07 | Disponibilidade | Backend sempre ligado, em melhor esforço numa única instância; nenhuma mensagem se perde durante uma queda ou um deploy ([ADR 0002](adr/0002-hospedagem-e-infraestrutura.md)) |
| RNF-P-08 | Privacidade | Dados financeiros, de saúde (perfil alimentar) e do dependente só dentro da família; o app de captura lê apenas os apps dos bancos; repositório público sem dado real |
| RNF-P-09 | Segurança | Login com sessão segura, papéis aplicados no backend, segredos fora do código, HTTPS em tudo |
| RNF-P-10 | Backup | Cópia semanal automática do Postgres e do MongoDB, com restauração testada uma vez por formação |
| RNF-P-11 | Custo | Planos gratuitos em tudo; a AWS só onde não houver opção gratuita (hoje, só o backend), até US$ 10 por mês com alerta de orçamento ([ADR 0002](adr/0002-hospedagem-e-infraestrutura.md)). IA: orçamento próprio de até R$ 50 por mês, alerta em 80% e corte automático em 100% (RNT-23) |
| RNF-P-12 | Idioma, fuso e moeda | Português do Brasil, America/Sao_Paulo, reais |
| RNF-P-13 | Usabilidade | Ação principal de cada módulo em até 3 toques; funciona bem em tela de celular |

## Dados e privacidade

Detalha o RNF-P-08. Nenhum dado desta tabela vai para o repositório.

| Dado | Sensibilidade | Onde fica | Quem vê | Por quanto tempo |
| --- | --- | --- | --- | --- |
| Conta e perfil (nome, e-mail, cor) | Pessoal | Postgres | A família | Enquanto a pessoa for membro |
| Anotações de gasto e conciliação | Financeiro | Postgres | Responsáveis | Enquanto a família usar o Ninho |
| Texto bruto das notificações de banco | Financeiro, o mais sensível | MongoDB | Só o administrador (RF-G-08) | 90 dias; depois fica só a anotação extraída |
| Leituras e valores das contas | Doméstico; o valor é financeiro | Postgres e MongoDB (séries) | Só os responsáveis; o dependente não acessa o Consumo | Sem limite (série histórica) |
| Tarefas, listas, pontos e mural | Baixa | Postgres e MongoDB (mural) | A família | Mural: 12 meses |
| Agenda | Pessoal (saúde, escola) | Postgres e Google Calendar | A família | Segue o Google Calendar |
| Perfil alimentar | Saúde | Postgres | A família | Enquanto a pessoa for membro |
| Dados do dependente | De menor de idade | Os acima | Conforme o papel de Dependente | Apagados por completo a pedido |
| Logs e traces | Mascarados (RNT-15) | Grafana e Sentry | Só o administrador | 30 dias |
| Chamadas de IA | Metadados (tokens, custo); o conteúdo pode ter cupom ou voz | Postgres (RNT-23) | Só o administrador | Metadados por 12 meses; o conteúdo não é guardado |
| Backups | Todos os acima, criptografados | Cloudflare R2 (RNF-P-10) | Só o administrador | Até a cópia expirar |

- **RNF-P-14** Exportar: o administrador baixa todos os dados da família em JSON e CSV.
- **RNF-P-15** Apagar: um membro ou a família inteira, de todos os bancos; nos backups, o dado some quando a cópia expira.
- **RNF-P-16** IA: só o necessário vai ao provedor, sem nome nem dados pessoais, e com provedor que não usa os dados para treino.

## Requisitos não funcionais técnicos de estudo

Estes requisitos não vêm do volume do Ninho: uma família de três cabe num Postgres com uma aplicação simples. Eles entram de propósito, porque são o que uma entrevista de system design cobra, e porque o Ninho dá um caso real para medir cada um. A regra é a mesma para todos: implementar atrás de uma porta (arquitetura hexagonal), medir com OpenTelemetry, registrar a decisão em ADR e ter uma versão simplificada que só troca o adaptador.

| ID e requisito | Onde aparece no Ninho | O que exercita | Versão simplificada |
| --- | --- | --- | --- |
| RNT-01 Monólito modular com 7 bounded contexts | Conciliação de gastos, Consumo, Rotina, Despensa, Agenda e lembretes, Notificações e Identidade como módulos NestJS isolados, falando por eventos. A Captura é a ACL da Conciliação; os bancos, o Mobills e o Google Calendar são contextos externos. Mapa em [ddd/mapa-estrategico.md](ddd/mapa-estrategico.md) | DDD estratégico, fronteiras, mapa de contextos | A mesma, é a base das duas versões |
| RNT-02 Arquitetura hexagonal | Domínio sem framework; portas para banco, fila, cache, notificações e leitores de banco (um adaptador por banco) | Inversão de dependência, testabilidade, troca de infraestrutura | É o mecanismo da simplificação |
| RNT-03 PostgreSQL como banco transacional | Família, membros, anotações, tarefas, eventos, itens | Modelagem relacional, transações, constraints, índices, migrações | A mesma |
| RNT-04 MongoDB como banco de documentos | Notificações brutas do banco, mural, auditoria, séries de consumo | Quando usar documento em vez de tabela, esquema flexível, coleções de série temporal | Tabelas com JSONB no Postgres |
| RNT-05 Redis para cache, placar e rate limit | Cache-aside da agenda e do consumo agregado; placar da Rotina em sorted set; rate limit da API de captura | Cache-aside, TTL, invalidação, hit ratio, estruturas do Redis | Cache em memória e placar por consulta SQL |
| RNT-06 Fila de mensagens com DLQ | Captura de compra, envio de lembretes, detecção de falhas no consumo; Amazon SNS (tópicos por contexto) e SQS (uma fila por consumidor, com DLQ), na cota gratuita permanente; desenho no [ADR 0002](adr/0002-hospedagem-e-infraestrutura.md) | Entrega at-least-once, retry com backoff e jitter, DLQ, mensagens com atraso, consumidores concorrentes | Tabela de jobs no Postgres com SELECT … FOR UPDATE SKIP LOCKED |
| RNT-07 Outbox transacional | Evento gravado na mesma transação da mudança (compra capturada, item acabou, leitura registrada) e publicado depois | Consistência entre banco e fila, dual write | Outbox lido direto pelo worker, sem broker |
| RNT-08 Idempotência de ponta a ponta | Chave por notificação no app Android, por mensagem no consumidor, por lembrete na entrega, por URL na importação de receitas | Exactly-once efetivo sobre at-least-once | A mesma |
| RNT-09 Tempo real por WebSocket | Listas de compras e mural atualizados ao vivo para todos | Fan-out, reconexão, presença, escala horizontal com pub/sub | Atualização ao abrir a tela ou polling |
| RNT-10 Offline-first com resolução de conflito | Listas e anotações no PWA; fila local no app Android | Sincronização, relógio e versão por item, last-write-wins por campo ou CRDT (ADR) | Bloquear edição sem internet |
| RNT-11 Extração de um serviço | Notificações vira serviço separado, escrito em Go, num container próprio na mesma instância do monólito, com seu próprio banco | Quando extrair, contrato entre serviços, consistência eventual, deploy independente | Notificações de volta como módulo do monólito |
| RNT-12 Integração externa resiliente | Google Calendar nos dois sentidos com webhooks; push e e-mail | Webhooks, renovação de canais, retry, circuit breaker, timeout, conflito de edição | Só exportar para o Google, sem webhooks |
| RNT-13 CQRS leve | Modelos de leitura do placar e do consumo agregado, alimentados por eventos | Separar escrita e leitura, projeções, reconstrução | Consulta direta nas tabelas |
| RNT-14 Pipeline de migração de dados | 101 leituras do painel Consumo da Casa para o Ninho | Extração, validação, idempotência da carga, relatório de divergências | Script único |
| RNT-15 Observabilidade com SLOs | Traces, métricas e logs com OpenTelemetry no Grafana; erros no Sentry; SLOs dos RNF-P-01 a 03 com alerta e error budget; política de logs: JSON com nível, trace id, módulo e família, e mascaramento de valores, estabelecimentos e dados pessoais antes de gravar (o texto das notificações de banco nunca vai cru para o log) | Instrumentação, SLI e SLO, alertas por sintoma, runbook | Logs estruturados e Sentry |
| RNT-16 Métricas de negócio e de produto | Dashboard de negócio no Grafana; PostHog para uso, funil e retenção | Separar saúde técnica de resultado de produto | Consultas SQL num relatório mensal |
| RNT-17 Estimativa de capacidade e carga sintética | Estimativa de ordem de grandeza para 10 mil famílias e teste de carga com k6 contra o ambiente local | Back-of-the-envelope, gargalos, onde a fila e o cache fazem diferença | Sem teste de carga |
| RNT-18 Teste de falha | Derrubar a fila, o Redis ou o Google no Docker Compose e ver o que acontece | Degradação graciosa, recuperação, validação do runbook | Não se aplica |
| RNT-19 Feature flags | Todo módulo novo entra desligado e é liberado aos poucos | Entrega contínua, lançamento separado do deploy | Variável de ambiente |
| RNT-20 Fitness functions e testes | Build quebra se um módulo importar o domínio de outro ou se o domínio depender de framework; pirâmide com containers reais e contrato entre front e API | Arquitetura evolutiva, testes como harness para agentes | A mesma |
| RNT-21 CI/CD e métricas de engenharia | GitHub Actions com testes, revisor automático e deploy; DORA do repositório | Pipeline, ambientes, frequência de deploy, lead time | A mesma |
| RNT-22 IA com evals | Cupom, voz, leitura do medidor por foto e assistente de cardápio com RAG e ferramentas; conjunto dourado no CI; tracing | Evals, RAG, uso de ferramentas, limite de autonomia, segurança de LLM | Entrada manual |
| RNT-23 Observabilidade de uso de IA e tokens | Toda chamada a modelo das features de IA (cupom, voz, medidor, cardápio) gera um span com modelo, tokens de entrada, saída e cache, latência, custo estimado, feature e pessoa; dashboard de tokens e custo por feature e por semana; orçamento mensal com alerta e corte automático | Convenções de GenAI do OpenTelemetry, FinOps de IA, cache de prompt, custo por unidade de valor (por cupom lido, por cardápio gerado) | Tabela de chamadas no Postgres com tokens e custo, e uma consulta mensal |
| RNT-24 Telemetria dos agentes de desenvolvimento | Uso e custo do Claude Code, do revisor de PR e do agente de operação exportados por OpenTelemetry para o mesmo Grafana do Ninho: tokens e custo por sessão, por PR e por issue, ao lado do DORA do repositório | Medir o custo de construir com agentes; a dimensão custo do DX AI Measurement Framework com dados reais | Anotar o custo da sessão em cada PR |

**Stack de referência:** React como PWA na Vercel; app Android (Kotlin ou React Native) instalado fora da loja; NestJS no AWS Lightsail (única peça paga); PostgreSQL no Supabase usado só como banco (Data API desligada ou RLS ligado); MongoDB Atlas; Redis no Upstash; Amazon SNS e SQS (LocalStack em desenvolvimento); Notificações em Go; e-mail por provedor gratuito; backups no Cloudflare R2; Socket.IO; Web Push e e-mail; OpenTelemetry com Grafana; Sentry; PostHog; GitHub Actions. Localmente, tudo sobe com Docker Compose.

**ADRs previstos:** hospedagem e infraestrutura (feito: [ADR 0002](adr/0002-hospedagem-e-infraestrutura.md)); hexagonal; conflito offline (last-write-wins ou CRDT); extração de Notificações; versão completa × simplificada, com as medições lado a lado; método de detecção de falhas no Consumo; front (PWA mais app Android de captura).

## Integrações e dependências

| Integração | Usada em | Limites e condições | Plano B |
| --- | --- | --- | --- |
| Google Calendar | RF-A-02 | App OAuth em modo de teste: tokens de atualização expiram em 7 dias; cotas por minuto; os canais de webhook expiram e precisam ser renovados | Reautorização avisada no app; sincronização periódica sem webhook |
| Acesso a notificações do Android | RF-G-05 | Permissão dada à mão; Android 13 ou mais novo restringe essa permissão para apps de fora da loja; economia de bateria pode parar o serviço | Anotação manual (MVP) |
| Push dos apps dos bancos | RF-G-06 | Sem contrato: o texto muda sem aviso | Leitor por banco com testes; RF-G-08 |
| Mobills | RF-G-09 | Sem API; só relatório exportado em arquivo | Conciliação manual |
| Web Push | RF-N-01 | Precisa da permissão do navegador; fora do Chrome do Android, a entrega é menos confiável | E-mail (RF-N-05) |
| Provedor de e-mail (Resend ou Brevo) | RF-N-01 | Plano gratuito com limite diário | Só push |
| Provedor de IA | RNT-22 | Custo por token e limites de taxa; orçamento do RNF-P-11 | Feature desligada por flag; entrada manual |
| AWS (Lightsail, SNS, SQS) | Backend e fila | Orçamento de US$ 10 por mês; cota gratuita da SQS e do SNS ([ADR 0002](adr/0002-hospedagem-e-infraestrutura.md)) | Versões simplificadas dos RNT (fila no Postgres) |
| Planos gratuitos (Vercel, Supabase, MongoDB Atlas, Upstash, Cloudflare R2) | Tudo | Pausam e têm cotas (ver Premissas e riscos) | Docker Compose local; troca de adaptador (RNT-02) |

## Plataformas suportadas

- **App Android de captura:** Android 10 ou mais novo, a confirmar com a versão dos três celulares.
- **PWA no celular:** Chrome no Android, que é o que a família usa; telas a partir de 360 px de largura.
- **PWA no computador:** as duas últimas versões do Chrome, Edge e Firefox.
- **Fora:** iPhone e Safari, porque ninguém da família usa (ver Fora de escopo).

## Fora de escopo

- Publicação em loja de apps e uso por outras famílias.
- Escala real (a de 10 mil famílias é só estimativa e teste sintético).
- Classificação de gastos e controle financeiro completo, que ficam no Mobills e no dashboard da skill /dashboard-financeiro; o Finanças completo é projeto futuro.
- Captura de outros bancos ou de iPhone.

## Premissas e riscos

| Risco ou premissa | Impacto | Mitigação |
| --- | --- | --- |
| A família não adota o Ninho ou não responde "o que foi isso?" (risco de valor, o maior) | O app vira mais uma coisa para manter, e as métricas de uso não saem do zero | Conversas com a família antes de cada módulo (`docs/discovery/`); MVP manual antes da captura automática; anotação e uso semanal medidos desde o primeiro mês; módulo sem uso por 4 semanas volta para o passo 1 do processo |
| O Android restringe o acesso a notificações para apps instalados fora da loja | A captura automática não funciona sem configuração | Passo a passo de instalação com a liberação manual das configurações restritas; testar nos 3 celulares antes da captura automática |
| Os bancos mudam o texto do push | Notificações não lidas | Leitor por banco com testes sobre exemplos reais anonimizados; o que falhar vai para a DLQ e aparece para o administrador (RF-G-08) |
| Supabase gratuito pausa após uma semana sem atividade | Projeto pausado | Tarefa agendada que mantém o banco ativo; app Android guarda e reenvia |
| Limites de planos gratuitos de terceiros mudam ou são atingidos (cota da SQS, Upstash, e-mail) | Fila, cache ou e-mail param | Limites conferidos antes do deploy e monitorados; troca de adaptador pela porta (RNT-02); AWS só se não houver outra opção gratuita |
| Conta da AWS passa do orçamento | Custo inesperado | Alerta no AWS Budgets em 50%, 80% e 100% de US$ 10 |
| Sem backup no plano gratuito do Supabase | Perda de dados | Dump semanal criptografado pelo GitHub Actions para o Cloudflare R2 (RNF-P-10) |
| Limites do app OAuth do Google em modo de teste | Sincronização para depois de alguns dias | Verificar, ao construir a Agenda, o estado de publicação e a validade do token; ADR com a escolha |
| Excesso de engenharia para 3 a 4 horas por semana | O módulo não fica pronto e a família não usa | A versão simplificada de cada requisito técnico é o plano B; o MVP funcional vem antes da peça extra |
| Dados reais num repositório público | Vazamento | Seeds e fixtures fictícios; segredos no ambiente; revisão no modelo de ameaças |

## Roadmap

| Etapa | Entrega |
| --- | --- |
| 1 | Repositório, CLAUDE.md, primeira skill, login, família, membros e papéis |
| 2 | Este PRD revisado, design doc da Memória de gastos, primeiros ADRs |
| 3 | Memória de gastos (MVP) com spec, testes antes do código, revisor no CI e observabilidade básica |
| 4 | Rotina (MVP), captura automática no Android, fila, cache, tempo real e offline |
| 5 | Agenda e lembretes; Notificações extraído como serviço; outbox, Google Calendar, CQRS leve; versão simplificada e ADR comparativo |
| 6 | Refatoração com DDD |
| 7 | Consumo: migração, detecção de falhas, SLOs, carga sintética e teste de falha |
| 8 | ADR do front, contexto e modelo de ameaças, agentes revisor e de operação, DORA |
| 9 | Despensa (MVP) com processo de produto e métricas de negócio |
| 10 | IA no Ninho com evals |


## Perguntas em aberto

1. ~~O dependente usa o app com conta própria desde o início?~~ Decidido: sim, com o papel de Dependente.
2. ~~Fila: RabbitMQ ou BullMQ?~~ Decidido: Amazon SNS + SQS, na cota gratuita ([ADR 0002](adr/0002-hospedagem-e-infraestrutura.md)).
3. Conflito offline nas listas: last-write-wins por campo ou CRDT?
4. Provedor de e-mail com plano gratuito para os lembretes: Resend ou Brevo. Escolha final ao construir a Agenda.
5. Formato do relatório do Mobills para a importação da fase 3.
6. ~~Onde guardar os backups semanais?~~ Decidido: Cloudflare R2 gratuito, com dump criptografado ([ADR 0002](adr/0002-hospedagem-e-infraestrutura.md)).
7. Login: conta Google ou e-mail com link mágico? Decidir em ADR antes do login.
8. Versão do Android dos três celulares, para fechar a versão mínima do app de captura.
