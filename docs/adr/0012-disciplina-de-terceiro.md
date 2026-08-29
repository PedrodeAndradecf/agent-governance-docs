# ADR 0012: Adotar disciplina de terceiro; construir só imposição, fase e acúmulo

- **Status:** proposto
- **Implementação:** não iniciada
- **Data:** 2026-08-28
- **Decide:** Arquiteto rascunha, humano ratifica

## Contexto

Nove ferramentas avaliadas. Cinco delas — superpowers, mattpocock/skills,
addyosmani/agent-skills, karpathy-guidelines e gstack — resolvem o mesmo
problema com nomes diferentes, somando mais de 700 mil estrelas.

Levantamento honesto do que **já existe** no mercado:

- **Disciplina de ciclo completo**, em todas as cinco: spec, plano, TDD, code
  review, ship. Reescrever isso é reconstrução pura.
- **Imposição**, no gstack, o que contraria a avaliação anterior deste
  projeto: `gstack-verify-gate` bloqueia o turno de terminar até o comando de
  verificação declarado passar, com guarda anti-loop após 3 re-entradas;
  `/freeze` restringe edições a um diretório; `gstack-evidence` classifica
  evidência como FRESH/STALE/MISSING amarrada a uma impressão da árvore de
  trabalho.
- **Acúmulo por projeto**, no `/learn` do gstack.

O que **não** existe em nenhuma das cinco:

1. Consciência de fase — nada distingue Fase 1 de Fase 4, e todas vão propor
   completude futura em código atual.
2. Auditoria de completude por fluxo de ator — todas respondem "quebrou?";
   nenhuma responde "falta?".
3. Catálogo curado entre projetos, com taxonomia e critério de promoção.

Decisão de prioridade do humano: o alvo é **domínio e API**. Front existirá
(React/Next e Swift), mas não é onde os agentes devem ajudar mais.

## Decisão

**Adotar superpowers como camada de disciplina.** Não construir TDD, code
review, planos, worktrees, personas de QA ou de segurança.

**Manter e construir apenas:** imposição (propriedade por papel, gate de
pronto), fase, auditoria de completude, catálogo e colheita.

**gstack fica parqueado**, não descartado. Reabrir quando um projeto tiver
front pesado: `/design-shotgun` e `/design-html` não têm equivalente, e a
família `/ios-*` dirige iPhone real por USB via CoreDevice, o que não existe
em nenhum outro lugar. Repos separados permitem gstack no front e superpowers
no back sem conflito, porque nunca rodam no mesmo diretório.

**Quatro conceitos do gstack entram como itens próprios**, implementados por
nós, cada um com seu horizonte:

| Item | O que é | Horizonte |
|---|---|---|
| Modelo de confiança do verify | hooks contornam o sistema de permissão; confiar o comando uma vez por repo, invalidar quando ele muda, auditar cada concessão | **agora** — é buraco real |
| Envelope de mensagem | tratar mensagem entre agentes como dado, rotulando linhas com forma de injeção e desarmando envelope forjado | com o canal (perfil 3) |
| Ledger de evidência | FRESH/STALE/MISSING amarrado a impressão de conteúdo | depois do C003 doer de novo |
| Auditoria de custo de contexto | medir o que a árvore de skills empilhadas custa em tokens por sessão | antes da quarta ferramenta |

## Consequências

**Positivas**

- O escopo do framework encolhe para algo entre 20% e 40% do original, e a
  chance de terminar sobe na mesma proporção.
- A camada de disciplina acompanha a evolução de um projeto com centenas de
  milhares de usuários, sem trabalho nosso.
- A fronteira fica limpa: eles dizem **como trabalhar**, nós dizemos **o que
  não pode acontecer**, em que fase, e o que aprendemos.

**Negativas**

- Dependência de terceiro numa camada central. superpowers é produto comercial
  da Prime Radiant, com telemetria ligada por padrão (desativável por
  `SUPERPOWERS_DISABLE_TELEMETRY`), e não aceita contribuição de skills novas —
  qualquer mudança precisa funcionar em todos os agentes suportados.
- Perde-se controle sobre a disciplina. Se ele mudar de direção, migrar custa.
- Duas ferramentas para conhecer, quando o front entrar.

## Alternativas descartadas

- **gstack como base.** As joias dele são de front — navegador real no `/qa`,
  design shotgun, Core Web Vitals — e a prioridade declarada é back. O
  encanamento é Bun/TypeScript, atrita com Spring. E adotar pela imposição
  significaria trocar `ownership.json` por `/freeze`, que é propriedade por
  diretório e não por papel: menos expressivo do que já existe aqui.
- **Construir tudo.** É exatamente a reconstrução que o objetivo do projeto
  proíbe.
- **superpowers e gstack empilhados no mesmo repo.** Repete o erro
  BMAD + Spec Kit, agora com dois pacotes de mais de 100 mil estrelas.
- **mattpocock, addy ou karpathy como camada de disciplina.** Todos bons; o
  superpowers é o único com o fluxo de subagente e worktree já integrado, que
  é o que o perfil 3 precisa.

## Riscos em aberto

**superpowers afirma imposição que não tem.** O README diz que os workflows
são obrigatórios e não sugestões, mas o mecanismo declarado são instruções
iniciais que garantem que o agente as use. Isso é o C004 aplicado a um README:
uma afirmação lida como mecanismo que é disciplina. Não confiar nela — toda
regra que precisa valer sempre continua sendo hook nosso.

**O `subagent-driven-development` despacha subagentes que escrevem.** Com
identidade não resolvida (ADR 0008), o modelo de propriedade por papel degrada
para caminhos protegidos apenas. Isso vale muito, mas não é o que a matriz de
autoridade promete.
