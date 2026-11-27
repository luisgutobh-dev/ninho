# Roteiro de conversas com a família

Roteiro das conversas do passo 1 do processo (Problema e evidência, ver `docs/processos.md`), no formato do *The Mom Test*: perguntas sobre fatos recentes da rotina, nunca sobre a ideia do app. Serve ao entregável "3 conversas com a família + síntese" e alimenta o PRD (`docs/prd.md`).

> Repositório público: nas anotações, use papéis (esposa, filho, eu), sem nomes nem dados pessoais. Nada de valores, estabelecimentos ou informações de saúde identificáveis.

## Como conduzir

- **Não apresente o Ninho.** Nada de "estou fazendo um app que...". A conversa é sobre como a casa funciona hoje. Se perguntarem, conte depois.
- **Fatos do passado, não opinião.** Troque "você usaria?" por "qual foi a última vez que...?". Quando vier "eu sempre" ou "normalmente", peça um caso concreto.
- **Fale pouco.** Depois de cada resposta, "e aí, o que você fez?" ou "me conta mais" rende mais que a próxima pergunta.
- **Curto e informal.** De 15 a 20 minutos, num momento tranquilo, um módulo ou dois por vez. Melhor três conversas curtas que uma longa.
- **Separadamente.** Converse com a esposa e com o filho em momentos diferentes, para um não responder pelo outro.
- **Anote citações literais.** São a evidência da síntese. Suas interpretações vão numa coluna separada.

## Conversa 1: com a esposa

### Memória de gastos

1. No fechamento do mês passado no Mobills, teve lançamento que vocês não lembravam o que era? Quantos, mais ou menos? O que fizeram com eles?
2. Me conta a última compra no cartão que você teve que descobrir depois o que era. Como descobriu?
3. Hoje, quando você faz uma compra, guarda alguma informação dela (cupom, mensagem, nota mental)? Onde?
4. Quando chega a notificação do banco de uma compra, o que você faz com ela?
5. Quanto tempo vocês gastaram na última vez para conciliar o mês? O que mais atrasou?

**A pergunta que dá medo:** se o app perguntasse "o que foi isso?" logo depois de cada compra, você acha que responderia? Peça um caso: na última semana, em que momento uma pergunta dessas teria chegado e o que você estava fazendo?

### Rotina (tarefas e listas de compras)

1. Qual foi a última tarefa da casa que ficou sem fazer porque cada um achou que era do outro? O que aconteceu?
2. Como vocês combinaram quem faz o quê na semana passada? Funcionou?
3. Na última ida ao mercado, como você soube o que comprar? Esqueceu alguma coisa? Como descobriu?
4. Hoje, onde ficam as listas de compras (papel, WhatsApp, cabeça)? Quem escreve e quem compra?
5. Já aconteceu de dois de vocês comprarem a mesma coisa, ou ninguém comprar? Quando foi a última vez?

### Agenda e lembretes

1. Qual foi o último compromisso da família que alguém esqueceu ou soube em cima da hora (escola, médico, aniversário)? Como foi?
2. Onde você anota os compromissos hoje? E os do filho?
3. Quando vocês precisam saber a agenda um do outro, como fazem?
4. Qual lembrete você já ignorou e depois se arrependeu? Por que ignorou?

### Consumo (água e energia)

1. Quando chegou a última conta de água ou de luz, você olhou o valor? O que pensou?
2. Já teve uma conta que veio alta e vocês foram descobrir o motivo depois? Como foi?
3. Se um vazamento começasse hoje, como vocês perceberiam?

### Despensa

1. Qual foi a última coisa que faltou em casa na hora em que você precisou? Como descobriu que tinha acabado?
2. Quem percebe primeiro quando algo está acabando? E o que essa pessoa faz?
3. Na semana passada, como vocês decidiram o que comer? Alguém reclamou de alguma coisa?
4. Tem algo que alguém da família não come ou não pode comer? Como isso entra na hora de comprar e cozinhar hoje?

**Fechamento:** "Se eu fizer um teste simples disso nas próximas duas semanas (um papel na geladeira, um grupo no WhatsApp), você topa usar?" O compromisso de tempo vale mais que qualquer elogio.

## Conversa 2: com o filho

Linguagem simples e tom de conversa, nada de entrevista. Melhor durante uma atividade (no carro, num lanche) do que sentado à mesa. Ele não participa da Memória de gastos nem do Consumo.

### Rotina

1. Qual foi a última tarefa de casa que você fez? Alguém pediu ou você lembrou sozinho?
2. Teve alguma tarefa essa semana que você esqueceu? O que aconteceu?
3. Como você sabe o que é sua tarefa e o que é dos outros?
4. Se a família tivesse um placar de quem ajudou mais na semana, como você acha que estaria nessa semana? Por quê? (Observe se a ideia de pontos anima ou incomoda.)

### Agenda e lembretes

1. Qual foi a última coisa da escola que você esqueceu ou lembrou em cima da hora (trabalho, prova, material)? Quem te lembrou?
2. Como você fica sabendo dos compromissos da semana?
3. Você usa algum alarme ou lembrete no celular? Pra quê?

### Despensa

1. Qual foi a última vez que você foi pegar alguma coisa e tinha acabado? O que você fez?
2. Quando acaba algo que você gosta, você avisa alguém? Como?
3. Tem alguma comida que você queria que tivesse mais em casa? E alguma que você não gosta?

**Fechamento:** "Se a gente tivesse um jeito fácil de avisar que algo acabou, você me ajuda a testar por duas semanas?"

## Conversa 3: os três juntos

Depois das duas conversas individuais, uma conversa curta em família sobre o que apareceu em comum.

1. "Vocês dois contaram que [um problema que apareceu nas duas conversas]. Isso aconteceu de novo essa semana?"
2. "Quando isso acontece, quem acaba resolvendo?"
3. Se houver desacordo (por exemplo, sobre pontos e placar na Rotina), deixe cada um contar o último caso concreto, sem decidir nada na hora.

## Depois das conversas

### Registro

| Pergunta | O que disse (citação literal) | Fato ou opinião? | O que isso sugere | Risco afetado (Cagan) | Módulo e requisito afetado |
|---|---|---|---|---|---|
| Quando chegou a última conta de água ou de luz, você olhou o valor? O que pensou? | "Somente eu olho o valor. Estou suspeitando que as contas de água e luz vêm aumentando de leve e gradativamente ao longo dos últimos meses." | Fato | Só uma pessoa acompanha as contas, então o alerta precisa chegar a ela, e não a todos. O aumento leve e gradual é justamente o que o escore z por ciclo não pega e o CUSUM pega. A suspeita é uma percepção: confirmar com os dados. No histórico, a energia de out/2026 foi a maior da série, com tendência de alta, mas o CUSUM ainda está abaixo do limite. | Valor: baixo, porque o problema existe e já há atenção a ele. Usabilidade: médio, porque o alerta precisa ir só para quem acompanha as contas e mostrar a tendência de um jeito fácil de entender. | Consumo: RF-C-05, RF-C-06 (CUSUM), RF-C-07. Agenda: RF-A-04 (canal por pessoa). |
| Já teve uma conta que veio alta e vocês foram descobrir o motivo depois? Como foi? | "Sim, houve um vazamento que não havíamos identificado e só fomos descobrir quando a conta veio; fui no relógio ver que estava rodando mesmo sem a água estar sendo consumida." | Fato | Confirma o problema central do Consumo: o vazamento só apareceu na conta, semanas depois. Com leitura semanal e detecção de falhas, teria sido pego em até uma semana. O alerta deve sugerir a ação feita: fechar tudo e olhar se o relógio continua girando. | Valor: baixo, porque o problema já custou dinheiro de verdade. Viabilidade técnica: baixo, porque a detecção foi validada no histórico real e pegou os 4 incidentes sem alarme falso. | Consumo: RF-C-01, RF-C-06, RF-C-07 (alerta com ação sugerida). |
| Se um vazamento começasse hoje, como vocês perceberiam? | "Hoje eu olho o consumo no relógio periodicamente e anoto em uma planilha que indica se teve uma divergência grande." | Fato | Já existe um processo próprio (leitura no relógio, planilha e painel), a evidência mais forte de que o problema é real. O Ninho precisa ser melhor que isso: trazer todo o histórico, lembrar a leitura e avisar sozinho. Sucesso é parar de usar a planilha e o painel antigos. | Valor: médio. O problema é real, mas já há uma solução que funciona, então o risco é o Ninho não ser melhor que ela. Usabilidade: médio, porque registrar a leitura precisa ser mais rápido que hoje. | Consumo: RF-C-04 (migração), RF-C-06. Métrica do PRD: leituras semanais no prazo. |

### Síntese de 1 página (o entregável)

- Os 3 problemas que mais apareceram, com as citações que os sustentam.
- O que confirmou o PRD e o que contradisse (por exemplo, se ninguém responderia "o que foi isso?" na hora, o risco de valor da captura automática sobe).
- Os compromissos obtidos: quem topou testar o quê, por quanto tempo.
- O que muda no backlog: itens que sobem, que descem ou que saem.

Use a IA para sintetizar as anotações (caso de uso *Synthesize user interviews into findings* da Claude Academy), mantendo as citações como evidência. Revise a síntese você mesmo: é fácil a IA, e você, ouvirem só o que queriam ouvir.

**Não vale como evidência:** "que legal", "eu usaria", "seria ótimo". Vale: um caso concreto recente, um jeito que a pessoa já resolve o problema hoje, um compromisso de tempo.
