# ADR 0001: Impor política por mecanismo, não por instrução

- **Status:** proposto
- **Implementação:** já em código
- **Data:** 2026-08-28

## Contexto

Regras escritas em markdown para agentes são conselho, não lei. Um agente
instruído a não editar `specs/` edita `specs/` quando isso resolve o problema
dele, e o modo de falha mais grave do desenvolvimento spec-driven é exatamente
esse: o agente ajusta a especificação para casar com o que construiu.

O mesmo vale para conclusão de trabalho. Sem oráculo externo, "terminei"
significa "acho que terminei".

Restrição que viabiliza a alternativa: hooks do Claude Code rodam como
processos reais do sistema operacional, deterministicamente, em pontos fixos
do ciclo de vida — não quando o modelo decide. `PreToolUse` com exit 2 bloqueia
a chamada antes de ela executar e devolve o stderr ao modelo como motivo.

## Decisão

Toda regra que possa ser expressa como predicado sobre um evento de ferramenta
**deve** ser imposta por hook bloqueante. Markdown fica reservado a julgamento
que exige contexto amplo.

Critérios de fronteira:

- Se a regra pode ser violada por um agente que decidiu violá-la, ela está no
  lugar errado.
- Se a decisão exige ler o repositório para ser tomada, ela não cabe em hook
  de `PreToolUse`, que precisa ficar abaixo de ~500ms porque gata toda chamada
  que casa com o matcher. Vai para um agente.
- Mensagem de bloqueio explica **o que fazer**, não só o que foi negado. O
  modelo lê esse texto e age sobre ele.

## Consequências

**Positivas**

- A política funciona com agente desalinhado, com contexto comprimido, e em
  sessão que nunca leu o CLAUDE.md.
- Vira trilha de auditoria: bloqueio é evento, não impressão.
- Subagentes não herdam automaticamente as permissões do agente principal, o
  que faz do hook a única barreira efetiva em execução paralela.

**Negativas**

- Custo de manutenção em shell, uma linguagem sem tipos e com armadilhas de
  quoting.
- Latência somada a toda chamada de ferramenta que casa com o matcher.
- Guarda mal escrito bloqueia trabalho legítimo, e o agente não tem canal de
  apelação: ele só vê a negativa e tenta outra coisa.
- Falso senso de segurança: o que não virou hook continua sendo honra, e é
  fácil esquecer quais regras estão de que lado.

## Alternativas descartadas

- **Regras de permissão e `.claudeignore` nativos.** Podem ser contornados por
  indexação e system reminders; o hook de `PreToolUse` roda antes de toda
  chamada que casa com o matcher.
- **Confiar em CLAUDE.md e prompt.** É a origem do problema, não a solução.
- **Só CI.** Feedback tarde demais: o trabalho já foi produzido e descartar
  custa contexto e tokens. CI continua sendo a última linha, não a primeira.

## Riscos em aberto

Se a taxa de falso bloqueio for alta, os agentes vão desenvolver estratégias
de contorno (escrever em caminho permitido e mover depois). Sinal de alerta:
uso de `Bash` para operações de arquivo. Se aparecer, o matcher precisa cobrir
`Bash` também.
