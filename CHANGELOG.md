# Changelog

Regra: **nenhuma linha aqui sem job de CI que a sustente.** Este arquivo é
derivado do release, não escrito à mão (C003).

## [Não publicado]

### Corrigido
- Entradas que afirmavam marketplace e CI como concluídos quando nenhum dos dois
  estava commitado. O registro anterior contradizia `docs/VERIFICACAO.md`, que
  já admitia que os itens 1 e 2 da definição de pronto nunca foram executados.
- Slug de instalação no README: `PedrodeAndradecf/agent-governance`.
- Removido `README.md.bak`, sobra de edição idêntica ao `README.md`.

### Adicionado
- `.claude-plugin/marketplace.json` — catálogo do marketplace. `[VERIFICAR]`
  schema contra a documentação antes do primeiro `/plugin marketplace add`.
- `.github/workflows/validate.yml` — JSON válido, componentes na raiz do plugin,
  sintaxe POSIX e bit de execução dos hooks, links de ADR.
- `docs/` — síntese, guia operacional, catálogo de cenários e 10 ADRs.
- `plugins/governance/templates/` — `PROJETO.md`, `FRICCAO.md`, `adr.md`.

### Não validado
- `claude plugin validate` não foi executado.
- Instalação em projeto vazio não foi executada.
- Os 10 cenários de guarda passaram com **payload sintético**, chamando os
  scripts diretamente. O ramo que depende de identidade de agente nunca viu
  payload real.
