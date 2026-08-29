# Mapa de aplicação no repo

`[HOJE]` Este arquivo **não** vai para o repo. É a instrução de como aplicar
esta pasta em `PedrodeAndradecf/agent-governance`.

## Resumo

**Não apague nada** do que existe. `plugins/governance/hooks/`, `agents/`,
`skills/` e `templates/` passaram em 10 de 10 cenários testados e são a única
parte com código funcionando. Esta pasta **acrescenta** a camada de documentação
e a camada de distribuição que faltava.

Única remoção: `README.md.bak`.

## Onde cada arquivo vai

| Arquivo desta pasta | Destino | Ação |
|---|---|---|
| `.claude-plugin/marketplace.json` | raiz | **novo** — era o gap do relatório |
| `.github/workflows/validate.yml` | raiz | **novo** — era o gap do relatório |
| `CHANGELOG.md` | raiz | **substitui** — o atual afirma o que não existe |
| `docs/SINTESE.md` | `docs/` | novo — porta de entrada |
| `docs/GUIA.md` | `docs/` | novo — instalação e fluxo operacional |
| `docs/CATALOGO.md` | `docs/` | novo — 6 cenários em arquivo único |
| `docs/adr/*` | `docs/adr/` | novo — 10 ADRs + template + índice |
| `plugins/governance/templates/PROJETO.md` | idem | novo |
| `plugins/governance/templates/FRICCAO.md` | idem | novo |
| `plugins/governance/templates/adr.md` | idem | novo (ou substitui, se já houver) |

## O que já existe e permanece intocado

```
plugins/governance/.claude-plugin/plugin.json
plugins/governance/hooks/          ownership-guard.sh, verify-gate.sh, …
plugins/governance/agents/
plugins/governance/skills/
plugins/governance/templates/      ownership.json, constitution.md
plugins/governance/commands/
docs/VERIFICACAO.md
README.md
```

## Correções a fazer à mão

1. **`README.md`** — o slug de instalação está `pedroacompany/agent-governance`.
   Correto: `PedrodeAndradecf/agent-governance`. É o primeiro comando que
   qualquer pessoa roda.
2. **`rm README.md.bak`** — idêntico ao `README.md`, diff vazio.
3. **`docs/VERIFICACAO.md`** — manter. Ele estava certo enquanto o CHANGELOG
   mentia; é o artefato honesto do par.

## Depois de aplicar

```sh
claude plugin validate ./plugins/governance
```

Isso fecha o item 1 da definição de pronto, que nunca foi executado.

## O que ainda não existe

- **ADRs 0009, 0010, 0011** — fluxo por ator como artefato normativo, triagem
  temporal de achados, marca temporal obrigatória. Deixados para depois da
  ratificação do 0012, porque são o que sobra de original no framework e
  merecem ser escritos com a cabeça pós-decisão.
- **Runbooks** — superpowers, ponytail, archify. Categoria de documento que não
  existe: ADR registra decisão, catálogo registra falha, nenhum registra
  operação.
- **Comando de colheita** (ADR 0013) e **comando de bootstrap** completo.
- **Packs por stack** — `springboot` primeiro.

## Ordem sugerida

1. Aplicar esta pasta, corrigir slug, apagar o `.bak`, commitar.
2. Rodar `claude plugin validate`.
3. Ratificar ou devolver os 10 ADRs. Ratificação é sua; rascunho não vincula.
4. Etapa 0 no brechó: cópia manual dos hooks, `PROJETO.md` preenchido, hook de
   log de payload ligado junto.
