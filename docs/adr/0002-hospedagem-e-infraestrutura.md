# 2. Hospedagem e infraestrutura

Data: 26/11/2026

## Status

Aceita

## Contexto

O case previa tudo em planos gratuitos: front na Vercel, backend NestJS no Render, Postgres no Supabase, MongoDB Atlas, Redis no Upstash e a fila em RabbitMQ (no Docker) ou BullMQ (sobre o Redis). Ao fechar os requisitos de infraestrutura, três problemas apareceram:

1. **O backend dorme.** O Render gratuito desliga depois de 15 min sem requisição. O WebSocket da Rotina e a captura do Android precisam de um backend acordado. Pior, o RNF-P-02 (lembrete entregue até 1 min depois do horário) não pode depender de alguém ter aberto o app antes.
2. **A fila não tem onde rodar.** O RabbitMQ no Docker funciona só na máquina de desenvolvimento, e manter um broker em produção é mais um serviço para hospedar.
3. **O BullMQ gasta a cota do Redis.** Ele consulta o Redis o tempo todo, mesmo com a fila vazia, e isso consome os comandos do plano gratuito do Upstash.

O programa também já prevê estudar a AWS com o Claude.

## Decisão

Usar uma hospedagem híbrida. Na AWS ficam o backend e as peças que têm plano gratuito permanente. O resto continua nos planos gratuitos que o case já previa.

| Peça | Onde | Observação |
| --- | --- | --- |
| Front (PWA) | Vercel, plano gratuito | Sem mudança |
| Monólito NestJS | AWS Lightsail, plano de 1 GB sempre ligado (cerca de US$ 7 por mês, com IPv4 e tráfego incluídos) | Única peça paga. EC2 foi descartado por ora: uma t4g.micro mais o IPv4 público cobrado à parte fica perto de US$ 10 e encosta no teto. Migrar para EC2 é possível se o estudo de AWS pedir |
| Fila | Amazon SQS com DLQ nativa | Substitui RabbitMQ e BullMQ em produção. Resolve a escolha da pergunta em aberto 2 do PRD |
| Mensagens com atraso e lembretes | Amazon EventBridge Scheduler, um agendamento por lembrete | O horário é disparado pela AWS, sem depender do backend acordado. O atraso da SQS (máximo 15 min) serve só para retry com backoff |
| Serviço de Notificações (extraído no F1 M5.1) | AWS Lambda em Go consumindo a SQS | Go pelo estudo da linguagem e pelo encaixe em Lambda (binário pequeno, cold start rápido, pouca memória); o monólito segue em NestJS, a stack do curso. Banco próprio exigido pelo RNT-11; DynamoDB é o candidato natural, decidido no ADR de extração |
| Postgres | Supabase, plano gratuito | Pausa depois de 7 dias sem atividade; o uso semanal da família e uma tarefa agendada evitam isso |
| MongoDB | Atlas, plano gratuito | Sem mudança |
| Redis | Upstash, plano gratuito | Só cache, placar e rate limit, sem fila |
| Backups | Bucket S3 privado | Fecha a pergunta em aberto 6 do PRD |
| E-mail dos lembretes | Amazon SES, se o custo couber no orçamento; senão, um provedor com plano gratuito | Pergunta em aberto 4 do PRD, decidida no F1 M5.1 |
| Observabilidade | Grafana Cloud, Sentry e PostHog nos planos gratuitos | Sem mudança |
| CI/CD | GitHub Actions, com deploy na AWS por OIDC, sem chave de acesso guardada no GitHub | Entra junto com o código (RNT-21) |

Regras que valem para todas as peças:

- **Custo:** orçamento de até US$ 10 por mês, com alerta no AWS Budgets em 50%, 80% e 100%.
- **Região:** banco, backend e fila na mesma região, para a latência entre eles não estourar o RNF-P-06. A escolha entre sa-east-1 (mais perto da família) e us-east-1 (mais barata) é feita no primeiro deploy.
- **Infraestrutura como código:** os recursos da AWS são criados por código versionado no repositório (ferramenta escolhida no F1 M3), nunca à mão no console.
- **Hexagonal:** fila e agendador ficam atrás de portas (RNT-02). Localmente, o Docker Compose sobe adaptadores locais (LocalStack ou um adaptador em memória), e a versão simplificada continua sendo só trocar o adaptador para a tabela de jobs no Postgres.
- **Segredos:** no AWS Systems Manager Parameter Store ou nas variáveis de ambiente da plataforma, nunca no repositório.

## Consequências

- O RNF-P-11 deixa de ser "R$ 0 por mês" e passa a ser "até US$ 10 por mês, só o backend pago". A restrição do case muda de "tudo grátis" para "grátis, exceto o backend".
- O RNF-P-07 não precisa mais aceitar backend dormindo nem a primeira resposta de 60 s.
- A fila de produção passa a ser gerenciada. O estudo de broker (RabbitMQ) sai do caminho principal. Se quiser estudá-lo, ele pode entrar só em desenvolvimento, atrás da mesma porta.
- A Lambda de Notificações dá ao F1 M5.1 um deploy realmente independente do monólito. Por estar em outra linguagem, ela só conversa com o monólito por contrato (eventos na SQS), o que deixa a fronteira explícita. O custo é manter duas toolchains no CI.
- Aparece um risco novo, a conta da AWS. Ele é mitigado pelo alerta de orçamento e pela infraestrutura como código, que permite destruir e recriar o ambiente.
- Os limites e preços dos planos gratuitos (créditos de conta nova, cota permanente da SQS, da Lambda e do EventBridge Scheduler) mudam com o tempo e devem ser conferidos no console antes do primeiro deploy.
