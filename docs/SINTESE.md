# Síntese: o que estamos construindo

`[HOJE]` Documento de contexto. Quem chega aqui — humano ou agente — deve
conseguir entender o projeto sem ler o histórico das conversas que o geraram.

Marca temporal em uso: `[HOJE]` existe e funciona · `[PROPOSTO]` decidido, não
implementado · `[NÃO EXISTE]` identificado como necessário, sem decisão.

---

## 1. Objetivo

Uma metodologia versionada e reutilizável para construir sistemas com agentes,
**de modo que a qualidade não seja reconstruída a cada projeto novo.**

O produto não são as regras. É o **acúmulo**. Hoje toda correção feita num
agente morre no chat: projeto novo começa do zero, os mesmos defeitos são
redescobertos, os mesmos portões reescritos, e o nível de qualidade é função de
quanto alguém lembrou de exigir naquele dia.

O framework existe para que **uma correção feita uma vez nunca precise ser
feita de novo**. É uma catraca: só anda para frente, e o nível de partida do
projeto N+1 é o nível de chegada do projeto N.

### Tese central

Agente é competente e não confiável individualmente. Logo, a garantia não pode
vir do agente. Vem de fora dele: hook que impõe, oráculo que verifica, artefato
que registra.

Corolário operacional: **toda regra que puder ser imposta por mecanismo não
deve ficar em markdown.** Correção que mora em prosa é lida e esquecida;
correção que vira hook ou teste persiste porque não depende de leitura.

---

## 2. As três camadas

| Camada | Responsabilidade | Dono |
|---|---|---|
| 1 — Artefato | o que é verdade sobre o sistema e a intenção | superpowers + nossos templates |
| 2 — Papéis e disciplina | quem faz o quê, e como se trabalha | superpowers (ADR 0012) |
| 3 — Execução | o que é permitido acontecer | hooks + worktrees (nosso) |

BMAD foi descartado por tentar ocupar as três (ADR 0003). Depois do ADR 0012,
**nós construímos só a camada 3, mais fase, completude e acúmulo.** Disciplina
vem de terceiro.

---

## 3. Mapa de artefatos

O eixo que organiza é **escopo**, porque é ele que decide o que atravessa
projetos e o que morre com eles.

### Escopo framework — atravessa todos os projetos

| Artefato | Papel | Estado |
|---|---|---|
| Catálogo de cenários | falhas observadas e como tratá-las | `[HOJE]` 6 entradas |
| ADRs do framework | decisões que governam o próprio framework | `[HOJE]` 8, todos `proposto` |
| Packs por stack | ArchUnit, verify, contrato prontos por ecossistema | `[NÃO EXISTE]` |
| Agentes e hooks | os papéis e os portões | `[HOJE]` parcial |

**É aqui que mora o valor.** Tudo abaixo é instanciação.

### Escopo projeto — vive enquanto o sistema vive

| Artefato | Papel | Escreve | Lê |
|---|---|---|---|
| `constitution.md` | princípios inegociáveis | humano | todo agente, toda sessão |
| `planejamento.md` | fases, regras do projeto, pendências | humano | todos |
| `SISTEMA.md` | o que aprendemos construindo isto | Arquiteto | próximo agente |
| `docs/adr/` | decisões de arquitetura do sistema | Arquiteto rascunha, humano ratifica | todos |
| lifecycle (IR Archify) | estados e transições do domínio | Arquiteto | humano, auditor |
| `debt.md` | dívida aceita conscientemente | Arquiteto | humano |
| `.claude/ownership.json` | mapa de propriedade | humano | hooks |
| `scripts/verify.sh` | definição de pronto executável | humano | hooks, CI |

### Escopo feature — vive até entregar

| Artefato | Papel |
|---|---|
| `specs/NNN/spec.md` | o quê e o porquê |
| `specs/NNN/plan.md` | o como: stack, contratos, portas |
| `specs/NNN/tasks.md` | unidades despacháveis |
| fluxo por ator | a jornada, passo a passo, com fase por transição |

### Escopo story — efêmero

| Artefato | Papel | Estado |
|---|---|---|
| parecer dos revisores | veto, achado, proposta de fase | `[NÃO EXISTE]` destino durável |

---

## 4. O loop de acúmulo

A peça que faz a catraca girar, e que estava implícita até agora.

```
agente erra num projeto
        ↓
[projeto]   §13 do SISTEMA.md: causa raiz + mecanismo generalizável
        ↓                     (Arquiteto escreve, humano ratifica)
        ↓  promoção — só se a causa não for específica deste sistema
        ↓
[framework] entrada no catálogo + uma das quatro vias
        ↓
projeto seguinte já nasce protegido
```

`SISTEMA.md` é a **dobradiça**. Ele é o registro por projeto do que foi
aprendido; o catálogo é o registro do framework. Sem a dobradiça, aprendizado
fica preso no projeto onde aconteceu.

Isso já acontece informalmente: a "Regra 1" do `planejamento.md` do brechó
("nenhum valor de enum sem transição que chegue nele") é uma regra de projeto
que generalizou e virou o C005. A promoção existe; o que falta é ser
procedimento em vez de acaso.

**Critério de promoção:** a causa raiz é do sistema ou do método? Bug de
domínio fica no projeto. Defeito que qualquer projeto com agentes teria sobe
para o catálogo.

---

## 5. As quatro vias

Todo cenário do catálogo se resolve por uma ou mais, em ordem crescente de
custo: **isolar > impedir > detectar > recuperar**.

Isolar elimina a classe e não cobra em runtime. Impedir é barato e rígido, e
bloqueio errado não tem canal de apelação. Detectar é caro e sempre tardio.
Recuperar depende de alguém ler o procedimento na hora certa.

---

## 6. Papéis

| Agente | Lê | Escreve | Veta | Muda spec/ADR |
|---|---|---|---|---|
| Implementador | tudo | só seu módulo | não | **nunca** |
| Ponytail | só o diff | nada | não | não |
| Arquiteto | tudo | rascunho de ADR, `SISTEMA.md`, `debt.md` | sim, em fronteira | propõe, humano ratifica |
| QA | tudo | apenas testes | sim, teste vermelho | não |
| Segurança | tudo | nada | sim, finding alto | não |

Invariantes: **quem escreve código não aprova código**; o implementador não tem
permissão de escrita em `specs/` nem `docs/adr/`.

`[NÃO EXISTE]` **Protocolo.** A tabela é autoridade, não procedimento. Falta
para cada papel: gatilho, entrada, formato do parecer, e o que significa
abstenção. Autoridade sem protocolo é organograma.

### Saída do Arquiteto — quatro vias

1. spec estava errada → emenda proposta, humano ratifica
2. implementação está errada → volta ao implementador
3. spec certa, custo alto → registra dívida, não bloqueia
4. contradição entre artefatos → **requer humano**, não escolhe sozinho

Em achado de fluxo, separa sempre **veracidade** (artefato mente sobre o código:
corrige agora, qualquer fase) de **completude** (falta capacidade: propõe fase,
humano ratifica).

Desempate sob dúvida de fase: classifica como fase corrente e escala. Construir
cedo é desperdício visível e recuperável; adiar gap da fase corrente é falha
silenciosa em produção.

---

## 7. Verificação

`[NÃO EXISTE]` O modelo completo. Hoje há só `verify.sh` como bloco único, que
é script, não modelo.

Cada camada prova uma coisa e é incapaz de provar as outras:

| Camada | Prova | Estado |
|---|---|---|
| Unidade e integração | o comportamento implementado funciona | por projeto |
| Contrato / web | o publicado é o especificado | `[NÃO EXISTE]` |
| ArchUnit | a estrutura obedece o ADR | `[NÃO EXISTE]` |
| Lifecycle validado | todo estado tem entrada e saída | `[PROPOSTO]` |
| Fluxo por ator | **falta alguma coisa?** | `[PROPOSTO]` |

As quatro primeiras respondem "quebrou?". Só a última responde "falta?" — teste
é fechado sobre o código e é logicamente incapaz de verificar que algo deveria
existir.

ArchUnit é a ponte entre camada 1 e 3: sem ele, ADR é prosa e depende de um
revisor lembrar.

`[PROPOSTO]` **Packs por stack.** O modelo é genérico e sempre presente; a
instanciação vem pronta por ecossistema (`springboot` traz ArchUnit derivado
dos ADRs, `verify.sh` com maven, setup de contrato). Sem pack pronto, cada
projeto reconstrói — que é o que o objetivo proíbe.

---

## 8. Formato dos artefatos visuais

Divisão por **modo de leitura**, não por tamanho:

- **fluxo por ator** → texto estruturado. Lido como lista, percorrido passo a
  passo. É o que encontra ausência.
- **lifecycle e arquitetura** → Archify (JSON IR tipado + render). Lido como
  desenho: estado sem seta de saída salta aos olhos.

Falta é muito mais fácil de **ver** do que de ler. Numa lista, a transição
inexistente simplesmente não está escrita e ninguém repara. Num grafo, o nó
fica órfão na tela. Isso torna o C005 barato.

**Regra de tamanho:** um artefato por fluxo, nunca um por sistema. Se não cabe
numa tela e você não percorre em voz alta, divide por ator ou por jornada.

**Regra de uso do Archify:** o IR normativo é escrito a partir da spec, nunca
derivado do repositório. Diagrama derivado do código só re-deriva as suposições
do código e não acha ausência nenhuma — o modo "mapeie este repositório" é
onboarding e PR review, não fonte de camada 1.

Archify **não** verifica o sistema: valida schema, layout e rota do próprio
desenho. Não substitui ArchUnit nem teste de contrato.

---

## 9. Ferramentas

| Ferramenta | Decisão | Runbook |
|---|---|---|
| BMAD | descartado (ADR 0003) | n/a |
| Spec Kit | **superado pelo ADR 0012** — superpowers ocupa a camada 1 | n/a |
| superpowers | disciplina, no back (ADR 0012) | `[NÃO EXISTE]` |
| gstack | parqueado para front pesado (ADR 0012) | n/a |
| Ponytail | aconselha, nunca bloqueia (ADR 0005) | `[NÃO EXISTE]` |
| Archify | formato de lifecycle e arquitetura | `[NÃO EXISTE]` |

`[NÃO EXISTE]` **Categoria de documento faltante: runbook.** ADR registra
decisão, catálogo registra falha, nenhum registra **operação** — como instalar,
quem invoca, em que fase, o que fazer com a saída, o que fazer quando falha.

Falha mais séria da tabela: nenhuma ferramenta adotada tem runbook. Decisão de
adoção sem procedimento de uso é o mesmo buraco dos papéis (autoridade sem
protocolo), na versão para ferramentas.

---

## 10. Estado real

### Funciona
Guarda de propriedade em `PreToolUse`, gate de `verify.sh` em `Stop`,
degradação aberta/fechada, definição dos agentes com ferramentas restritas.
Dez cenários validados — **com payload sintético**.

### Não funciona
`.claude-plugin/marketplace.json` não existe, então o primeiro comando do README
falha. Sem CI. CHANGELOG afirma as duas coisas como feitas.

### Bloqueios, em ordem de custo
1. **Destino durável do parecer.** Trava o perfil 2, o ADR 0005 e o protocolo
   dos papéis. É a lacuna mais cara. Proposta pendente: comentário em PR.
2. **Camada de distribuição.** `marketplace.json` e CI não existiam; foram
   escritos, faltam validar com `claude plugin validate`.
3. **Identidade do agente** (ADR 0008). Deixou de bloquear com um terminal por
   agente: o worktree é a chave. Falta confirmar contra payload real.

### Achado que reorienta o projeto
**Cinco dos seis cenários do catálogo existem com um agente só.** O valor está
majoritariamente em completude e veracidade de artefato, não em coordenação de
concorrência. O perfil solo já justifica a adoção.

---

## 10.1 O que este framework é, depois do ADR 0013

Ponto de partida, não companhia. Instalado num repo novo, prepara o time, e o
projeto segue a vida dele. **O time é instalado; a memória é gerada.**

O ciclo fecha pela colheita sob demanda: você dispara quando percebe que juntou
o suficiente, e ela devolve um PR classificado para o catálogo. Ida sem volta
seria scaffold, e scaffold envelhece.

## 11. Risco sobre este documento

Ele é o artefato com mais leitores e maior vida útil do projeto, e é exatamente
o tipo de texto que descreve o estado final no presente e induz comportamento de
Fase 4 em código de Fase 1. Por isso toda afirmação sobre o framework carrega
marca. **Afirmação nova sem marca é defeito, não estilo.**
