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
| Fila | RabbitMQ gerenciado no plano gratuito da CloudAMQP | Mantém o RabbitMQ do case. Mensagens com atraso e retry por TTL e dead-letter exchange; DLQ nativa. Resolve a pergunta em aberto 2 do PRD a favor do RabbitMQ |
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
- **Hexagonal:** fila, e-mail e armazenamento de backup ficam atrás de portas (RNT-02). Localmente, o Docker Compose sobe RabbitMQ, Postgres, MongoDB e Redis, e a versão simplificada continua sendo só trocar o adaptador para a tabela de jobs no Postgres.
- **Segredos:** nas variáveis de ambiente de cada plataforma, nunca no repositório.
- **Revisão da regra:** se um plano gratuito acabar ou deixar de atender, a peça vai para outro plano gratuito. Ela só vai para a AWS se não houver nenhum, e isso vira um ADR novo.

## Consequências

- O RNF-P-11 deixa de ser "R$ 0 por mês" e passa a ser "até US$ 10 por mês, só o backend na AWS".
- O RNF-P-07 não precisa mais aceitar backend dormindo nem a primeira resposta de 60 s.
- Monólito e Notificações dividem uma instância de 1 GB. A extração continua real (processo, linguagem, deploy e banco separados), mas uma queda da instância derruba os dois. Isso fica explícito no ADR de extração.
- Em linguagens diferentes, os dois serviços só conversam por contrato (eventos na fila), o que deixa a fronteira explícita. O custo é manter duas toolchains no CI.
- O Ninho passa a depender de vários planos gratuitos de terceiros. Os limites deles (mensagens da CloudAMQP, comandos do Upstash, envios do provedor de e-mail) mudam com o tempo, devem ser conferidos antes do primeiro deploy e entram nos alertas de observabilidade.
- A AWS fica com uso mínimo. O estudo de AWS do programa pode usar outros recursos à parte, sem entrar no caminho de produção do Ninho.
