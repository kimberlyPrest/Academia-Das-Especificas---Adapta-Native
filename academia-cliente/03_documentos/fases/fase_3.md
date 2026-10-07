# Fase 3 — Tarefas

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

- [ ] T3.1 — Qualificação obrigatória + orçamento + próxima ação @"Academia das Específicas" !03/11/2026  <!-- id:9fd3da77-ee10-4a97-a747-47108a3045a4 -->
  > SPEC-3-001 · CA-3-01 · Prova: RPC negativa · Pré: Fase 2 aceita
- [ ] T3.2 — Cadência FUP 1/2/3 (AG-04) com contagem do GATE-07 @"Academia das Específicas" !04/11/2026  <!-- id:c01a1f33-2365-48bd-ace7-4896495d9b45 -->
  > SPEC-3-001 · CA-3-02 · Prova: cron de teste · Pré: T3.1; GATE-07
- [ ] T3.3 — Reuniões (AG-06) + negociação com alçada (GATE-05) @"Academia das Específicas" !05/11/2026  <!-- id:aff82870-1a79-4a58-b43d-1d16ca3604ab -->
  > SPEC-3-001 · CA-3-03, CA-3-04 · Prova: fluxos manuais · Pré: T3.2; GATE-05/06
- [ ] T3.4 — Checklist de matrícula (AG-08) + marco de conversão (GATE-12) @"Academia das Específicas" !06/11/2026  <!-- id:62c20021-d28e-4850-ac4e-8e747a34b13d -->
  > SPEC-3-001 · CA-3-05 · Prova: tentativa incompleta · Pré: T3.3; GATE-12
- [ ] T3.5 — Interesse 2027 (AG-07) + retomada de novembro @"Academia das Específicas" !09/11/2026  <!-- id:d8ce6e0b-ea42-45b1-81f5-657fdf0d68a6 -->
  > SPEC-3-001 · CA-3-08 · Prova: simulação de data · Pré: T3.4
- [ ] T3.6 — Painel gerencial origem→matrícula + K2 @"Academia das Específicas" !10/11/2026  <!-- id:d53df137-b9de-4817-a5ae-9f05e24e9b94 -->
  > SPEC-3-001 · CA-3-06 · Prova: conferência numérica · Pré: T3.5
- [ ] T3.7 — Import Kommo (GATE-04) com snapshot imutável @"Academia das Específicas" !12/11/2026  <!-- id:1482b238-90ae-4a54-997a-846b7b2aeb60 -->
  > SPEC-3-001 · CA-3-07 · Prova: replay + tentativa de UPDATE · Pré: T3.6; GATE-04
