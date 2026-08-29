# ADR 0003: Três camadas com donos distintos; descartar BMAD

- **Status:** proposto
- **Implementação:** parcial
- **Data:** 2026-08-28

## Contexto

BMAD, Spec Kit e os recursos nativos do Claude Code se sobrepõem. Instalar mais
de um framework de orquestração no mesmo repositório produz duas fontes de
verdade para a especificação e nenhum árbitro entre elas.

BMAD tenta ocupar as três camadas de uma vez. Ele foi desenhado quando o Claude
Code ainda não tinha subagentes, hooks e worktrees nativos.

## Decisão

Uma camada, um dono:

| Camada | Responsabilidade | Dono |
|---|---|---|
| Artefato | constitution, spec, plan, tasks, ADR | Spec Kit + templates deste plugin |
| Papéis | quem faz o quê, com qual autoridade | `agents/` deste plugin |
| Execução | o que é permitido acontecer | `hooks/` e worktrees nativos |

BMAD não é instalado. Suas checklists de papel podem ser lidas como material
de referência.

## Consequências

**Positivas**

- Uma fonte de verdade por camada; conflito de ferramenta deixa de ser possível
  por construção.
- As camadas 2 e 3 acompanham a evolução do Claude Code sem trabalho nosso.

**Negativas**

- Dependência do Spec Kit na camada 1, cuja superfície de CLI já mudou de forma
  incompatível (a família de flags `--ai` foi substituída por `--integration`).
- Perde-se a curadoria de papéis do BMAD, que é boa.
- Fica um vão: o plugin hoje não sabe nada sobre `specs/`. A camada 1 está
  atribuída mas não conectada.

## Alternativas descartadas

- **BMAD como framework único.** Conflita com o que o Claude Code já faz nativo
  nas camadas 2 e 3, e traz cerimônia demais para operação solo.
- **BMAD e Spec Kit juntos.** Duas fontes de verdade, sem árbitro.
- **Tudo caseiro.** Reescrever a camada 1 sem ganho sobre uma ferramenta MIT
  já adotada.

## Riscos em aberto

Se o Spec Kit quebrar compatibilidade de novo, a camada 1 fica órfã. Como o
acoplamento é só de diretório (`specs/`, `.specify/`), a substituição é viável,
mas ninguém testou isso. Manter o acoplamento nesse nível é uma decisão ativa,
não acidente.
