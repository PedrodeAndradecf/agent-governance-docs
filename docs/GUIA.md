# Guia operacional

`[HOJE]` Como ligar a ferramenta num projeto novo e trabalhar com ela.

Marcas: `[HOJE]` existe · `[PROPOSTO]` decidido, não implementado ·
`[NÃO EXISTE]` necessário, sem decisão · `[VERIFICAR]` comando ou fato que
precisa ser confirmado antes do uso.

---

## 1. Divisão de responsabilidade

Quatro peças, sem sobreposição. Se duas quiserem o mesmo trabalho, é defeito.

| Peça | Responde | Camada |
|---|---|---|
| **superpowers** | como o agente deve trabalhar | disciplina |
| **ponytail** | esse código precisa existir? | corte |
| **archify** | como o domínio se comporta, visualmente | artefato |
| **governança (nosso)** | o que o agente **não pode** fazer, em que fase, e o que aprendemos | imposição |

A regra que evita briga: **só a nossa camada bloqueia.** superpowers e ponytail
aconselham; archify descreve. Bloqueio é hook, e hook é nosso.

---

## 2. Instalação — uma vez por máquina

### superpowers

```
/plugin install superpowers@claude-plugins-official
```

Desligue a telemetria antes do primeiro uso:

```
export SUPERPOWERS_DISABLE_TELEMETRY=1
```

### archify

```
npx skills add tt-a1i/archify -g
```

Alternativa sem instalar skill, usando só o CLI (preferível se você quer o IR
sem superfície de sessão):

```
git clone https://github.com/tt-a1i/archify.git
node archify/bin/archify.mjs doctor
```

**Pinar versão.** Está em `v2.16.0-dev.0` e move rápido.

### ponytail

```
/plugin marketplace add DietrichGebert/ponytail
```

`[VERIFICAR]` confirmar o comando no repositório antes de rodar.

**Configuração obrigatória:** intensidade `lite`, always-on **desligado**.
Motivo no ADR 0005 — a cadeia de interceptadores precisa ser nossa, e hook de
terceiro roda como processo do SO com suas permissões.

### governança

Enquanto o marketplace não existe `[NÃO EXISTE]`, cópia manual: os quatro
scripts de `hooks/` para `.claude/hooks/` do projeto, e o bloco de hooks para
`.claude/settings.json`. O formato é idêntico entre plugin e settings, então
migrar depois é mover o bloco.

---

## 3. Ligar num projeto

### Passo 1 — declarar o projeto

Copie `PROJETO.template.md` para `PROJETO.md` na raiz e preencha. **Nada
funciona antes disso**: é ele que diz a stack (decide `verify.sh` e ArchUnit),
as fases (decide o que é achado e o que é ausência esperada), o perfil (decide
quanto aparato liga) e os atores (decide quais fluxos auditar).

Use `/brainstorming` do superpowers para preencher, não escreva sozinho. Ele
existe para extrair o que você quer de fato em vez do que você acha que quer.

### Passo 2 — instalar os arquivos de projeto

```
.claude/ownership.json      # mapa de propriedade
.claude/settings.json       # a cadeia de hooks
.claude/hooks/              # os guardas
scripts/verify.sh           # definição de pronto executável
docs/adr/                   # decisões, começa com o template
docs/SISTEMA.md             # o que aprendemos (começa vazio)
docs/FRICCAO.md             # o que não encaixou entre as ferramentas
specs/                      # fluxos por ator e specs
lifecycle/                  # IR do archify
```

### Passo 3 — hook de log de payload

Ligue junto com os guardas, na primeira semana:

```sh
#!/usr/bin/env sh
cat > "/tmp/payload-$(date +%s%N).json"
exit 0
```

Isso resolve o ADR 0008 pelo uso em vez de por experimento isolado. Depois de
alguns dias, compare os payloads de sessão principal, subagente e worktree, e
decida a chave de identidade com dado real.

---

## 4. O fluxo, do zero

### Fase 0 — domínio antes de código

1. `PROJETO.md` preenchido e ratificado por você.
2. **Lifecycle no archify**, escrito a partir da intenção, nunca derivado de
   código (não há código ainda — essa é a janela em que a regra é grátis).
   Estado sem seta de saída aparece na tela.
3. ADRs iniciais: fronteiras de módulo, persistência, autenticação. Portas de
   sentido único primeiro.
4. `constitution.md` com a regra de três vias (adaptador / interceptador /
   chamada direta) e a convenção de marca temporal.

### Fase 1..N — por fatia vertical

1. `/brainstorming` → spec da fatia, aprovada por você.
2. **Fluxo por ator** em texto estruturado, com fase por passo. Escrito agora,
   antes do código. Protegido contra o implementador no `ownership.json`.
3. `/writing-plans` → tarefas pequenas.
4. Worktree, `/test-driven-development`, implementação.
5. **Portão:** `verify.sh` no `Stop` (nosso, bloqueia) + `/requesting-code-review`
   (superpowers) + parecer do ponytail (não bloqueia).
6. Fecha a branch.

### Fim de fase — auditoria

1. Percorrer cada fluxo por ator contra o código, nas duas direções.
2. Comparar contra o lifecycle: transição que existe no código e não no IR, ou
   o contrário.
3. `archify compare` entre o IR da fase anterior e o atual: mudança de
   arquitetura vira recibo de máquina em vez de invisível.
4. Achados classificados em veracidade (corrige agora) e completude (propõe
   fase, você ratifica).
5. Cada achado confirmado **vira teste**.
6. Causa raiz generalizável sobe para `SISTEMA.md` e, se for do método e não do
   sistema, promove para o catálogo.

---

## 5. Atrito conhecido, a observar

Não são bugs. São tensões previstas que só o uso resolve. Registre em
`FRICCAO.md` quando aparecerem.

| # | Tensão | O que observar |
|---|---|---|
| 1 | ponytail × regra de três vias | ele vai atacar porta e adaptador legítimos. Se acontecer 3×, a regra vira exemplo explícito na skill |
| 2 | subagentes do superpowers × identidade | `subagent-driven-development` despacha subagentes que **escrevem**. Sem identidade resolvida, o guarda degrada para só caminhos protegidos |
| 3 | archify × derivação | a ergonomia dele empurra para "mapeie o repositório". Isso invalida a auditoria de completude |
| 4 | superpowers × fase | ele não conhece fase. Vai propor completude de Fase 4 em código de Fase 1 |
| 5 | dois planos | `writing-plans` e `specs/` podem virar duas fontes de verdade |
| 6 | latência | o guarda gata toda escrita. Meça e registre |

**A tensão 2 é a mais séria hoje.** Com identidade não resolvida, o modelo de
propriedade por agente não está ativo — só a proteção de `specs/` e
`docs/adr/`. Isso ainda vale muito, mas não é o que a matriz de autoridade
promete. Não trate como se fosse.
