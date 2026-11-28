# Ninho: prompts para o Claude Design

Prompts para gerar protótipos das telas do Ninho no Claude Design, tirados do PRD (v0.2). Ordem sugerida: cole o prompt 0 uma vez no início do projeto (ou use-o para criar o design system), depois um prompt por vez, começando pelo fluxo 1. Cada prompt cita os requisitos do PRD entre parênteses só para referência; pode apagar antes de colar.

Dados de exemplo sempre fictícios: pessoas como "Responsável A", "Responsável B" e "Dependente"; bancos como "Banco 1" e "Banco 2".

---

## 0. Contexto e design system (cole primeiro)

```
Vou prototipar o Ninho, um app da família (PWA mobile-first, usado no Chrome do Android, telas a partir de 360 px; também abre no computador). A família tem três pessoas: dois responsáveis e um dependente (menor de idade). Tudo em português do Brasil, fuso de São Paulo, valores em reais (R$ 1.234,56), datas no formato 27/11.

O app tem cinco módulos: Memória de gastos, Rotina, Agenda e lembretes, Consumo (água e energia) e Despensa. Navegação por uma barra inferior com os cinco módulos e uma tela inicial "Hoje". O dependente não vê a Memória de gastos nem o Consumo: preveja como a navegação fica para esse papel (quatro destinos: Hoje, Rotina, Agenda e Despensa).

Regras de experiência:
- A ação principal de cada módulo sai em até 3 toques a partir da tela inicial.
- Uso com uma mão: botões grandes, ação principal embaixo, alvos de toque de pelo menos 48 px.
- Todo texto da interface em sentence case: só a primeira letra maiúscula (botões, chips, itens de lista, títulos). Ex.: "Mercado da semana", nunca "Mercado Da Semana" nem "mercado da semana".
- Funciona sem internet nas listas e na anotação de gasto: preveja um indicador discreto de "offline, vai sincronizar" e de "sincronizado".
- Cada pessoa tem uma cor própria, usada em avatares, na agenda e em "quem fez".
- Linguagem simples, calorosa e direta, sem jargão financeiro. Nada de gamificação infantil exagerada.

Design system:
- Visual acolhedor e calmo, de casa, não de banco. Fundo claro quente, um tom de destaque principal (sugira um verde ou terracota) e três cores de pessoa bem distintas entre si e acessíveis (contraste AA).
- Tipografia sans-serif legível, números tabulares para valores.
- Componentes: barra inferior, cabeçalho com título e avatar, cartão, item de lista com checkbox, chip de filtro, botão flutuante de adicionar, teclado numérico de valor, seletor de pessoa por avatar, badge de estado (pendente, conciliado, tem, acabou, atenção, alerta), banner de aviso, folha inferior (bottom sheet), estado vazio, estado offline.
- Tema claro e escuro.

Use dados fictícios: "Responsável A", "Responsável B", "Dependente", "Banco 1", "Banco 2". Primeiro me mostre o design system (cores, tipografia, componentes) numa página só, antes de desenhar telas.
```

---

## 1. Primeiro fluxo: lançar um gasto (Memória de gastos)

```
Usando o design system do Ninho, crie um protótipo clicável do fluxo "anotar um gasto" no celular.

Objetivo: no fim do mês alguém classifica os lançamentos no app de finanças e não lembra o que foi cada compra. A anotação rápida guarda essa memória na hora da compra.

Telas:
1. Tela "Hoje" com um botão de destaque "Anotar gasto" (é o primeiro toque).
2. Anotação rápida (segundo toque já é digitar): teclado numérico grande para o valor em reais; campo "O que foi?" com sugestões recentes em chips; quem gastou (um dos dois responsáveis, por avatar, o usuário atual já marcado); meio de pagamento (cartão de crédito, conta ou dinheiro), com o cartão do Banco 1 ou do Banco 2 quando for crédito; data e hora preenchidas automaticamente, editáveis; foto opcional do cupom. Botão "Salvar" fixo embaixo (terceiro toque).
3. Confirmação breve (toast) com "Desfazer", voltando para "Hoje" com a anotação no topo de "Anotações de hoje".
4. Variação sem internet: a anotação salva com o selo "vai sincronizar".

Mostre também o estado de erro de valor vazio e como fica a tela num celular de 360 px.
```

## 2. Lista do mês e conciliação (Memória de gastos)

```
Continuando a Memória de gastos, crie a tela "Conciliar" do mês.

- Lista das anotações do mês, agrupadas por dia, cada uma com valor, o que foi, avatar de quem gastou, meio de pagamento e estado (pendente ou conciliada).
- Filtros em chips: pessoa, meio de pagamento e período; busca por valor (digitar "45,90" acha a anotação).
- Resumo no topo: "32 de 40 conciliadas".
- Marcar como conciliada com um toque (ou deslizar). Uma anotação conciliada não volta a pendente sem confirmação explícita: mostre o diálogo "Voltar para pendente?".
- Detalhe da anotação numa folha inferior, com a foto do cupom se tiver.
- Tela de fechamento do mês: o que ficou sem conciliar, com destaque e um botão para revisar cada um.

Essa área só aparece para os responsáveis.
```

## 3. "O que foi isso?" (Memória de gastos, fase 2)

```
Crie o fluxo de resposta à pergunta "o que foi isso?". Uma compra é capturada da notificação do app do banco e vira uma anotação pendente com valor, estabelecimento e cartão já preenchidos.

- Notificação push do Android: "R$ 87,40 em ESTABELECIMENTO EXEMPLO, cartão do Banco 1. O que foi isso?", com resposta rápida direto na notificação (campo de texto ou sugestões como "mercado", "farmácia").
- Tela no app com a pergunta, os dados capturados (somente leitura) e um campo grande "O que foi?", com sugestão "da última vez aqui foi: farmácia".
- Fila de perguntas pendentes da pessoa, para responder várias seguidas.
- Tela do administrador: "Notificações que não deu para ler", com o texto original e a opção de criar a anotação à mão.
```

## 4. Tarefas e placar (Rotina)

```
Crie o módulo Rotina, parte de tarefas.

- Lista de tarefas da semana, filtrável por "minhas" e "todas", cada uma com responsável (avatar e cor), prazo, pontos e se é recorrente.
- Criar tarefa: nome, responsável (qualquer pessoa da família, inclusive o dependente), única ou recorrente (diária, semanal, dias específicos), prazo e pontos.
- Concluir tarefa com um toque, sem aprovação; foto opcional. Pequena celebração discreta ao concluir.
- Placar da semana por pessoa (pontos, sem dinheiro), com o histórico das semanas anteriores.
- Mural da família: o que cada um fez na semana, em formato de linha do tempo.

Mostre a mesma tela vista pelo dependente.
```

## 5. Listas de compras (Rotina)

```
Crie as listas de compras da Rotina, compartilhadas em tempo real.

- Várias listas (Mercado, Farmácia, Feira) em abas ou cartões, com a contagem de itens pendentes.
- Lista aberta: adicionar item em um toque (campo fixo embaixo), cada item com quem pediu (avatar), quantidade opcional e checkbox de comprado; comprados descem para uma seção recolhida.
- Indicador de presença: "Responsável B está nesta lista agora" e itens que chegam ao vivo com uma animação sutil.
- Itens que vieram da Despensa com o selo "acabou na despensa".
- Modo sem internet: banner "offline, suas mudanças vão sincronizar" e, quando duas pessoas mexem no mesmo item, um aviso claro do conflito com as duas versões para escolher.
```

## 6. Agenda da família (Agenda e lembretes)

```
Crie o módulo Agenda e lembretes.

- Calendário compartilhado com visões de dia, semana e mês; cada evento na cor da pessoa (ou de várias pessoas).
- Categorias com ícone: escola, saúde, aniversários, casa, lazer; filtro por categoria e por pessoa.
- Criar evento: título, data e hora, quem participa, categoria, recorrência e lembretes (antecedência configurável, por exemplo 1 dia e 1 hora antes).
- Opção "lembrete com confirmação": repete até alguém marcar "feito". Mostre a notificação com o botão "Feito" e o estado "confirmado por Responsável A às 14:05".
- Indicador da sincronização com a agenda "Família" do Google Calendar e o aviso quando houve conflito ("este evento foi alterado no Google; ficou a última edição").
```

## 7. Água e energia (Consumo)

```
Crie o módulo Consumo, para água e energia da casa.

- Painel com dois cartões: Água (leitura semanal do hidrômetro em m³) e Energia (pela conta, em kWh). Cada um mostra o consumo do ciclo atual, a média diária e a comparação com o ciclo anterior e o mesmo mês do ano passado.
- Registrar leitura de água: teclado numérico para os 3 dígitos do hidrômetro (o contador volta a 000 depois de 999; mostre como o app explica isso), data automática.
- Registrar conta: tipo (água ou energia), consumo, dias do ciclo e valor em reais.
- Gráfico do histórico por ciclo, com faixa do normal e os ciclos marcados em "atenção" e "alerta".
- Alerta de possível vazamento: notificação urgente e tela de detalhe com a explicação simples ("o consumo deste ciclo está bem acima do normal dos últimos 12 ciclos") e a pergunta "Era mesmo um problema?" com Sim e Não.

O Consumo é só dos responsáveis: o dependente não vê esse módulo nem recebe os alertas dele.
```

## 8. Despensa

```
Crie o módulo Despensa.

- Itens da casa por categoria (limpeza, mercearia, higiene, geladeira), cada um com estado "tem" ou "acabou" alternado com um toque.
- Ao marcar "acabou", um aviso breve: "Foi para a lista Mercado". Quando o item é comprado na lista, volta para "tem".
- Adicionar item: nome, categoria e em qual lista de compras ele entra.
- Perfil alimentar da família, preenchido aos poucos: por pessoa, preferências, restrições e o que não gosta, em chips; com um convite suave para completar ("Faltam 2 pessoas").
```

## 9. Configurações da família (Identidade e Notificações)

```
Crie as telas de família e preferências.

- Criar a família (quem cria vira administrador) e convidar membro por link com o papel já definido (responsável ou dependente); o link vale 7 dias e serve uma vez.
- Lista de membros com papel e cor; trocar papel e remover membro (só o administrador; avise que a família não pode ficar sem administrador).
- Perfil: nome, cor e preferências de notificação em uma grade de tipo de aviso (lembrete, "o que foi isso?", alerta de consumo, item que acabou) por canal (push e e-mail), com interruptores.
- Horário de silêncio, com a explicação de que avisos urgentes, como vazamento, passam mesmo assim.
```

---

## Dicas de uso

- Um fluxo por conversa dentro do mesmo projeto, para o design system valer em todos.
- Depois de gerar, refine comentando direto na tela ("aumente o teclado", "troque a cor de destaque").
- Para virar código depois, use a passagem para o Claude Code.
