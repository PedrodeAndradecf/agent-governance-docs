# ADR 0005: Ponytail aconselha, nunca bloqueia

- **Status:** proposto
- **Implementação:** não iniciada
- **Data:** 2026-08-28

## Contexto

Ponytail é uma skill de terceiro que injeta uma escada de decisão minimalista
no contexto do agente antes de ele escrever código. O valor é real em corte de
gordura. O conflito também é real: a escada tende a atacar adaptadores
legítimos (ver ADR 0004).

Além disso, ele opera em modo always-on por meio de hooks de ciclo de vida em
Node.js executados no início da sessão. A própria documentação da ferramenta
reconhece que ela é um ponto de injeção de prompt e recomenda revisar os hooks
antes de confiar. Quando os hooks não são ativados, o modo always-on fica
silencioso e as skills continuam funcionando por invocação manual.

## Decisão

Modo always-on **desligado**. Ponytail entra como parecer, num evento de hook
que não bloqueia.

Em divergência entre Ponytail e Arquiteto, o **Arquiteto vence**, e o corte
recusado vira uma linha de dívida registrada. Só escala para o humano o caso em
que o Arquiteto quer alterar um ADR ratificado.

A prioridade é codificada em **qual evento cada um ocupa**, não em texto
pedindo bom senso.

## Consequências

**Positivas**

- Prioridade não é renegociável em tempo de execução.
- A cadeia de interceptadores permanece nossa, auditável e versionada.
- Superfície de terceiro reduzida a código que roda quando invocamos.

**Negativas**

- Perde-se parte do valor da ferramenta, que foi desenhada para always-on.
- Depende de a dívida ser efetivamente registrada em algum artefato durável.
  Enquanto o parecer dos revisores for texto efêmero de chat, essa metade da
  decisão não existe de fato.

## Alternativas descartadas

- **Instalar com always-on.** Hook de terceiro roda como processo do SO com as
  permissões do usuário, no mesmo contexto dos agentes autônomos.
- **Copiar a escada para dentro do `constitution.md` e não usar a ferramenta.**
  Descartada por decisão do humano: o objetivo é uma estratégia de composição de
  ferramentas, não uma cópia parcial.
- **Não usar Ponytail.** O parecer tem valor mensurável em código supérfluo.

## Riscos em aberto

Esta decisão não é implementável até existir destino para o parecer e para a
dívida. Ela está bloqueada pela mesma lacuna que o ADR 0008: falta modelar o
artefato de saída dos revisores.
