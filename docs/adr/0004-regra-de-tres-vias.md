# ADR 0004: Adotar a regra de três vias de fronteira

- **Status:** proposto
- **Implementação:** já em código (skill `boundary-rule`)
- **Data:** 2026-08-28

## Contexto

Duas forças opostas atuam sobre o código gerado por agente:

- Minimização ingênua (YAGNI aplicado sem contexto) ataca fronteiras que foram
  compradas de propósito. Porta e adaptador quase nunca passam no teste
  "isso precisa existir agora".
- Adaptador indiscriminado, aplicado porque "o padrão manda", é gordura real,
  e resolver preocupação transversal com adaptador replica a mesma lógica em
  N adaptadores.

O erro de desenho anterior foi colapsar dois eixos diferentes — **fronteira** e
**preocupação transversal** — numa regra binária de adaptador sim ou não.

## Decisão

Três vias, decididas por pergunta:

| Pergunta | Resposta |
|---|---|
| Vou trocar quem está do outro lado? | **Adaptador** |
| Vale para toda chamada, independente da implementação? | **Interceptador** |
| Nenhum dos dois, e o leitor precisa ver isso acontecendo? | **Chamada direta** |

**Adaptador é obrigatório** quando o outro lado é externo e fora do nosso
controle, provável de ter segunda implementação, lento ou não-determinístico em
teste, ou muda em cadência diferente da do domínio.

**Adaptador é ruído** quando é wrapper 1-para-1 sobre algo que controlamos, com
implementação única e nenhum ganho de teste.

**Interceptador** cobre retry, timeout, circuit breaker, cache, log, métrica,
tracing, transação, autorização, idempotência.

## Consequências

**Positivas**

- O desempate entre revisores vira critério aplicável, não gosto.
- Interceptador passa a ser cidadão de primeira classe. No eixo *Easy To Change*
  ele frequentemente ganha do adaptador: é aditivo e removível, não altera a
  forma do código permanentemente.

**Negativas**

- Exige estimar taxa de mudança esperada, que é julgamento, não fato. Dois
  revisores competentes podem divergir de boa-fé.
- Interceptador tem custo próprio: fluxo de controle invisível, difícil de
  depurar. A regra **não** é "prefira interceptador".
- Adaptador é a via que erra para o lado seguro. Em dúvida genuína, adaptador,
  e registra a dúvida.

## Alternativas descartadas

- **"Sempre porta e adaptador".** Paga o prêmio do seguro para sempre em
  fronteira que nunca muda.
- **YAGNI puro.** Destrói reversibilidade, que é o valor que estamos protegendo.
- **Caso a caso, sem regra.** Revisores não convergem, e o humano vira árbitro
  de toda divergência.

## Riscos em aberto

A regra é aplicada por agente, então está sujeita a deriva de interpretação.
Se o mesmo tipo de divergência aparecer três vezes, ela vira exemplo explícito
no texto da skill, não discussão nova.
