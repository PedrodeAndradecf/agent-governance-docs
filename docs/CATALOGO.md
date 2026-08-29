# Catálogo de cenários

Falhas observadas, com o que fazer em cada uma. Entrada nova só entra depois de
**observada** pelo menos uma vez — cenário deduzido vai para hipóteses, não para
o catálogo.

Marca temporal: `[HOJE]` existe · `[PROPOSTO]` decidido, não implementado ·
`[NÃO EXISTE]` necessário, sem decisão. Texto sem marca descreve o **cenário**,
não o framework.

## Alvo de execução

Padrão: sessões em background, uma por worktree, com paralelismo real de
escrita. Revisão: subagentes na mesma sessão, read-only, sobre o mesmo diff.

## As quatro vias

| Via | Mecanismo | Quando |
|---|---|---|
| **Isolar** | worktree, instância dedicada | o recurso pode ser duplicado |
| **Impedir** | hook bloqueante em `PreToolUse` | a regra é predicado barato sobre o evento |
| **Detectar** | agente, auditoria de fluxo | exige julgamento ou contexto amplo |
| **Recuperar** | procedimento escrito | as três acima não cabem |

Ordem de preferência: **isolar > impedir > detectar > recuperar**. Isolar
elimina a classe e não cobra em runtime. Impedir é barato e rígido, e bloqueio
errado não tem canal de apelação. Detectar é caro e sempre tardio: o trabalho já
foi feito. Recuperar depende de alguém ler o procedimento na hora certa, que é a
suposição mais frágil.

## Índice

| ID | Cenário | Vias | Precisa de paralelismo? |
|---|---|---|---|
| [C001](#c001) | Estado durável compartilhado fora do worktree | isolar, impedir, recuperar | **sim** |
| [C002](#c002) | Implementador ajusta a spec ao que construiu | impedir | não |
| [C003](#c003) | Agente declara pronto o que não fez | impedir, detectar | não |
| [C004](#c004) | Afirmação de fase futura escrita no presente | detectar, impedir | não |
| [C005](#c005) | Estado alcançável sem transição | detectar | não |
| [C006](#c006) | Código que nenhum fluxo alcança | detectar | não |

**Cinco dos seis existem com um agente só.** Só o C001 exige paralelismo. Isso
contraria a intuição que originou o projeto: o valor está em completude e
veracidade de artefato, não em coordenação de concorrência.

## Perfis de adoção

**Perfil 1 — Solo.** Um agente por vez, sessão interativa. Liga guarda de
propriedade `[HOJE]`, gate de verify `[HOJE]`, auditoria de fluxo `[PROPOSTO]`,
marca temporal `[PROPOSTO]`. Cobre C002–C006.

**Perfil 2 — Paralelo leve.** Um implementador escrevendo, revisores read-only
como subagentes. Adiciona portão de revisão paralela e protocolo de parecer
`[NÃO EXISTE]`. Detecção mais cedo, mesma cobertura.

**Perfil 3 — Paralelo pesado.** N sessões em background, uma por worktree.
Adiciona identidade por agente `[PROPOSTO, bloqueado — ADR 0008]`, isolamento de
instância e nota de recuperação `[NÃO EXISTE]`. Cobre tudo, incluindo C001. Só
compensa quando a revisão humana não é o gargalo.

O perfil escolhe **quanto aparato de concorrência**, nunca quanta qualidade.
Verificação e completude entram em todos, inclusive solo — se o solo puder abrir
mão de ArchUnit, a catraca destrava e o nível cai no primeiro projeto apressado.

**Regra de adoção:** comece no perfil 1. Suba quando um cenário doer de verdade,
não por antecipação.

---

<a id="c001"></a>
## C001: Estado durável compartilhado fora do worktree

**Precisa de paralelismo:** sim, só existe no perfil 3.

## Cenário

Dois agentes em worktrees separados operam sobre um recurso com estado durável
que não é duplicado pelo worktree: banco de dados, histórico de migration,
volume Docker, porta de rede, cache de dependência, lockfile de ambiente.

Caso observado: um agente edita a migration `V1`, outro sobe a aplicação. O
`flyway_schema_history` guarda o checksum antigo e a subida falha na validação.

## Sintoma

Falha em tempo de execução, no worktree de **quem não fez a mudança**.
Invisível no diff. A mensagem de erro aponta para o schema, não para a causa.

## Por que escapa

Worktree isola o **checkout**, não a infraestrutura. Essa premissa nunca foi
examinada quando o paralelismo foi desenhado — é a suposição implícita mais
perigosa do perfil 3.

O agente afetado vai gastar contexto depurando um problema que não é dele, e a
correção plausível dele (mexer no schema) piora a situação.

Estruturalmente é o mesmo defeito de C003 e C004: duas fontes de verdade
discordando sem árbitro. Aqui, o histórico aplicado contra os arquivos de
migration.

## Detecção

Nenhuma antes da execução. É a pior propriedade deste cenário.

## Estratégias

**Isolar** — uma instância por worktree (Testcontainers, compose com sufixo).
Elimina a classe inteira, não só migration: some também colisão de porta e
contaminação de dados de teste. `[NÃO EXISTE]`

**Impedir** — hook bloqueando edição de migration já aplicada. Precisa
consultar o banco para saber o que foi aplicado, em caminho quente, o que
briga com o limite de latência do `PreToolUse`. Só vale depois que o schema
estabilizar, quando editar deixa de ser legítimo. `[NÃO EXISTE]`

**Serializar** — `db/migration/**` como módulo de dono único no
`ownership.json`. Usa mecanismo que já existe e converte concorrência em
propriedade. Não protege quem apenas **sobe** a aplicação. `[HOJE]`

**Recuperar** — nota de recuperação junto da mudança, no mesmo commit, não no
chat. `[NÃO EXISTE]`

## Como escolher

Pela estabilidade do recurso, não pela gravidade:

- schema ainda mexendo, nada em produção → **isolar**; editar `V1` é legítimo
- schema estabilizado ou algo depende do estado → **impedir**; regra é sempre
  `V+1`, migration aplicada é imutável

A regra "migration aplicada é imutável" é do Flyway e vale para humano também.
O que muda com agentes é a frequência, não a natureza.

## Generalização

Migration é o primeiro caso de uma classe: **ação com efeito fora do worktree**.
Mesma forma em mudança de porta, variável de ambiente, volume, versão em
lockfile.

Um caso observado não sustenta a generalização. A classe fica registrada aqui
como hipótese; promove a regra quando aparecer o segundo caso.

---

<a id="c002"></a>
## C002: Implementador ajusta a spec ao que construiu

**Precisa de paralelismo:** não.

## Cenário

O agente encontra atrito entre a especificação e o que implementou, e resolve
editando a especificação. O resultado fica internamente coerente e
completamente errado.

## Sintoma

Nenhum. É o pior caso do catálogo nesse aspecto: tudo passa, tudo bate, e a
divergência com a intenção original desapareceu junto com a evidência dela.

## Por que escapa

A spec é o oráculo de intenção. Quando o oráculo é editável por quem ele
julga, ele deixa de ser oráculo. Nenhum teste detecta isso, porque testes são
derivados da spec — se a spec mudou, os testes concordam com ela.

Modo de falha mais grave do desenvolvimento spec-driven.

## Detecção

Só por revisão de histórico, e tarde. Diff em `specs/` no mesmo commit que
código é o sinal, mas quem olha?

## Estratégias

**Impedir** — o implementador não tem permissão de escrita em `specs/` nem
`docs/adr/`. Imposto por `PreToolUse` com exit 2, não por instrução em
markdown. `[HOJE]`

Ausente do catálogo por decisão: detectar não serve aqui. Quando a detecção
acontece, a informação original já foi destruída.

## Como escolher

Não há escolha. Impedir é a única via válida, e é a razão de existir do ADR
0001.

O que **tem** escolha é o que acontece quando o atrito é legítimo — a spec
realmente estava errada. Aí o caminho não é editar, é a saída de três vias do
Arquiteto: *spec errada* (emenda proposta, humano ratifica) / *implementação
errada* (volta ao implementador) / *spec certa, custo alto* (registra dívida).

## Nota

Este cenário é o único do catálogo com solução completa e implementada. Serve
de referência de qualidade para os outros: regra expressável como predicado
sobre caminho de arquivo, imposta em `PreToolUse`, custo próximo de zero.

---

<a id="c003"></a>
## C003: Agente declara pronto o que não fez

**Precisa de paralelismo:** não.

## Cenário

O agente conclui o trabalho e escreve um artefato de conclusão — CHANGELOG,
relatório, resumo — afirmando entregas que não existem no repositório.

Caso observado: o CHANGELOG registrou "marketplace com plugin único" e "CI que
valida JSON e sintaxe POSIX" como fatos consumados. Nenhum dos dois estava
commitado. No mesmo repositório, `docs/VERIFICACAO.md` admitia honestamente que
dois itens da definição de pronto nunca foram executados.

## Sintoma

Dois artefatos do mesmo repositório afirmando coisas incompatíveis. Nenhum
árbitro entre eles. Quem lê primeiro define o que acredita.

## Por que escapa

A distribuição dos erros é o dado interessante: **passou honesto exatamente
onde havia oráculo, e falhou exatamente onde havia só prosa.** Os itens da
definição de pronto com teste comportamental foram cumpridos e relatados
corretamente. Os que só tinham descrição textual foram declarados prontos.

Não é desonestidade do agente. É ausência de verificação onde ela era possível.

## Detecção

Auditoria cruzada entre artefatos, ou tentativa de uso real — que é como o caso
foi descoberto: alguém reparou que o primeiro comando do README não funcionaria.

## Estratégias

**Impedir** — artefato de conclusão vira **derivado**, não autoral. Nenhuma
linha de CHANGELOG sem job de CI que a sustente. `[PROPOSTO]`

**Detectar** — a auditoria de fluxo (C004, C005) encontra isso de graça quando
percorre o caminho de instalação prometido pelo README. `[PROPOSTO]`

## Como escolher

Impedir onde o artefato é mecanicamente derivável (CHANGELOG a partir de
release e CI). Detectar para o resto, porque nem todo relatório é derivável.

## Regra geral

**A definição de pronto nunca pode ser o resumo do agente.** Se um item da
definição de pronto não tem oráculo, ele não é um item da definição de pronto:
é uma intenção, e deve estar marcado como tal.

---

<a id="c004"></a>
## C004: Afirmação de fase futura escrita no presente

**Precisa de paralelismo:** não.

## Cenário

Um artefato descreve comportamento que ainda não vale — intenção, fase futura,
design abandonado — em linguagem que o leitor entende como descrição do estado
atual.

Caso observado: o comentário de `ProductService.findDetail` afirma que peça
arquivada segue acessível por link direto. O método chama
`findByIdAndDeletedAtIsNull`, que filtra arquivadas. Verificado em runtime:
retorna 404, não os 200-com-status que o comentário promete.

## Sintoma

Dois leitores chegam a conclusões opostas sobre o mesmo comportamento, cada um
com base numa fonte legítima do repositório.

## Por que escapa

**Todo texto sem marca temporal é lido no presente.** O default da linguagem
natural é o presente do indicativo, e quem lê acredita no comentário — é a
fonte mais próxima do código.

O comentário não era mentira quando foi escrito. Era intenção. Virou mentira
porque o formato não tinha onde guardar o tempo. Defeito de suporte, não de
autor.

`// TODO` não cobre isso: marca que falta trabalho, não que a frase ao lado é
falsa hoje. Um comentário descritivo bem escrito não parece pendência nenhuma.

## Por que é o mais perigoso do catálogo

É o único que **envenena os próximos agentes**, não só os humanos. Um agente
que lê o comentário implementa contra uma premissa falsa, e o defeito se
propaga para código novo.

Também é a origem provável do problema de fase: artefato que descreve o estado
final no presente induz comportamento de Fase 4 em código de Fase 1.

## Detecção

**Grep de linha de base** — varredura por comentários que afirmam capacidade,
conferidos contra o código. O caso observado não é isolado, é o que alguém
encontrou.

**Auditoria de fluxo** — a classe entra como sexta categoria de achado, ao
lado de veracidade e completude.

## Estratégias

**Impedir** — `PostToolUse` não bloqueante: comentário com verbo de capacidade
sem marca de fase gera aviso. Heurística imperfeita de propósito; bloquear aqui
seria insuportável. `[NÃO EXISTE]`

**Detectar** — auditoria periódica e grep de linha de base. `[PROPOSTO]`

**Convenção** — marca como prefixo, nunca sufixo, porque quem lê em diff ou
grep vê o começo da linha. E o par importa: `[FASE N]` sozinho deixa o leitor
sem saber o que vale hoje. `[PROPOSTO — ADR 0011]`

```java
// [FASE 4] Peça arquivada segue acessível por link direto. NÃO VALE HOJE.
// [HOJE] findByIdAndDeletedAtIsNull — arquivada retorna 404.
```

## Duas correções, dois horizontes

O caso observado contém dois defeitos que precisam ser separados:

- **veracidade** — o comentário mente. Corrige **agora**, custa uma linha,
  qualquer que seja a fase. O Arquiteto faz por conta própria.
- **completude** — se peça arquivada *deve* seguir acessível é decisão de
  produto e pode legitimamente ser Fase 4. O Arquiteto **propõe** fase; o
  humano ratifica.

Regra de desempate sob dúvida: classificar como fase corrente e escalar, nunca
adiar por conta própria. Construir cedo é desperdício visível e recuperável;
adiar um gap da fase corrente é falha silenciosa em produção.

## Alcance

A convenção vale para toda camada, e cada uma precisa do próprio slot:

| Artefato | Slot | Estado |
|---|---|---|
| ADR | campos `Status` e `Implementação` | `[HOJE]` |
| Fluxo por ator | fase por passo e transição | `[PROPOSTO]` |
| Comentário em código | prefixo `[HOJE]` / `[FASE N]` | `[NÃO EXISTE]` |
| `constitution.md` | nenhum | `[NÃO EXISTE]` |

A última linha é a mais urgente: é lida por todo agente em toda sessão.

---

<a id="c005"></a>
## C005: Estado alcançável sem transição que entre ou saia

**Precisa de paralelismo:** não.

## Cenário

O sistema tem um estado, campo ou valor que pode ser alcançado, mas falta o
caminho de entrada, o de saída, ou ambos. Não é bug: o código faz corretamente
o que faz. É ausência.

Três casos observados, todos com a mesma forma:

- campo presente no DTO de Create e ausente no de Update — sem sinal de erro,
  porque os dois foram escritos à mão lado a lado
- `Product.archive(Instant)` existe; `unarchive` não existe em lugar nenhum.
  Combinado com o filtro na listagem da própria vendedora, arquivar é porta sem
  volta — ela não consegue ver, contar ou recuperar
- `Category.active` existe na entidade e no schema, e nenhum código jamais o
  altera: sem `deactivate()`, sem `PUT`, `PATCH` ou `DELETE` no controller

## Sintoma

Nenhum, no código. Aparece só quando um usuário real tenta fazer algo e não
encontra caminho.

## Por que escapa

**Teste é fechado sobre o código.** Verifica que o que existe funciona, e é
logicamente incapaz de verificar que algo deveria existir. `verify.sh`, exit 2
e CI são todos oráculos: respondem "quebrou?", nunca "falta?".

Nenhum portão do framework detecta ausência. Esse é o ponto cego estrutural que
originou a técnica do fluxo por ator.

## Detecção

**Fluxo por ator, percorrido contra o código.** É o único artefato construído
de fora do código, a partir da intenção do usuário. Por isso encontra ausência.

A caminhada tem duas direções e as duas rendem:

| Direção | Encontra |
|---|---|
| fluxo → código | o usuário precisa, o código não oferece — **este cenário** |
| código → fluxo | o código oferece, nenhum fluxo alcança — **C006** |

## Condição de validade

Se o fluxo for escrito **a partir do código**, ele não acha nada: apenas
re-deriva as suposições do código e confirma que o código faz o que faz.

Portanto o fluxo é artefato da camada 1 — escrito junto da spec, antes da
implementação, e protegido no `ownership.json` contra o implementador. Isso não
é organização, é a condição que torna a técnica válida. `[PROPOSTO]`

## Estratégias

**Detectar** — auditoria de fluxo em fronteira de fase e sempre que o artefato
de fluxo mudar. Não cabe em hook: exige ler o repositório, executar coisas e
julgar intenção, e a saída é uma lista de julgamentos, não exit 0 ou 2. É
agente. `[PROPOSTO]`

## Regra generalizada

**Todo estado alcançável precisa de caminho de entrada, caminho de saída, ou
marcação explícita de terminal.**

Generalização da Regra 1 do `planejamento.md` ("nenhum valor de enum sem
transição que chegue nele"), estendida a booleano, campo e qualquer estado.

## Registro de ausência

"Esquecido e decidido são indistinguíveis olhando o código." Três valores, não
dois:

| Estado da ausência | Significado |
|---|---|
| esquecida | achado, entra na fase corrente |
| adiada para fase N | conhecida, com destino |
| terminal por decisão | nunca vai existir, e isso é a decisão |

Sem o terceiro, a auditoria re-encontra os mesmos itens todo ciclo. Sem o
segundo, gap real vira "depois" e some.

## Fecho do ciclo

**Todo achado confirmado vira teste.** É assim que a camada 3 aprende com a
camada 1, e é o que evita pagar o custo desta auditoria de novo pelo mesmo
defeito.

---

<a id="c006"></a>
## C006: Código que nenhum fluxo alcança

**Precisa de paralelismo:** não.

## Cenário

Existe código correto, testável e alcançável tecnicamente, que nenhum fluxo de
usuário exercita. Caso observado: `ProductPhotoRepository.countByProductId`,
referenciado em lugar nenhum.

## Sintoma

Nenhum. Compila, passa em tudo, não faz nada.

## Por que escapa

É o **dual** de C005, e escapa pelo mesmo motivo invertido. Ferramenta de
cobertura mostra código não coberto por teste, não código não alcançado por
fluxo — e um método com teste unitário e nenhum chamador aparece como coberto.

## Detecção

Segunda direção da caminhada de fluxo: código → fluxo. Análise estática de
referência ajuda, mas dá falso positivo em ponto de extensão, API pública e
uso por reflexão.

## Estratégias

**Detectar** — sai de graça na mesma auditoria de C005, na direção inversa.
`[PROPOSTO]`

## Como classificar o achado

Órfão tem três causas com destinos diferentes, e confundi-las é o erro comum:

- **sobra** — restou de design anterior. Remove.
- **antecipação** — escrito para fase futura. É C004 em forma de código: existe
  sem marca temporal, e quem lê acha que é usado. Marca ou remove.
- **fluxo faltando** — o código está certo e o *fluxo* é que está incompleto.
  Vira achado de C005, não de C006.

O terceiro caso é o que justifica não deletar órfão automaticamente. Remover
código órfão sem classificar apaga a evidência de uma ausência no fluxo.

## Severidade

Menor. Não causa dano funcional. Entra no catálogo porque é **sinal barato**: a
presença de órfão indica que o fluxo e o código divergiram em algum ponto, e
esse ponto costuma ter um C005 por perto.

---
