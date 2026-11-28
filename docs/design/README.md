# Design do Ninho

Design system v0.1 e protótipos dos nove fluxos do MVP, aprovados em 28/11/2026.

- **Canvas para ver e clicar:** https://claude.ai/artifact/674JfiUxmA6QvYQzopf9iY (privado do mantenedor)
- **Fontes do canvas:** [`prototipos/`](prototipos/), um arquivo `.dc.html` por prancha e o índice `canvas.json`
- **Prompts que geraram os fluxos:** [`prompts.md`](prompts.md)

Os arquivos de `prototipos/` dependem do runtime do canvas (`support.js`), que não está no repositório. Eles servem como fonte versionada: toda mudança no canvas entra aqui por pull request. Para ver as telas, use o canvas.

## Design system

### Cores

| Papel | Claro | Escuro |
| --- | --- | --- |
| Fundo | `#FAF6F0` | |
| Superfície | `#FFFFFF` | |
| Superfície 2 | `#F1EBE2` | |
| Borda | `#E6DED3` | |
| Texto | `#2B2620` | |
| Texto secundário | `#6B6258` | |
| Destaque | `#2F6B4F` | `#7CC4A0` |

### Pessoas

Cada pessoa tem uma cor própria, usada em avatares, na Agenda e em "quem fez". A cor nunca é o único sinal: vem sempre com a inicial ou o nome.

| Pessoa | Claro | Escuro |
| --- | --- | --- |
| Responsável A | `#24508F` (azul) | `#8DB3F2` |
| Responsável B | `#7A4BB0` (roxo) | `#C9A6F0` |
| Dependente | `#B4561A` (laranja) | `#F2A469` |

### Estados

Cada estado vem com cor, ícone e texto.

| Estado | Fundo | Texto |
| --- | --- | --- |
| Pendente | `#EFE8DE` | `#5C5248` |
| Conciliado, tem | `#E1F0E7` | `#22573D` |
| Atenção | `#FCEFD6` | `#7A4E00` |
| Acabou | `#F8E0DC` | `#9A2B1E` |
| Alerta (sólido) | `#9A2B1E` | `#FFFFFF` |

### Tipografia

Figtree nos títulos e Inter no corpo. Os valores usam números tabulares.

### Regras

- **Escrita:** todo texto da interface fica em sentence case, com só a primeira letra maiúscula.
- **Toque:** os alvos têm pelo menos 48 px. A ação principal fica embaixo, ao alcance de uma mão.
- **Telas:** a largura base é 360 px.
- **Navegação dos responsáveis:** a barra inferior tem seis destinos: Hoje, Gastos, Rotina, Agenda, Consumo e Despensa.
- **Navegação do dependente:** a barra tem quatro destinos: Hoje, Rotina, Agenda e Despensa. Ele não vê a Memória de gastos nem o Consumo.
- **Dados de exemplo:** são sempre fictícios, como Responsável A, Responsável B, Dependente, Banco 1 e Banco 2.

## Fluxos

| Fluxo | Protótipo clicável | Estados |
| --- | --- | --- |
| 1. Anotar um gasto | `GastoFluxo` | `GastoEstados` |
| 2. Conciliar o mês | `Conciliar` | `ConciliarEstados` |
| 3. O que foi isso? | `Pergunta` | `PerguntaEstados` |
| 4. Tarefas e placar da Rotina | `Rotina` | `RotinaEstados` |
| 5. Listas de compras | `Listas` | `ListasEstados` |
| 6. Agenda da família | `Agenda` (e o componente `Icone`) | `AgendaEstados` |
| 7. Água e energia | `Consumo` | `ConsumoEstados` |
| 8. Despensa | `Despensa` | `DespensaEstados` |
| 9. Família e preferências | `Familia` | `FamiliaEstados` |

As pranchas de base são `Main` (cores, pessoas e tipografia), `Componentes` e `Navegacao` (responsável, dependente e tema escuro).

### Decisões tomadas nos protótipos

- Só os responsáveis criam tarefas na Rotina. O dependente abre a Rotina em "Minhas".
- Na Despensa, o dependente marca e adiciona itens. No perfil alimentar, ele vê a família toda e edita só o próprio perfil.
- Só o administrador convida, troca papéis e remove membros.
- Cada pessoa tem uma cor única. As cores já usadas aparecem desativadas.
- O dependente configura só os avisos de lembrete e de item que acabou.
