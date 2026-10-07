# Fase 2 — Tarefas

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

- [ ] T2.1 — Webhooks (Meta HMAC + formulários) com idempotência e dead-letter @"Academia das Específicas" !22/10/2026  <!-- id:064c607e-877a-40bf-9ed3-c80489dbdf9e -->
  > SPEC-2-001 · CA-2-01, CA-2-06, CA-2-07 · Prova: replay + prova negativa · Pré: Fase 1 aceita; GATE-03/09
- [ ] T2.2 — AG-01..AG-03 (validação, deduplicação, distribuição) @"Academia das Específicas" !26/10/2026  <!-- id:9bdb21dc-ac5a-4374-a0f7-f085bf56cf66 -->
  > SPEC-2-001 · CA-2-02, CA-2-03 · Prova: fixtures de números/duplicatas · Pré: T2.1
- [ ] T2.3 — Inbox por unidade + janela 24h + templates com custo @"Academia das Específicas" !28/10/2026  <!-- id:002289e1-d530-44c6-8b8d-f7d036885f57 -->
  > SPEC-2-001 · CA-2-04 · Prova: fluxo manual em sandbox · Pré: T2.2
- [ ] T2.4 — Cron de SLA + painel K1 (primeira resposta humana) @"Academia das Específicas" !30/10/2026  <!-- id:7bb35268-1e93-4f21-966d-9bc14e4be95d -->
  > SPEC-2-001 · CA-2-05 · Prova: conferência numérica · Pré: T2.3
