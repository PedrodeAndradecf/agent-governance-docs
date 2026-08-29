# ADR 0007: Falhar aberto sem config, fechado com config inválida

- **Status:** proposto
- **Implementação:** já em código
- **Data:** 2026-08-28

## Contexto

Um projeto pode querer só os agentes, sem a governança completa. Travar esse
caso mata a adoção incremental. Por outro lado, configuração presente e
malformada é o pior cenário possível: o projeto acredita estar protegido e não
está.

## Decisão

| Situação | Comportamento |
|---|---|
| `ownership.json` ausente | exit 0 + aviso |
| `ownership.json` presente e malformado | **exit 2** |
| `verify.sh` ausente | exit 0 + aviso |
| dependência do guarda ausente (ex.: `jq`) | exit 0 + aviso |

Ausência é escolha legítima. Malformação é defeito.

## Consequências

**Positivas**

- Adoção incremental viável: instala o plugin, usa os agentes, adiciona
  governança depois.
- Erro de digitação não vira falso senso de proteção.

**Negativas**

- O caso "eu achei que tinha configurado, mas o arquivo está no caminho errado"
  degrada para aberto e passa despercebido se o aviso não chegar ao humano.

## Alternativas descartadas

- **Falhar fechado sempre.** Trava adoção incremental e transforma o plugin em
  tudo-ou-nada.
- **Falhar aberto sempre.** `ownership.json` quebrado desprotegeria o projeto
  em silêncio, que é exatamente o que esta decisão evita.

## Riscos em aberto

**O canal do aviso não foi verificado.** Em hooks, stderr de um processo que sai
com 0 nem sempre chega ao humano — em vários eventos vai só para o log de debug,
e nesse caso a metade "aviso" desta decisão não existe. Verificar qual mecanismo
torna o aviso visível ao usuário e ajustar. Enquanto isso, o modo aberto é mais
silencioso do que esta decisão pressupõe.
