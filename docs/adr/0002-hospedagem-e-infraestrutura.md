# 2. Hospedagem e infraestrutura

Data: 26/11/2026

## Status

Aceita

## Contexto

O case previa tudo em planos gratuitos: front na Vercel, backend NestJS no Render, Postgres no Supabase, MongoDB Atlas, Redis no Upstash e a fila em RabbitMQ (no Docker) ou BullMQ (sobre o Redis). Ao fechar os requisitos de infraestrutura, três problemas apareceram:

1. **O backend dorme.** O Render gratuito desliga depois de 15 min sem requisição. O WebSocket da Rotina e a captura do Android precisam do backend acordado, e o RNF-P-02 (lembrete entregue até 1 min depois do horário) não pode depender de alguém ter aberto o app antes.
2. **A fila não tem onde rodar.** O RabbitMQ no Docker funciona só na máquina de desenvolvimento.
3. **O BullMQ gasta a cota do Redis.** Ele consulta o Redis o tempo todo, mesmo com a fila vazia, e isso consome os comandos do plano gratuito do Upstash.

## Decisão

**Regra:** cada peça fica num plano gratuito. A AWS entra somente onde não existe opção gratuita que atenda ao requisito, e é o único ambiente pago aceito, com teto de US$ 10 por mês.

Aplicando a regra, a única peça sem opção gratuita adequada é o backend sempre ligado. As alternativas gratuitas ou dormem (Render) ou foram descartadas (Oracle Cloud Always Free, pelo cadastro e pelo risco de recuperação de VM ociosa).

| Peça | Onde | Observação |
| --- | --- | --- |
| Front (PWA) | Vercel, plano gratuito | Sem mudança |
| Monólito NestJS | **AWS Lightsail**, plano de 1 GB sempre ligado (cerca de US$ 7 por mês, com IPv4 e tráfego incluídos) | Única peça paga. Com o backend sempre ligado, os lembretes saem do próprio backend, sem agendador externo |
| Serviço de Notificações (extraído no F1 M5.1) | Container separado em Go, na mesma instância do Lightsail | Go pelo estudo da linguagem. Deploy e processo independentes do monólito, sem custo extra; banco próprio exigido pelo RNT-11, decidido no ADR de extração |
| Mensageria | **Amazon SNS** (tópicos de eventos) e **Amazon SQS** (uma fila por consumidor, cada uma com DLQ), dentro da cota gratuita permanente da AWS | Escolha do mantenedor. Não é recurso pago: SNS e SQS têm cota gratuita permanente (1 milhão de publicações e 1 milhão de requisições por mês). Desenho em "Mensageria" abaixo. Resolve a pergunta em aberto 2 do PRD |
| Postgres | Supabase, plano gratuito | Pausa depois de 7 dias sem atividade; o uso semanal e uma tarefa agendada evitam isso |
| MongoDB | Atlas, plano gratuito | Sem mudança |
| Redis | Upstash, plano gratuito | Só cache, placar e rate limit, sem fila |
| Backups | Cloudflare R2, plano gratuito (10 GB), com o dump criptografado antes do envio | Fecha a pergunta em aberto 6 do PRD |
| E-mail dos lembretes | Provedor com plano gratuito (Resend ou Brevo) | Fecha a pergunta em aberto 4, com a escolha final no F1 M5.1 |
| Observabilidade | Grafana Cloud, Sentry e PostHog nos planos gratuitos | Sem mudança |
| CI/CD | GitHub Actions, com deploy no Lightsail | Entra junto com o código (RNT-21) |

Regras que valem para todas as peças:

- **Custo:** alerta no AWS Budgets em 50%, 80% e 100% de US$ 10. Os planos gratuitos de terceiros não podem ter cartão que permita cobrança automática acima do gratuito.
- **Região:** serviços na mesma região ou na mais próxima possível, para a latência entre eles não estourar o RNF-P-06. A região é escolhida no primeiro deploy.
- **Hexagonal:** fila, e-mail e armazenamento de backup ficam atrás de portas (RNT-02). Localmente, o Docker Compose sobe Postgres, MongoDB, Redis e o LocalStack (SNS e SQS), e a versão simplificada continua sendo só trocar o adaptador para a tabela de jobs no Postgres.
- **Segredos:** nas variáveis de ambiente de cada plataforma, nunca no repositório.
- **Revisão da regra:** se um plano gratuito acabar ou deixar de atender, a peça vai para outro plano gratuito. Ela só vai para a AWS se não houver nenhum, e isso vira um ADR novo.

## Mensageria

O padrão é **fan-out SNS → SQS**: cada contexto publica os eventos dele num tópico SNS, e cada consumidor tem a própria fila SQS assinando os tópicos que interessam, com filter policy por tipo de evento. Quem publica não sabe quem consome. Todas as filas são **Standard** (at-least-once), o que exige consumidor idempotente (RNT-08).

**Tópicos (SNS), um por contexto que publica**

| Tópico | Publicado por | Eventos |
| --- | --- | --- |
| `ninho-conciliacao` | Conciliação (monólito) | `CompraCapturada`, `PerguntaCriada` ("o que foi isso?"), `LancamentoConciliado` |
| `ninho-rotina` | Rotina (monólito) | `ItemComprado`, `TarefaConcluida` |
| `ninho-despensa` | Despensa (monólito) | `ItemAcabou` |
| `ninho-agenda` | Agenda (monólito) | `LembreteVencido` |
| `ninho-consumo` | Consumo (monólito) | `LeituraRegistrada`, `AlertaConsumo` |

**Filas (SQS), uma por consumidor, cada uma com a sua DLQ**

| Fila | Consumidor | Assina | Faz |
| --- | --- | --- | --- |
| `conciliacao-compras` | Conciliação (monólito) | `ninho-conciliacao`, filtro `CompraCapturada` | Leitor por banco (Itaú ou Nubank) extrai valor, estabelecimento e cartão e cria a pergunta |
| `notificacoes-entregas` | Notificações (serviço Go) | `PerguntaCriada`, `LembreteVencido`, `AlertaConsumo` | Aplica as preferências de canal e entrega Web Push e e-mail |
| `rotina-itens-acabados` | Rotina (monólito) | `ninho-despensa`, filtro `ItemAcabou` | Põe o item na lista de compras certa |
| `despensa-itens-comprados` | Despensa (monólito) | `ninho-rotina`, filtro `ItemComprado` | Volta o item para "tem" |
| `consumo-deteccao` | Consumo (monólito) | `ninho-consumo`, filtro `LeituraRegistrada` | Roda escore z e CUSUM e publica `AlertaConsumo` se for o caso |

**Regras**

- **Outbox (RNT-07):** nenhum módulo publica direto no SNS. O evento é gravado na tabela de outbox do Postgres na mesma transação da mudança, e um relay no monólito publica no tópico e marca como enviado.
- **Retry com backoff e jitter:** na falha, o consumidor aumenta o visibility timeout da mensagem (exponencial com jitter). Depois de 5 recebimentos (`maxReceiveCount`), a SQS move a mensagem para a DLQ. Uma DLQ com mensagem dispara alarme, e o reprocessamento é feito por redrive da DLQ para a fila de origem.
- **Lembretes no horário:** o atraso nativo da SQS vai só até 15 min, então não serve para agendar. A Agenda guarda os lembretes no Postgres, e um agendador no monólito procura a cada 30 s os que venceram (`SELECT … FOR UPDATE SKIP LOCKED`) e publica `LembreteVencido` pela outbox. O lembrete com confirmação é reagendado até alguém marcar "feito".
- **Idempotência:** cada evento carrega um id único, e cada consumidor grava os ids já processados antes de confirmar (apagar) a mensagem.
- **O SNS não entrega aos usuários:** ele é só o barramento entre contextos. Web Push (VAPID) e e-mail saem do serviço de Notificações, porque o SNS não faz Web Push de PWA, e o e-mail dele exige confirmação de assinatura por endereço.
- **Contrato:** os eventos são versionados (`tipo`, `versao`, `id`, `ocorridoEm`, `dados`), e o esquema fica no repositório, compartilhado entre NestJS e Go.

## Consequências

- O RNF-P-11 deixa de ser "R$ 0 por mês" e passa a ser "até US$ 10 por mês, só o backend na AWS".
- O RNF-P-07 não precisa mais aceitar backend dormindo nem a primeira resposta de 60 s.
- Monólito e Notificações dividem uma instância de 1 GB. A extração continua real (processo, linguagem, deploy e banco separados), mas uma queda da instância derruba os dois. Isso fica explícito no ADR de extração.
- Em linguagens diferentes, os dois serviços só conversam por contrato (eventos na fila), o que deixa a fronteira explícita. O custo é manter duas toolchains no CI.
- A cota de 1 milhão de requisições da SQS é gasta mais pelo polling do que pelas mensagens: cada fila com long polling de 20 s faz cerca de 130 mil requisições por mês, mesmo vazia. As 5 filas somam cerca de 650 mil, e por isso uma fila nova só entra com essa conta refeita. As DLQs não são consultadas em polling; só um alarme lê a profundidade delas.
- O estudo de broker muda de RabbitMQ (exchanges, bindings) para pub/sub gerenciado (fan-out SNS → SQS, filter policies, visibility timeout). Os conceitos do RNT-06 (at-least-once, DLQ, retry, idempotência) continuam todos.
- O Ninho passa a depender de vários planos gratuitos. Os limites deles (cota da SQS e do SNS, comandos do Upstash, envios do provedor de e-mail) mudam com o tempo, devem ser conferidos antes do primeiro deploy e entram nos alertas de observabilidade.
- A AWS fica com uso mínimo. O estudo de AWS do programa pode usar outros recursos à parte, sem entrar no caminho de produção do Ninho.
