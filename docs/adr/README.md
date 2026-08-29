# Registro de decisões de arquitetura

Decisões que governam o repositório `agent-governance` e o plugin que ele
distribui. Decisão de projeto-alvo não mora aqui, mora no ADR do projeto.

## Como isto funciona

- O Arquiteto rascunha, o humano **ratifica**. Rascunho não ratificado tem
  status `proposto` e não vincula ninguém.
- Dois eixos independentes: **status** (a decisão vale?) e **implementação**
  (o código reflete isso?). A distância entre os dois é dívida visível.
- Alterar um ADR ratificado é o único caso que escala obrigatoriamente para
  o humano. Todo o resto o Arquiteto decide sozinho.
- Este diretório é protegido pelo guarda de propriedade. Implementador não
  escreve aqui.

## Índice

| ADR | Título | Status | Implementação |
|---|---|---|---|
| [0001](0001-politica-por-mecanismo.md) | Impor política por mecanismo, não por instrução | proposto | já em código |
| [0002](0002-distribuicao-por-plugin.md) | Distribuir via plugin e marketplace | proposto | parcial |
| [0003](0003-tres-camadas.md) | Três camadas com donos distintos; descartar BMAD | proposto | parcial |
| [0004](0004-regra-de-tres-vias.md) | Adotar a regra de três vias de fronteira | proposto | já em código |
| [0005](0005-ponytail-aconselha.md) | Ponytail aconselha, nunca bloqueia | proposto | não iniciada |
| [0006](0006-generico-x-especifico.md) | Separar genérico de específico | proposto | já em código |
| [0007](0007-modo-de-degradacao.md) | Falhar aberto sem config, fechado com config inválida | proposto | já em código |
| [0008](0008-identidade-do-agente.md) | Identificar o agente pelo worktree | proposto | não validada |
| [0012](0012-disciplina-de-terceiro.md) | Adotar disciplina de terceiro; construir só imposição, fase e acúmulo | proposto | não iniciada |
| [0013](0013-framework-como-ponto-de-partida.md) | Framework como ponto de partida, com colheita sob demanda | proposto | não iniciada |

## Dívida visível hoje

- **0002 e 0003 estão parciais.** A camada de distribuição não existe no repo,
  apesar de o CHANGELOG afirmar o contrário.
- **0008 deixou de ser bloqueio.** Com um terminal por agente, o worktree é a
  chave de identidade e o hook de log de payload confirma pelo uso. O ramo de
  propriedade por agente segue não validado contra payload real até a primeira
  rodada.
- **0009 a 0011 estão pendentes de escrita**: fluxo por ator como artefato
  normativo, triagem temporal de achados, e marca temporal obrigatória.
- **0012 reduz o escopo** dos anteriores: TDD, code review, planos e worktrees
  saem de cena. O que fica é imposição, fase, completude e acúmulo.
