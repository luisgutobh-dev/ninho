# Processo de construção do Ninho

Como o Ninho é construído, do problema à operação. Vale para pessoas e para agentes: antes de começar qualquer trabalho, o agente lê este arquivo e o `CLAUDE.md`.

**Princípio:** quanto mais cedo no processo, mais a decisão é humana (o quê, por quê, limites). Quanto mais tarde, mais o agente executa, dentro dos trilhos definidos antes.

**Regras gerais**

- Fatias pequenas: uma story por vez, um PR por story.
- Harness antes de escala: nada de agentes em paralelo sem testes, sensores e revisor no CI.
- Toda decisão que alguém pode questionar vira ADR em `docs/adr/`.
- Nenhum dado real da família no repositório. Logs mascarados (ver `docs/prd.md`, RNT-15).

---

## 1. Problema e evidência

- **O quê:** quem sofre, em que situação, como saberemos que resolvemos.
- **Como:** conversas curtas com a família; dados que já existem (Mobills, painel de consumo).
- **IA:** sintetiza as conversas, mantendo as citações como evidência. Não inventa a dor.
- **Saída:** problem statement e métricas de sucesso, no topo do PRD do módulo.
- **Referências:** *The Mom Test* (Rob Fitzpatrick); *The Four Big Risks* (Marty Cagan, SVPG).

## 2. PRD

- **O quê:** requisitos funcionais (com IDs `RF-*`), não funcionais de produto (`RNF-P-*`), técnicos de estudo (`RNT-*`), fora de escopo, riscos e perguntas em aberto.
- **IA:** sparring. Pedir riscos, contra-argumentos e casos de borda; não pedir o texto pronto.
- **Saída:** `docs/prd.md` e, por módulo, um PRD curto em `docs/prd/`.
- **Para features de IA:** o critério de qualidade e o limite de autonomia entram aqui.

## 3. Protótipo descartável

- **O quê:** um protótipo clicável do fluxo principal, mostrado para quem vai usar antes de qualquer arquitetura.
- **IA:** gera o protótipo (Lovable ou Claude).
- **Saída:** prints e o que mudou no PRD. O código do protótipo é jogado fora.

## 4. Domínio (DDD estratégico)

- **O quê:** linguagem ubíqua, subdomínios (núcleo, suporte, genérico), bounded contexts e o mapa de contextos.
- **Como:** event storming (mesmo que só uma pessoa, em post-its ou no Miro); bounded context canvas para cada contexto.
- **Saída:** `docs/dominio/glossario.md` e `docs/dominio/mapa-de-contextos.md`.
- **Referências:** *Learning Domain-Driven Design* (Vlad Khononov), parte I; livro *Full Cycle* (DDD); [EventStorming cheat sheet](https://github.com/ddd-crew/eventstorming-glossary-cheat-sheet), [Bounded Context Canvas](https://github.com/ddd-crew/bounded-context-canvas) e [Context Mapping](https://github.com/ddd-crew/context-mapping) (DDD Crew).

## 5. Arquitetura e design doc

- **O quê:**
  - C4 de contexto, contêineres e componentes;
  - estilo: monólito modular, um módulo por bounded context, hexagonal dentro de cada um;
  - cada requisito não funcional vira uma decisão (fila, cache, SQL ou NoSQL, tempo real), com estimativa de capacidade;
  - modelo de ameaças.
- **IA:** propõe alternativas e aponta riscos; a escolha e o ADR são humanos.
- **Saída:** design doc em `docs/design/`, ADRs em `docs/adr/`, diagramas C4 em `docs/arquitetura/`, `docs/seguranca/modelo-de-ameacas.md`.
- **Referências:** [C4 model](https://c4model.com); [Hexagonal architecture (Alistair Cockburn)](https://alistair.cockburn.us/hexagonal-architecture/); [Documenting Architecture Decisions (Michael Nygard)](https://cognitect.com/blog/2011/11/15/documenting-architecture-decisions); [System Design Primer](https://github.com/donnemartin/system-design-primer); [microservices.io patterns](https://microservices.io/patterns/index.html); [Modular Monolith with DDD (Kamil Grzybek)](https://github.com/kgrzybek/modular-monolith-with-ddd); [OWASP Threat Modeling Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Threat_Modeling_Cheat_Sheet.html).

## 6. Fundação: harness e walking skeleton

Antes da primeira feature, o repositório precisa ter:

- `CLAUDE.md` e `AGENTS.md`, constitution (Spec Kit) e `docs/definition-of-done.md`;
- skills do projeto em `.claude/skills/`;
- CI no GitHub Actions: testes, lint, fitness functions (dependency-cruiser), revisor automático e teto de custo por execução de agente;
- observabilidade básica: OpenTelemetry, política de logs com mascaramento, telemetria de tokens do Claude Code;
- feature flags e ambientes (local com Docker Compose, produção).

Depois, um **walking skeleton**: uma fatia fina de ponta a ponta já no ar (login → família → tela inicial vazia), com deploy automático a cada merge. Toda feature nasce em cima dele.

- **Referências:** [Claude Code: Best practices](https://www.anthropic.com/engineering/claude-code-best-practices); [Spec Kit](https://github.com/github/spec-kit); [AgileSpec](https://github.com/mtov/AgileSpec); [Learn Harness Engineering](https://walkinglabs.github.io/learn-harness-engineering/pt-BR/); [dependency-cruiser](https://github.com/sverweij/dependency-cruiser); [OpenTelemetry para Node.js](https://opentelemetry.io/docs/languages/js/getting-started/nodejs/); [Claude Code: Monitoring usage](https://docs.claude.com/en/docs/claude-code/monitoring-usage); *Growing Object-Oriented Software, Guided by Tests* (Freeman e Pryce), sobre o walking skeleton.

## 7. Backlog

- **O quê:** stories por bounded context, como issues do GitHub com critérios de aceite, label de contexto e de tipo, milestone do módulo do programa.
- **Priorização:** RICE; o que não tem métrica de sucesso não entra.
- **Saída:** quadro "Ninho" no GitHub Projects (Backlog → Pronto para o agente → Em andamento → Em revisão → Feito).

## 8. Ciclo de cada story

1. **Spec da story:** critérios de aceite (casos normais e de borda) e design, no formato do Spec Kit ou do AgileSpec.
2. **Testes antes do código**, com DDD tático: agregados, invariantes, eventos de domínio. Pirâmide: muitos testes de domínio, integração com containers reais, contrato entre front e API, poucos E2E.
3. **O agente implementa**, pedindo esclarecimento quando a spec conflitar com as specs de projeto.
4. **Sensores:** testes, lint, fitness functions, revisor automático e teto de custo. Vermelho em qualquer um é trabalho não terminado.
5. **Revisão humana** com a rubrica de review, e merge.
6. **Deploy** atrás de feature flag.
7. **Consolidação:** as decisões da story atualizam ADRs, glossário, specs de projeto e `CLAUDE.md`.

- **Referências:** [The Practical Test Pyramid (Ham Vocke)](https://martinfowler.com/articles/practical-test-pyramid.html); [Engrave your SDLC (Davi de Castro Reis)](https://daviws.medium.com/engrave-your-sdlc-04b428a40dcf); [Feature Toggles (Pete Hodgson)](https://martinfowler.com/articles/feature-toggles.html).

## 9. Lançamento

- Checklist de lançamento; release notes para a família.
- Liberação gradual por feature flag, pessoa a pessoa, com o que observar em cada etapa e o critério do kill switch.

## 10. Operação

- SLOs com alerta e error budget (ver PRD, RNF-P-01 a 03).
- Runbook para fila parada, DLQ cheia e integração fora do ar.
- Postmortem sem culpados para todo incidente real, mesmo pequeno.
- Agente de operação: investiga e propõe; ação irreversível só com aprovação humana.
- **Referências:** [Google SRE: Service Level Objectives](https://sre.google/sre-book/service-level-objectives/); [SRE Workbook: Implementing SLOs](https://sre.google/workbook/implementing-slos/); [Postmortem culture](https://sre.google/sre-book/postmortem-culture/).

## 11. Medição e aprendizado

- **Produto:** uso semanal, funil e retenção por módulo (PostHog); métricas de negócio do PRD.
- **Engenharia:** DORA do repositório; custo em tokens por PR e por módulo.
- **Saída:** relatório de impacto de 1 página por módulo, um mês depois do lançamento. O que ele mostrar volta para o passo 1.

---

## Features de IA: o que muda

- **Passo 2:** critério de qualidade e limite de autonomia no PRD; o conjunto dourado de evals nasce antes do código.
- **Passo 5:** o modelo fica atrás de uma porta, separado da lógica de negócio; dados privados, conteúdo não confiável e capacidade de agir nunca juntos no mesmo agente.
- **Passos 6 e 10:** tracing com as convenções de GenAI do OpenTelemetry, guardrails, versionamento de prompt e modelo, orçamento mensal com alerta.
- **Referências:** [Demystifying evals for AI agents](https://www.anthropic.com/engineering/demystifying-evals-for-ai-agents); [Your AI product needs evals (Hamel Husain)](https://hamel.dev/blog/posts/evals); [OpenTelemetry GenAI](https://opentelemetry.io/docs/specs/semconv/gen-ai/); [OWASP Top 10 for LLM](https://genai.owasp.org/llm-top-10/).
