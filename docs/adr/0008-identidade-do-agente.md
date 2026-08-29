# ADR 0008: Identificar o agente pelo worktree

- **Status:** **proposto — bloqueado por experimento**
- **Implementação:** escrita, não validada contra payload real
- **Data:** 2026-08-28

## Contexto

O modelo de propriedade exige responder "qual agente está fazendo esta
chamada?". Sem isso, `ownership.json` degrada para uma lista de caminhos
protegidos, sem autoridade por papel.

O que se sabe hoje:

- Hooks vindos de settings, de política gerenciada e de plugins **rodam dentro
  de subagentes**. Quando um subagente chama uma ferramenta, `PreToolUse` e
  `PostToolUse` disparam os mesmos hooks configurados.
- O input carrega os campos comuns `agent_id` e `agent_type`, populados quando
  o hook dispara dentro de um subagente.
- Subagentes **não herdam** automaticamente as permissões do agente principal,
  o que reforça o papel do hook como única barreira.
- A semântica de `agent_type` é ambígua na documentação disponível: aparece
  tanto como `"main"` quanto como `"general-purpose"`. Se for um bucket
  genérico e não o nome do agente customizado, ele não serve como chave.
- Existe issue aberta no repositório do Claude Code pedindo identidade de agente
  no `PreToolUse`, argumentando que nenhuma das opções existentes oferece
  imposição técnica de permissão por agente.
- Alguns exemplos de payload trazem um campo `worktree`.

**Fato crítico sobre o estado atual:** os 10 cenários validados chamaram os
scripts diretamente, com JSON sintético. O ramo que depende de identidade nunca
viu um payload real. Se a chave escolhida não existir na prática, todos os
testes continuam verdes e o guarda cai sempre no default.

## Decisão (proposta)

Chave primária de identidade: **o worktree**. `agent_type` entra como reforço
opcional, nunca como única fonte.

Justificativa: o worktree já é a fronteira de isolamento do paralelismo, então
identidade e isolamento passam a ser a mesma coisa, e a chave não depende de o
modelo declarar quem ele é.

## Experimento que desbloqueia

Hook descartável em `PreToolUse` que só grava o payload:

```sh
cat > /tmp/payload-$(date +%s%N).json
```

Disparar `Write` a partir de: sessão principal, cada subagente customizado, e
uma sessão em worktree separado. Comparar os campos. Vinte minutos.

## Consequências, se a proposta se confirmar

**Positivas**

- Identidade independente de declaração do modelo.
- Alinha a autoridade com a fronteira que já existe para evitar colisão de merge.

**Negativas**

- Exige que todo agente autônomo rode em worktree dedicado, o que passa a ser
  requisito e não recomendação.
- A sessão interativa principal não tem worktree dedicado. Precisa de regra
  explícita para o caso "sem worktree", e o default seguro é tratá-la como o
  papel de menor privilégio, não de maior.

## Alternativa se nada servir

Degradar declaradamente para o modelo de **caminhos protegidos apenas**, sem
propriedade por agente, e **documentar a limitação como limitação**. Manter no
repositório um modelo de autoridade que não é imposto seria repetir o defeito
que este projeto existe para corrigir.

## Riscos em aberto

Tudo. Este ADR é a única decisão do conjunto cuja premissa técnica ainda não foi
observada diretamente. Nenhuma outra decisão que dependa de identidade deve ser
tomada antes do experimento.
