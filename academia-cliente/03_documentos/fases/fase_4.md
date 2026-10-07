# Fase 4 — Tarefas

<!-- fase-format:2 -->

Cada linha é uma tarefa da Jornada de Execução. **Tudo que cabe num card cabe nesta linha** — se um
campo não estiver aqui, ele não tem como ser preenchido, porque é este arquivo que cria a tarefa.

```
- [ ] Título da tarefa @responsável !30/09/2026 #projeto [interno]   <!-- id:… -->
      > descrição da tarefa, uma ou mais linhas
  - [ ] subtarefa (basta indentar 2 espaços)                         <!-- id:… -->
    - [ ] sub-subtarefa (indente mais 2)                             <!-- id:… -->
```

| marcador | o que define | se você não escrever |
|---|---|---|
| `- [ ]` / `- [/]` / `- [x]` | a fazer / em andamento / concluída | a fazer |
| `@nome` | responsável (`@"Nome Composto"` com aspas) | fica **sem responsável** |
| `!dd/mm/aaaa` | prazo | fica **sem prazo** |
| `#projeto` / `#aculturamento` | tipo | Projeto de IA |
| `[interno]` | o cliente **não** vê esta tarefa | o cliente vê |
| `> texto` na linha de baixo | descrição (aparece ao abrir o card) | sem descrição |
| indentar 2 espaços | vira subtarefa da tarefa acima (vale em qualquer profundidade) | tarefa de topo |

Os marcadores só valem **no fim da linha** — `Revisar #3 do contrato` continua sendo um título.
Um título que TERMINA na forma de um marcador sai escapado com `\\` (`Ligar para \\@joao`); a barra é
só para o parser e nunca aparece no card. Você não precisa escrever isso à mão.
Marque `[x]` para concluir e adicione linhas novas à vontade: elas entram no quadro na próxima
sincronização e voltam aqui com o `<!-- id:… -->` preenchido. **Não apague o marcador de id** das
tarefas que já têm um.

- [ ] T4.1 — AG-10 sugestor de resposta com feature flag e métricas @"Academia das Específicas" !17/11/2026  <!-- id:4b98c0b2-fc71-414e-8ec9-0ad641240dc8 -->
  > SPEC-4-001 · CA-4-01, CA-4-02 · Prova: fluxo assistido em sandbox · Pré: Fase 3 aceita; GATE-13
- [ ] T4.2 — AG-11 resumidor de handoff + AG-12 classificador de perda @"Academia das Específicas" !19/11/2026  <!-- id:f18e478b-d7d6-497e-87d8-299b2ab837b5 -->
  > SPEC-4-001 · CA-4-03, CA-4-04 · Prova: fluxos assistidos · Pré: T4.1
- [ ] T4.3 — Guardião de IA (bloqueio/auditoria) + modelo sem retenção @"Academia das Específicas" !24/11/2026  <!-- id:3219b323-dd0a-405b-9069-19bcd44dfd1b -->
  > SPEC-4-001 · CA-4-05, CA-4-06 · Prova: execução maliciosa simulada · Pré: T4.2
