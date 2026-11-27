# Ninho: aprendizados do projeto

O que o Guto aprendeu estudando e construindo o Ninho, por tema. Cada sessão nova acrescenta suas lições aqui, sempre por pull request.

## DDD estratégico

Sessão de 24/11/2026. O erro que mais se repetiu foi chegar à conclusão certa por um critério que não decide nada. O modelo resultante está no [mapa de DDD](ddd/mapa-estrategico.md).

1. **Se tudo é core, nada é.** Core exige as duas coisas: regra de negócio complexa e nada pronto para usar. "Resolve um problema da família" vale para todos os módulos.
2. **A complexidade que conta é a de negócio.** Sync offline e idempotência são difíceis, mas tecnicamente. A regra "acabou vai para a lista" é trivial.
3. **Classifique pela regra mais difícil, não pela tela mais visível.** O Consumo parecia CRUD porque a tela é de cadastro, mas a detecção é o core do app.
4. **Classifique pelo que existe hoje.** Um subdomínio pode mudar de tipo depois, e isso se registra quando acontecer.
5. **Quantos usam algo não define o tipo.** Notificações é generic porque tem solução pronta, não porque quatro contextos usam.
6. **Fronteira é de linguagem, não de tecnologia.** Trocar push por e-mail não muda fronteira nenhuma.
7. **Upstream é quem não precisa conhecer o outro.** A Rotina não sabe que a Despensa existe.
8. **Conformista e ACL são alternativas.** "Não controlo o outro lado" vale para os dois. O que decide é se vale proteger o meu modelo.
9. **Com uma equipe só, sobram os padrões de proteção de modelo.** Parceria, Cliente-fornecedor e Separate Ways são de organização de equipes. ACL, OHS, Published Language e Conformista continuam valendo.
10. **Não compartilhe o que cada contexto modela diferente.** Uma classe Pessoa única acumularia tudo, e mexer em pontos quebraria a Conciliação.

## DDD tático

Sessões de 25 e 26/11/2026, com peso maior em agregados. O modelo resultante está na seção "DDD tático" do [mapa de DDD](ddd/mapa-estrategico.md).

11. **Active Record e Domain Model se separam por onde mora a regra.** No primeiro ela fica num serviço fora do objeto; no segundo, dentro dele, que recusa estados inválidos.
12. **Event Sourcing é para quando o histórico é a fonte da verdade.** Não tem a ver com quantos estados existem. O Ninho não é o livro-razão do dinheiro, então não precisa.
13. **Imutabilidade é consequência de ser objeto de valor, não o teste.** O teste é se importa *qual* é. "Vem pronto de fora" não decide: os dois cafés iguais são duas entidades.
14. **Fuja da obsessão por primitivos.** Um `number` aceita reais mais pontos. Um `Dinheiro` valida no construtor e não deixa misturar.
15. **Entre contextos e entre agregados, referência por id.** A pessoa é entidade na Identidade; na Conciliação é só um MembroId.
16. **Agregado nunca atravessa bounded context.** A Despensa não pode ser agregado de algo da Rotina; os dois conversam por mensagem.
17. **Invariante é regra que vale sempre.** Se der para corrigir depois, não é invariante: é consistência eventual.
18. **Uma transação altera um agregado só.** É isso que define o tamanho dele, não "nascem em momentos diferentes". Agregado grande gera conflito de concorrência e cresce sem limite.
19. **Regra que envolve dois agregados mora no que precisa poder dizer não.** O Lançamento recusa o segundo casamento; a Anotação fica sabendo pelo evento. O preço é desfazer quando o passo seguinte falha (Saga).
20. **Pertencer não é proteger.** A raiz é quem atravessa o ciclo de vida da regra e guarda o mínimo para protegê-la: o Hidrômetro com a última leitura, não o Ciclo nem a Leitura.
21. **Comando é intenção, evento é fato.** Comando no imperativo, para um destinatário que pode recusar. Evento no passado, para quem quiser ouvir.
22. **ECST é levar o estado junto.** Quando o consumidor não conhece seus ids, devolva o id de correlação que ele mandou.
