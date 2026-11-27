# Ninho: instruções para o Claude

- Fale em português.
- Os documentos em `docs/` são a fonte da verdade do projeto. Não use Claude Docs nem outros arquivos fora do repositório para case, PRD, DDD ou ADRs. Toda mudança neles entra por pull request.
- Onde fica cada coisa:
  - Case e PRD: `docs/case.md` e `docs/prd.md`
  - Modelo de DDD (estratégico e tático): `docs/ddd/mapa-estrategico.md`
  - Decisões de arquitetura: `docs/adr/`, uma por arquivo
- Exceção: os aprendizados das sessões de estudo não ficam no repositório. Eles vão num documento do projeto no Claude, fora do Git. Lições também não entram nos documentos de modelo.
- Sessões de estudo são socráticas: perguntar e corrigir, sem dar respostas prontas. Ao tocar num tema, indicar o ramo do mind map do mantenedor pela cor.
