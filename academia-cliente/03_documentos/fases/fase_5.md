# Fase 5 — Tarefas

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

- [ ] T5.1 — Congelar baseline e período de medição (GATE-02) @"Academia das Específicas" !26/11/2026  <!-- id:b64d39ed-e047-4468-91d9-d8247113172d -->
  > SPEC-5-001 · pré-requisito do CA-5-03 · Prova: ata de congelamento · Pré: Fase 4 aceita
- [ ] T5.2 — Matriz de validação transversal + relatório de KPIs por ciclo @"Academia das Específicas" !01/12/2026  <!-- id:7a90682f-af7b-49c5-9e0a-a4ddb07199d9 -->
  > SPEC-5-001 · CA-5-01 a CA-5-04 · Prova: conferência programática · Pré: T5.1
- [ ] T5.3 — Auditoria de amostras + decommission do Kommo + retro @"Academia das Específicas" !04/12/2026  <!-- id:4a4e1249-1c95-4d1d-a971-e24913c36d4f -->
  > SPEC-5-001 · CA-5-05, CA-5-06 · Prova: ata de decommission · Pré: T5.2
