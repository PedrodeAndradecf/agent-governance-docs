# ADR 0006: Separar genérico de específico

- **Status:** proposto
- **Implementação:** já em código
- **Data:** 2026-08-28

## Contexto

Consequência direta do ADR 0002: plugins são copiados para um cache na
instalação e não conseguem referenciar arquivos fora do próprio diretório.
Qualquer regra de projeto embutida no plugin torna o plugin utilizável por um
projeto só, o que anula a razão de ele existir.

## Decisão

**No plugin (genérico, versionado):** lógica dos guardas, definições de agentes
e seus limites de ferramenta, a regra de três vias, templates, comando de
bootstrap.

**No projeto (específico):** `.claude/ownership.json`, `scripts/verify.sh`,
`.specify/memory/constitution.md`, `docs/adr/`.

**Contrato:** os scripts do plugin **leem** configuração do projeto e nunca
embutem regra de projeto. A configuração é declarativa e legível por máquina.

## Consequências

**Positivas**

- Um plugin serve N projetos; cada projeto mantém controle da própria política.
- O bootstrap fica sendo a única costura entre os dois mundos.

**Negativas**

- O schema de `ownership.json` vira superfície pública versionada. Mudá-lo quebra
  projetos instalados, então ele segue as mesmas regras de compatibilidade de
  uma API.
- Mais arquivos gerados no projeto alvo, e mais coisas que podem estar ausentes
  ou desatualizadas (ver ADR 0007).

## Alternativas descartadas

- **Regra embutida no plugin.** Serve um projeto só.
- **Symlink para diretório compartilhado.** Frágil, e worktrees multiplicam o
  problema de resolução de caminho.

## Riscos em aberto

Não existe hoje versionamento do próprio `ownership.json`. Um campo `schema`
no arquivo, checado pelo guarda, evitaria falha silenciosa quando o plugin for
atualizado à frente do projeto. Ainda não implementado.
