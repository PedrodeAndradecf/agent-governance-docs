# ADR 0002: Distribuir via plugin e marketplace

- **Status:** proposto
- **Implementação:** parcial
- **Data:** 2026-08-28

## Contexto

A política precisa valer em N projetos, com versionamento e sem cópia manual
de configuração. `.claude/settings.json` é escopado no projeto e não tem
mecanismo próprio de versão ou atualização.

Um plugin do Claude Code é um diretório com `.claude-plugin/plugin.json` como
manifesto e os componentes na raiz do plugin. Um marketplace é um catálogo em
`.claude-plugin/marketplace.json` na raiz do repositório, e o GitHub é a forma
recomendada de hospedá-lo. O formato de hook é idêntico em `settings.json` e
em `hooks/hooks.json`, então migrar entre os dois é mover o bloco.

## Decisão

O repositório hospeda **um marketplace e um único plugin**. Instalação por
`/plugin marketplace add PedrodeAndradecf/agent-governance`.

Um plugin só. Plugin que faz demais é o erro mais comum; se crescer, divide
depois com evidência.

## Consequências

**Positivas**

- Versionamento semântico e pinagem por projeto.
- Hooks, agentes e comandos registram automaticamente na instalação.
- Instalação em um comando, sem tutorial de cópia de arquivos.

**Negativas**

- Plugins são copiados para um cache na instalação e **não conseguem**
  referenciar arquivos fora do próprio diretório por caminhos como `../shared`.
  Isso força o ADR 0006.
- Uma mudança de comportamento em um guarda afeta N projetos ao mesmo tempo.
  Mitigação obrigatória: projetos pinam versão; mudança de guarda é sempre
  minor ou major, nunca patch.
- Componente colocado dentro de `.claude-plugin/` em vez da raiz do plugin não
  carrega, e o sintoma é silencioso. `claude plugin validate` e `claude --debug`
  são parte do procedimento de release, não opcionais.

## Alternativas descartadas

- **`settings.json` copiado entre projetos.** Sem versionamento; drift garantido.
- **Git submodule ou subtree.** Fricção alta, e worktrees multiplicam o problema.
- **Instalador `npx`, no modelo do BMAD.** Mais código para manter, sem ganho
  sobre o mecanismo nativo.

## Riscos em aberto

A camada de distribuição não existe no repositório hoje. O primeiro comando do
README não funciona. Até isso ser fechado, esta decisão está registrada mas não
entregue — e o CHANGELOG afirma o contrário, o que é um defeito por si só
(ver ADR proposto sobre CHANGELOG derivado).
