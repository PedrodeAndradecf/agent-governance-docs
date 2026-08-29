# ADR 0013: Framework como ponto de partida, com colheita sob demanda

- **Status:** proposto
- **Implementação:** não iniciada
- **Data:** 2026-08-28
- **Decide:** Arquiteto rascunha, humano ratifica

## Contexto

Este repositório não acompanha os projetos. Ele **inicia** projetos: um SaaS ou
app novo nasce em repo próprio, e o framework é instalado ali para que o time
comece do mesmo lugar, com o mesmo preparo, organização e planejamento.

Isso é a diferença entre biblioteca e scaffold, e ela tinha ficado implícita.

Também resolve por si três coisas que estavam tratadas como problema:

- **Um repo ou vários** vira decisão do projeto, expressa no `ownership.json`
  gerado, não decisão do framework.
- **Qual ferramenta de disciplina** vira campo do `PROJETO.md`, não escolha
  nossa.
- **Genérico × específico** (ADR 0006) fica sem ambiguidade: o que é gerado
  pelo bootstrap é específico; o que fica no repo é genérico.

O problema que a formulação cria: se o framework sai de cena depois do
bootstrap, nada leva de volta o que o projeto aprendeu. **Ida sem volta é
scaffold, e scaffold envelhece.**

## Decisão

### Duas classes de entrega

**Instalado, atualizável** — guardas, agentes, skills, packs por stack. São
genéricos, chegam por plugin com versão pinada pelo projeto, e recebem
correção quando o projeto decide subir de versão.

**Gerado, do projeto para sempre** — `PROJETO.md`, `ownership.json`,
`verify.sh`, `constitution.md`, ADRs. Nascem do bootstrap e passam a ser do
sistema. Ninguém atualiza por cima.

Em uma frase: **o time é instalado, a memória é gerada.**

### O time não se divide entre repos

Cada repo instala o time inteiro. O `ownership.json` daquele repo define o que
cada papel toca ali — `src/main/java` num, `src/app` noutro. Um papel só,
escopo por configuração. Se fossem times diferentes, haveria duas definições de
Arquiteto para manter.

### Colheita sob demanda

`SISTEMA.md` e `FRICCAO.md` acumulam continuamente, sem esforço, porque já
fazem parte do fluxo. A colheita é disparada **quando o humano percebe que
juntou o suficiente** — no mesmo dia ou semanas depois.

Cadência fixa foi descartada: produz colheita vazia, e ritual sem conteúdo é o
que faz alguém parar de rodar. Sob demanda mantém o sinal alto, porque o
"percebi que aprendemos algo" é justamente o julgamento que o agente não faz
sozinho.

Propriedades obrigatórias do comando:

- **Marca d'água.** Registra até onde já colheu; a próxima execução processa só
  o que entrou desde então. Sem isso, a segunda rodada repropõe tudo.
- **Não destrutivo.** Lê e propõe; nunca apaga do `SISTEMA.md`. Registro local
  não some por ter virado regra global.
- **Saída é PR classificado** no repo do framework: entrada de catálogo,
  melhoria de pack, ou remoção.
- **Ratificação humana.** Decidir que uma lição vale para todos os projetos
  futuros é a decisão mais irreversível do sistema.

### O que volta e o que não volta

| Volta | Não volta |
|---|---|
| entrada de catálogo (defeito do método) | bug de domínio |
| regra que virou teste e generaliza | teste específico do sistema |
| tensão entre ferramentas com 3+ ocorrências | anedota de uma ocorrência |
| melhoria no pack da stack | decisão daquele produto |
| **remoção** de proteção que nunca disparou | — |

A última linha impede o framework de só crescer. Colheita que só adiciona
produz algo que ninguém instala no quinto projeto.

### Critério de promoção e arbitragem

Observado uma vez, num projeto: **candidato**. Observado em dois contextos
diferentes: **regra**.

Isso resolve o conflito entre colheitas simultâneas de projetos distintos — dois
sistemas propondo regras opostas, cada um certo no seu contexto. E é a única
forma honesta de distinguir defeito do método de defeito do sistema, porque um
projeto sozinho nunca consegue separar os dois.

## Consequências

**Positivas**

- O ciclo fecha: projeto 4 nasce com o que os três anteriores aprenderam.
- O framework tem mecanismo de encolher, não só de crescer.
- Ritual humano pequeno e concreto: uma revisão de PR por colheita.

**Negativas**

- Correção nos artefatos **gerados** não chega em projeto já iniciado, por
  construção. Um bug no template de `verify.sh` fica em todos os projetos
  criados antes da correção.
- Sob demanda depende do humano lembrar. É o único mecanismo do desenho sem
  gatilho, e portanto o mais provável de nunca acontecer.
- A colheita vira o gargalo humano se vários projetos rodarem ao mesmo tempo.

## Alternativas descartadas

- **Colheita no fim do projeto.** SaaS não acaba; o aprendizado azeda e sobra o
  fato sem a causa; e o projeto seguinte já começou com a bagagem velha.
- **Colheita em cadência fixa (fim de fase).** Produz rodadas vazias.
- **Colheita automática, sem ratificação.** Promover uma lição a regra global é
  irreversível na prática; agente não decide isso.
- **Framework acompanhando o projeto como dependência viva.** Acopla o projeto
  à evolução do framework e desfaz a premissa de que a memória é do sistema.

## Riscos em aberto

**A lembrança.** Mitigação proposta, e ela precisa ser leve: quando o
`FRICCAO.md` bater três ocorrências da mesma tensão, ou o `SISTEMA.md` ganhar
seção nova de causa raiz, o hook de `SessionStart` menciona que há material não
colhido. Não bloqueia, não insiste. Colheita é julgamento, então não vira hook
bloqueante — mas a **visibilidade** do material acumulado pode ser mecanismo, e
deve ser.

**Nada disso foi exercitado.** Não existe segundo projeto ainda, e a promessa
de "não reconstruir" só é testável na segunda instalação. Até lá, este ADR é
desenho, não resultado.
