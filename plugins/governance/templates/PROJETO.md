# PROJETO

Declaração do projeto. **Lido por todo agente, toda sessão.** É o que torna a
ferramenta reutilizável: nada aqui está no plugin, tudo aqui é deste sistema.

Preencha com `/brainstorming` antes de escrever qualquer código. Campo vazio é
bloqueio, não pendência.

Marcas obrigatórias em toda afirmação sobre o sistema: `[HOJE]` vale agora ·
`[FASE N]` **não vale hoje**. Afirmação sem marca é lida como presente.

---

## Identidade

- **Nome:**
- **Uma frase:** o que o sistema faz, para quem
- **Repositório:**

## Stack

- **Linguagem e framework:**
- **Persistência:**
- **Pack aplicável:** `springboot` | `node` | nenhum `[NÃO EXISTE]`
- **Comando de verificação:** o que `verify.sh` roda

## Atores

Quem usa o sistema. Cada ator gera pelo menos um fluxo a ser auditado — se um
ator não tem fluxo escrito, a auditoria não cobre ele.

| Ator | O que ele quer | Fluxos |
|---|---|---|
| | | |

## Fases

O eixo que separa achado de ausência esperada. A auditoria compara o código
contra o fluxo **filtrado até a fase corrente**; o que está acima é ausência
esperada, não defeito.

| Fase | Escopo | Status |
|---|---|---|
| 1 | | corrente |
| 2 | | futura |

**Fase corrente:**

## Perfil de adoção

- [ ] **1 — Solo.** Um agente por vez. Guarda de propriedade, gate de verify,
      auditoria de fluxo, marca temporal. Cobre C002–C006.
- [ ] **2 — Paralelo leve.** Um implementador; revisores read-only.
      `[BLOQUEADO]` falta destino durável do parecer.
- [ ] **3 — Paralelo pesado.** N sessões, uma por worktree.
      `[BLOQUEADO]` falta identidade do agente (ADR 0008).

Comece no 1. Suba quando um cenário do catálogo doer, não por antecipação.

## Ferramentas ligadas

| Ferramenta | Ligada? | Configuração |
|---|---|---|
| superpowers | | telemetria desligada |
| ponytail | | intensidade `lite`, always-on **off** |
| archify | | versão pinada: |
| governança | | perfil acima |

## Fronteiras arquiteturais

Portas de sentido único. Cada uma precisa de ADR antes do primeiro código que a
atravessa.

| Fronteira | ADR | Decidido? |
|---|---|---|
| módulos e seus limites | | |
| persistência | | |
| autenticação e autorização | | |
| integrações externas | | |

Para cada fronteira, a regra de três vias: vou trocar o outro lado
(**adaptador**) · vale para toda chamada (**interceptador**) · nenhum dos dois
e o leitor precisa ver (**chamada direta**).

## Propriedade

Fonte do `.claude/ownership.json`. Caminho protegido é o que o implementador
**nunca** escreve.

```json
{
  "protected": ["specs/**", "docs/adr/**", "PROJETO.md", "lifecycle/**",
                "scripts/verify.sh", ".claude/**"],
  "modules": {
    "implementer": ["src/main/**"],
    "qa": ["src/test/**"],
    "architect": ["docs/adr/**", "lifecycle/**"]
  },
  "verify": "./scripts/verify.sh"
}
```

## Definição de pronto

O que `verify.sh` precisa provar. Cada linha é uma camada, e cada camada é
incapaz de provar o que a outra prova.

| Camada | Prova | Ligada? |
|---|---|---|
| unidade e integração | o comportamento funciona | |
| contrato / web | o publicado é o especificado | |
| ArchUnit | a estrutura obedece o ADR | |
| lifecycle validado | todo estado tem entrada e saída | |
| fluxo por ator | **falta alguma coisa?** | manual, por fase |

As quatro primeiras respondem "quebrou?". Só a última responde "falta?".

## Linguagem do domínio

Glossário. Termo ambíguo aqui vira código ambíguo depois, e agente que não tem
a palavra usa vinte para dizer uma.

| Termo | Significa | Não confundir com |
|---|---|---|
| | | |
